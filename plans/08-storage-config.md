# Storage and configuration model

## Goals
- Restartable: recover state after app restart without cloud services.
- Deterministic: clear precedence when configs overlap.
- Safe: avoid drift between generated tool configs and source profiles.
- Extensible: allow global defaults and per-project overrides.

## Global store
Located at `~/.config/lazyagent/`.

### Contents
- `registry.json`: list of registered projects, last opened, defaults.
- `skills/`: reusable skill packs (global scope).
- `profiles/`: reusable role profiles (global scope).
- `adapters/`: templates and defaults for tool adapters.
- `templates/env/`: reusable environment templates.
- `templates/project/`: reusable project templates (saved from wizard).

## Per-project store
Located at `<repo>/.lazyagent/` and git-ignored by default.

### Contents
- `workspace.json`: project metadata, default role/env, adapter settings.
- `project.json`: project creation blueprint (language, runtime, tool packs, choices).
- `skills/`: project-specific skill overrides and prompts.
- `profiles/`: project-specific role profiles.
- `env/`: environment spec and journal (see below).
- `memory/`: decisions, assumptions, invariants, candidates.
- `variants/`: per-variant metadata and status.
- `context/`: context packs and fact store.
- `runs/`: stdout/stderr logs, test reports, artifacts.

## Generated tool configs (per variant)
Stored in the variant worktree root and regenerated as needed.

- OpenCode: `opencode.json` + `.opencode/*`
- Cursor: `.cursor/rules/*` or `.cursorrules`

## Config resolution order
1. Global defaults in `~/.config/lazyagent/`.
2. Project overrides in `.lazyagent/`.
3. Role profile (global or project).
4. Variant overrides (metadata under `.lazyagent/variants/`).
5. Generated tool configs (derived artifacts).

## Restartability behavior
- On startup, load `registry.json` and scan projects.
- Reconcile worktrees and containers; mark stale entries.
- Restore sidebar session state from last known run status.

## Agent-editable configs (optional)
- Default: skills/configs are read-only inside containers (bind-mounted).
- If enabled: sidecar requests updates via daemon; daemon applies edits on host.
- During updates, tool sessions are paused or locked to avoid partial writes.

## Environment specs
Located at `.lazyagent/env/`.

- `spec.json`: canonical environment spec (base image + features + packages).
- `journal.jsonl`: append-only log of runtime installs detected by sidecar.
- `overlays/`: per-variant env overlays (optional).

## Memory store
Located at `.lazyagent/memory/`.

- `decisions.md`: durable product/tech decisions.
- `assumptions.md`: assumptions pending validation.
- `invariants.md`: rules that should not be violated.
- `candidates.md`: pending memory suggestions.

## Example file: `.lazyagent/workspace.json`
```json
{
  "project_name": "lazyagent",
  "default_role": "backend-worker",
  "default_env": "devcontainer",
  "adapter": {
    "opencode": {"enabled": true},
    "cursor": {"enabled": true}
  }
}
```
