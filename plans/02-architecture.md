# Architecture

## High-level components
- CLI and daemon (local): orchestrates variants, runs, and adapters.
- Workspace registry: stores metadata for variants, roles, env profiles.
- Variant manager: wraps git worktrees and branch lifecycle.
- Run executor: executes commands in isolated environments.
- Context manager: builds and stores context packs and facts.
- Capability manager: stores skills/prompts and exports to tools.
- Merge planner: builds patch bundles and merge plans.
- UI (optional early): dashboard for variants, context, and runs.

## Proposed local file layout (working names)
- .agentplane/ (or similar): local metadata and caches.
  - workspace.json: workspace spec and registry.
  - variants/: per-variant metadata and logs.
  - runs/: stdout/stderr logs and artifacts.
  - context/: context packs and facts store.
  - capabilities/: skill assets and profiles.

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

## APIs and surfaces
- CLI commands: init, variant create, run, context inspect, merge plan.
- JSON outputs for integration with other tools.
- UI consumes the same registry and run logs.

## Security model (local)
- Permissions per role: read/write scope and command allowlists.
- Environment isolation optional; default least privilege.
- Redaction rules for secrets in run logs.
