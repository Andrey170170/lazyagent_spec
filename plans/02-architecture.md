# Architecture

## High-level components
- CLI (primary) and daemon (local): orchestrates variants, runs, and adapters.
- Daemon is the primary source of truth; CLI is primary client, UI optional later.
- Workspace registry: stores metadata for variants, roles, env profiles.
- Project manager: clones and registers repos in managed root.
- Project scaffolder: drives creation wizard, project blueprints, and templates.
- Variant manager: wraps git worktrees and branch lifecycle.
- Run executor: executes commands in isolated environments.
- Session manager: handles tool sessions and PTY streams.
- Context manager: builds and stores context packs and facts.
- Memory manager: maintains decisions/assumptions and candidate memories.
- Capability manager: stores skills/prompts and exports to tools.
- Merge planner: builds patch bundles and merge plans.
- UI clients (optional, later): dashboard for variants, context, and runs (GUI/TUI/extension).

## MVP implementation choices
- Python 3.12+ for daemon and sidecar.
- Typer for CLI; UI client TBD (Textual if TUI is chosen later).
- VS Code/Cursor extension in TypeScript.

## Proposed local file layout (working names)
## Storage and configuration layout

### Global store (shared across projects)
- `~/.config/lazyagent/`
  - `registry.json`: registered projects, last open, defaults.
  - `skills/`: reusable skill packs (global).
  - `profiles/`: reusable role profiles (global).
  - `adapters/`: adapter templates or defaults.

### Per-project store (repo-local, git-ignored by default)
- `<repo>/.lazyagent/`
  - `workspace.json`: project metadata, default role/env, adapter settings.
  - `skills/`: project-specific skill overrides.
  - `profiles/`: project-specific role profiles.
  - `variants/`: per-variant metadata (base ref, env, role, timestamps).
  - `context/`: context packs and fact store.
  - `memory/`: decisions, assumptions, invariants, candidates.
  - `runs/`: stdout/stderr logs and artifacts.

### Generated tool configs (per-variant worktree root, via Layer 2 export plugins)
- OpenCode: `opencode.json` + `.opencode/*`
- Cursor: `.cursorrules` or `.cursor/rules/*`
- Claude Code: `.claude/skills/*`, `CLAUDE.md`
- Generic: Markdown prompt blocks in known paths
- Fallback (Layer 1): `LAZYAGENT_*` env vars for any tool

## Config resolution order
1. Global defaults (`~/.config/lazyagent`).
2. Project overrides (`.lazyagent/`).
3. Role profile (global or project).
4. Variant overrides (per-variant metadata).
5. Generated tool configs (derived artifacts).

## Restartability
- On startup, the daemon reads `registry.json` and reconciles projects.
- Worktrees/containers are reattached when possible; stale entries are flagged.
- Sidebar session state is restored from the last known run.

## Tool memory injection
- Adapters always point tools to the canonical memory files.
- After memory promotion, prompt user to restart sessions.

## Data model (minimal)
- Workspace: name, root path, default role, default env, created_at.
- Variant: id, name, git_worktree_path, base_ref, role_profile, env_profile.
- Run: id, variant_id, role_profile, command, status, timestamps, artifacts.
- ContextPack: id, run_id, items[], budget, provenance, hash.
- SkillAsset: id, name, version, tags, files, compatible_tools[]
- RoleProfile: id, name, skills[], permissions, allowed_commands.
- EnvProfile: id, type (devcontainer/compose/native), config_ref.
- PatchBundle: id, variant_id, base_ref, commits[], tests_run, notes.

## Variant manager design
- Use git worktrees for parallel workspaces.
- Map variant metadata to worktree path and branch.
- Support detached or branch-backed worktrees.
- Handle worktree cleanup with explicit remove and prune safety.
- Future direction: allow a change-graph or patch-stack backend, with worktrees
  as a materialization step rather than the source of truth.

## Environment strategy
- Native: run commands in host environment (no container).
- Devcontainer: use devcontainer.json to build/run per variant.
- Compose: attach to a compose service for multi-service stacks.
- Resource limits: use container runtime flags where available.
- Bind-mount worktrees into containers for fast variant startup.

## Environment templates and specs
- Template store provides reusable base images and feature sets.
- Environment spec captures the reproducible setup for a project.
- Variants can apply temporary overlays without mutating the base spec.
- Sidecar records runtime installs and can promote them into the spec.
- Drift signals: package manager shims, docker diff, targeted file watches.

## Sidecar bridge (container)
- Lightweight agent gateway running inside each container.
- Exposes PTY stream for interactive tool UIs.
- Exposes status endpoints (git status, diff count, run state, todos).
- Coordinates skill/config mounts with safe updates.

## Port management
- MVP: allocate ports at startup; record logical->actual mappings per variant.
- Inject PORT/BASE_URL env vars before process start.
- Future: host-side proxy for dynamic remapping without restarts.

## Context manager design
- Context pack pipeline: select -> rank -> review -> lock.
- Store provenance: file path, line ranges, reason, timestamp.
- Budget ledger: record token allocation per source class.
- Fact store: curated facts and decisions with versioning.

## Capability manager design
- Canonical registry for prompts, skills, hooks, and policies.
- Role profiles = named sets of skills + permissions.

### Two types of "adapters" - important distinction

**Config/Skill Export (Good - We Build These)**
- Convert canonical skills/configs to tool-specific formats
- Export to `.cursorrules`, `opencode.json`, Claude Code skills, etc.
- Stable: these are documented config file formats
- This is the core value prop: one source, multiple tool outputs

**Deep Behavioral Integration (Risky - We Avoid This)**
- Hooking into tool internals, intercepting behavior
- Depending on undocumented APIs or data structures
- Breaks on minor tool updates

### Integration layers

**Layer 1: CLI-first Foundation**
- Core platform: workspace, env, context, merge
- Works with ANY tool even without export plugins
- UI optional for dashboard views and sessions

**Layer 2: Config/Skill Export Plugins**
- "Adapt project for work with X tool" feature
- Plugin architecture: canonical format in → tool format out
- Supported: OpenCode, Cursor, Claude Code, Aider, generic markdown
- Community can contribute new export plugins

**Layer 3: OpenCode Deep Integration**
- OSS collaboration, not fragile adaptation
- Contribute features upstream
- Co-design APIs and shared formats
- Native support for LazyAgent workspaces in OpenCode

**Layer 4: Emergent Protocol**
- Document stable patterns as schemas
- Integration guides for tool authors
- We maintain docs, not behavioral adapters

See `plans/16-integration-strategy.md` for full details.

## Merge planner design
- Agentic merge assistance with explicit merge plans.
- Patch bundles from commits or diffs per variant.
- Apply via git apply or git am, with metadata checks.
- Verification gates: lint/test/status before merge.
- Optional 3-way conflict helper, but human-approved.
- Supports intent-slice and pattern merge modes.

## Non-git rollback (future)
- Consider a snapshot store for ad-hoc tasks outside Git.
- Store snapshots in the global store with explicit path allowlists.
- Surface the same diff/restore UX as git-backed variants.

## APIs and surfaces
- CLI commands: init, variant create, run, context inspect, merge plan.
- JSON outputs for integration with other tools.
- UI consumes the same registry and run logs.
- VS Code/Cursor extension uses RPC to open/attach to variants.

## Security model (local)
- Permissions per role: read/write scope and command allowlists.
- Environment isolation optional; default least privilege.
- Redaction rules for secrets in run logs.
