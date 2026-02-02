# Roadmap (implementation, 4-person team)

This roadmap prioritizes early end-to-end validation and fast iteration.
Each phase yields a working slice that can be tested with real agents.

## Integration strategy alignment

The roadmap follows the 4-layer integration strategy:

```
Phase 0-1: Layer 1 (CLI-first Foundation) - core platform
Phase 1-2: Layer 2 (Config/Skill Export) - OpenCode, Cursor, etc.
Phase 2-3: Layer 3 (OpenCode Deep) - OSS collaboration
Phase 3+:  Layer 4 (Protocol) - document patterns as specs
```

See `plans/16-integration-strategy.md` for layer details.

## Workstreams (team of 4)
- Core/Daemon: registry, variants, run executor, APIs.
- CLI/UX: prompts, output formatting, status views.
- Env/Sidecar: containers, env spec, drift detection, port mapping.
- Merge/Agentic: patch bundles, merge plans, refactor modes.
- UI Clients (post-CLI): evaluate GUI vs TUI; VS Code extension for observability.

## Phase 0: Scaffolding and vertical slice (Layer 1)
Focus: CLI-first foundation that works with ANY agent tool.

Deliverables:
- CLI commands: `lazyagent project init`, `lazyagent project fork create`, `lazyagent project list`, `lazyagent list`.
- CLI project creation wizard (new project + register) with project-local blueprint.
- Variant manager wrapping git worktrees.
- Native run executor: `lazyagent project run <any-command>` with logs.
- Basic context snapshot on fork (tracked files, git state).
- Patch bundle export (diff-based) and apply.
- Daemon skeleton with project registry.

Exit: can fork a variant, run ANY tool inside it, capture changes as patch.

**Layer 1 validation**: User runs `lazyagent project fork create && opencode` (or cursor, or claude) and it just works.

## Phase 1: MVP user loop (Layer 1 complete + Layer 2 starts)
Focus: complete CLI loop, begin config/skill export plugins.

Deliverables:
- Project onboarding (CLI): clone + init + registry.
- Embedded PTY sessions (native), accessible via CLI.
- "Create from template" and "create as project" flows with prefilled wizard choices.
- Optional save-as-template prompt and template export utility.
- Baseline merge flow with gates (lint/test/build).
- Port allocation at startup (static mapping) + CLI display.
- Minimal memory store (facts/decisions files).
- **Layer 2**: OpenCode export plugin (opencode.json, .opencode/*).

Exit: a user can onboard a repo, fork, run an agent, and merge safely.

**Layer 1 complete**: CLI core works with any tool; UI clients deferred.
**Layer 2 begins**: OpenCode users get config/skill export.

## UI assessment checkpoint (post-Phase 1)
Focus: decide if a UI client is needed based on CLI usage.

Deliverables:
- UI requirements list driven by real CLI workflows.
- Decision: simple GUI dashboard, TUI, VS Code extension, or none.
- If chosen, a minimal read-only dashboard for observability and quick switching.

## Phase 2: Containers + sidecar + env spec
Focus: true isolation and repeatable environments.

Deliverables:
- Container runner with bind-mounted worktrees.
- Sidecar bridge for PTY streaming + status.
- Env spec + journal + overlays.
- Drift detection signals (package shims + docker diff + watchers).
- Template selection (basic) and devcontainer config generation.

Exit: containerized variants with auditable env changes.

## Phase 3: Capabilities, context, and memory UX (Layer 2 + 3 + 4)
Focus: transparency, managed skills, deep OpenCode collaboration, protocol docs.

Deliverables:
- Capability registry (skills/plugins/MCPs) with tags/groups.
- Role profiles with on/off toggles.
- Context pack builder with provenance and budgets.
- Memory view + candidate approvals + restart prompts.
- **Layer 2**: More export plugins (Cursor, Claude Code, Aider).
- **Layer 3**: OpenCode deep - skill sync, hook integration, upstream contributions.
- **Layer 4**: Document context pack schema, variant metadata format.

Exit: repeatable roles, transparent context, managed skills.

**Layer 2 mature**: Multiple tool exports working.
**Layer 3 active**: OpenCode collaboration underway.
**Layer 4 begins**: Other tools can follow documented specs.

## Phase 4: Agentic merge modes + refactor
Focus: advanced merge value and cleanup.

Deliverables:
- Mode 1 rebase-and-repair for stale variants.
- Mode 2a intent-slice merge (unit selection).
- Mode 2b pattern merge (idea-level) with refactor variant.
- Post-merge refactor flow with scope ladder.
- Env refactor proposal and prune workflow.

Exit: reliable advanced merges with guardrails and cleanup.

## Phase 5: Workflow orchestration (north star)
Focus: multi-agent pipelines and governance.

Deliverables:
- Declarative workflow runner with step artifacts.
- Parallel worker dispatch to variants.
- Optional UI dashboard or VS Code extension for multi-run status; CLI provides status outputs.
- Policy enforcement for roles and commands.

Exit: repeatable multi-agent pipelines.

## Phase 6: Hardening and scale
Focus: stability, performance, packaging.

Deliverables:
- Reliability improvements, crash recovery, data pruning.
- Optional Go rewrite targets (daemon/sidecar) if needed.
- Telemetry hooks (local-only by default).

Exit: production-ready local system.

## Future directions (not scheduled, post-MVP)
These are explicit "nice-to-have" directions. They are not committed to the
roadmap unless the core CLI product proves strong demand and resources allow.

- Codex-inspired Git surface in a UI: diff panel, stage/revert, commit, and
  PR creation scoped to variants/worktrees with merge-gate awareness.
- "Open in editor" launcher: per-project default editor, multi-editor menu, and
  one-click open for a variant's worktree path.
- Action buttons in UI: per-project run/test/lint/build shortcuts backed by
  embedded terminal logs and run artifacts.
- Review mode: single-pane inspection of uncommitted changes and patch bundles
  before merge.

## Agent-assisted workflow (all phases)
- Use coding agents for scaffolding, test generation, and docs sync.
- Require plan + rationale for agentic changes.
- Human review for merge/refactor outputs.
