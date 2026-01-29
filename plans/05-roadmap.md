# Roadmap (implementation, 4-person team)

This roadmap prioritizes early end-to-end validation and fast iteration.
Each phase yields a working slice that can be tested with real agents.

## Workstreams (team of 4)
- Core/Daemon: registry, variants, run executor, APIs.
- TUI/UX: navigation, session views, status, search.
- Env/Sidecar: containers, env spec, drift detection, port mapping.
- Merge/Agentic: patch bundles, merge plans, refactor modes.

## Phase 0: Scaffolding and vertical slice
Focus: run real workflows quickly.

Deliverables:
- Daemon skeleton with project registry and variant create/list.
- TUI skeleton (project list, variant list, status area).
- Native run executor with logs.
- Minimal memory store (facts/decisions files).
- Patch bundle export (diff-based) and apply on integration variant.

Exit: can fork a variant, run a command, and apply a patch bundle.

## Phase 1: MVP user loop (fast iteration)
Focus: test ideas with actual tools and fast feedback.

Deliverables:
- Project onboarding (TUI): clone + init + registry.
- Embedded PTY sessions (native).
- Baseline merge flow with gates (lint/test/build).
- Port allocation at startup (static mapping) + UI display.
- Minimal adapter export for OpenCode (config files only).

Exit: a user can onboard a repo, fork, run an agent, and merge safely.

## Phase 2: Containers + sidecar + env spec
Focus: true isolation and repeatable environments.

Deliverables:
- Container runner with bind-mounted worktrees.
- Sidecar bridge for PTY streaming + status.
- Env spec + journal + overlays.
- Drift detection signals (package shims + docker diff + watchers).
- Template selection (basic) and devcontainer config generation.

Exit: containerized variants with auditable env changes.

## Phase 3: Capabilities, context, and memory UX
Focus: transparency and managed skills.

Deliverables:
- Capability registry (skills/plugins/MCPs) with tags/groups.
- Role profiles with on/off toggles.
- Context pack builder with provenance and budgets.
- Memory view + candidate approvals + restart prompts.
- Cursor adapter export (rules files).

Exit: repeatable roles, transparent context, managed skills.

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
- UI dashboard for multi-run status.
- Policy enforcement for roles and commands.

Exit: repeatable multi-agent pipelines.

## Phase 6: Hardening and scale
Focus: stability, performance, packaging.

Deliverables:
- Reliability improvements, crash recovery, data pruning.
- Optional Go rewrite targets (daemon/sidecar) if needed.
- Telemetry hooks (local-only by default).

Exit: production-ready local system.

## Agent-assisted workflow (all phases)
- Use coding agents for scaffolding, test generation, and docs sync.
- Require plan + rationale for agentic changes.
- Human review for merge/refactor outputs.
