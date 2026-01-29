# Architecture

## High-level components
- CLI and daemon (local): orchestrates variants, runs, and adapters.
- Daemon is the primary source of truth; TUI/CLI are clients.
- Workspace registry: stores metadata for variants, roles, env profiles.
- Project manager: clones and registers repos in managed root.
- Variant manager: wraps git worktrees and branch lifecycle.
- Run executor: executes commands in isolated environments.
- Session manager: handles tool sessions and PTY streams.
- Context manager: builds and stores context packs and facts.
- Memory manager: maintains decisions/assumptions and candidate memories.
- Capability manager: stores skills/prompts and exports to tools.
- Merge planner: builds patch bundles and merge plans.
- UI (optional early): dashboard for variants, context, and runs.

## MVP implementation choices
- Python 3.12+ for daemon, TUI, and sidecar.
- Textual for TUI; Typer for CLI.
- VS Code/Cursor extension in TypeScript.

## Proposed local file layout (working names)
## Storage and configuration layout

### Global store (shared across projects)
- `~/.config/agentplane/`
  - `registry.json`: registered projects, last open, defaults.
  - `skills/`: reusable skill packs (global).
  - `profiles/`: reusable role profiles (global).
  - `adapters/`: adapter templates or defaults.

### Per-project store (repo-local, git-ignored by default)
- `<repo>/.agentplane/`
  - `workspace.json`: project metadata, default role/env, adapter settings.
  - `skills/`: project-specific skill overrides.
  - `profiles/`: project-specific role profiles.
  - `variants/`: per-variant metadata (base ref, env, role, timestamps).
  - `context/`: context packs and fact store.
  - `memory/`: decisions, assumptions, invariants, candidates.
  - `runs/`: stdout/stderr logs and artifacts.

### Generated tool configs (per-variant worktree root)
- OpenCode: `opencode.json` + `.opencode/*`
- Cursor: `.cursor/rules/*` or `.cursorrules`

## Config resolution order
1. Global defaults (`~/.config/agentplane`).
2. Project overrides (`.agentplane/`).
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
- Adapters export into tool-specific formats:
  - OpenCode: opencode.json and .opencode/* directories.
  - Cursor: .cursorrules or .cursor/rules (if used).
  - MVP first-class targets: OpenCode and Cursor.
  - CLI agents: generate prompt blocks or config snippets.

## Merge planner design
- Agentic merge assistance with explicit merge plans.
- Patch bundles from commits or diffs per variant.
- Apply via git apply or git am, with metadata checks.
- Verification gates: lint/test/status before merge.
- Optional 3-way conflict helper, but human-approved.
- Supports intent-slice and pattern merge modes.

## APIs and surfaces
- CLI commands: init, variant create, run, context inspect, merge plan.
- JSON outputs for integration with other tools.
- UI consumes the same registry and run logs.
- VS Code/Cursor extension uses RPC to open/attach to variants.

## Security model (local)
- Permissions per role: read/write scope and command allowlists.
- Environment isolation optional; default least privilege.
- Redaction rules for secrets in run logs.
