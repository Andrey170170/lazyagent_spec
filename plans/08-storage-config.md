# Storage and configuration model

## Goals
- Restartable: recover state after app restart without cloud services.
- Deterministic: clear precedence when configs overlap.
- Safe: avoid drift between generated tool configs and source profiles.
- Extensible: allow global defaults and per-project overrides.

## Global store
Located at `~/.config/agentplane/`.

### Contents
- `registry.json`: list of registered projects, last opened, defaults.
- `skills/`: reusable skill packs (global scope).
- `profiles/`: reusable role profiles (global scope).
- `adapters/`: templates and defaults for tool adapters.

## Per-project store
Located at `<repo>/.agentplane/` and git-ignored by default.

### Contents
- `workspace.json`: project metadata, default role/env, adapter settings.
- `skills/`: project-specific skill overrides and prompts.
- `profiles/`: project-specific role profiles.
- `variants/`: per-variant metadata and status.
- `context/`: context packs and fact store.
- `runs/`: stdout/stderr logs, test reports, artifacts.

## Generated tool configs (per variant)
Stored in the variant worktree root and regenerated as needed.

- OpenCode: `opencode.json` + `.opencode/*`
- Cursor: `.cursor/rules/*` or `.cursorrules`

## Config resolution order
1. Global defaults in `~/.config/agentplane/`.
2. Project overrides in `.agentplane/`.
3. Role profile (global or project).
4. Variant overrides (metadata under `.agentplane/variants/`).
5. Generated tool configs (derived artifacts).

## Restartability behavior
- On startup, load `registry.json` and scan projects.
- Reconcile worktrees and containers; mark stale entries.
- Restore sidebar session state from last known run status.

## Agent-editable configs (optional)
- Default: skills/configs are read-only inside containers (bind-mounted).
- If enabled: sidecar requests updates via daemon; daemon applies edits on host.
- During updates, tool sessions are paused or locked to avoid partial writes.

## Example file: `.agentplane/workspace.json`
```json
{
  "project_name": "agentplane",
  "default_role": "backend-worker",
  "default_env": "devcontainer",
  "adapter": {
    "opencode": {"enabled": true},
    "cursor": {"enabled": true}
  }
}
```
