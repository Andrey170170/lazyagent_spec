# Roadmap (proposed)

## Phase 0: Spec and validation
- Finalize glossary and workspace spec format.
- Document adapter targets and capabilities.
- Validate key tech assumptions (git worktrees, devcontainers).
Exit: spec approved and constraints captured.

## Phase 1: MVP control plane
- CLI init and workspace registry.
- Variant manager using git worktree add/list/remove.
- Run executor for native commands with logs.
- Basic context pack artifact (manual selection).
 - Python-first core services and Textual TUI.
Exit: user can create variants and run commands with auditable logs.

## Phase 2: Capability and context
- Skill registry and role profiles.
- Tool adapters for OpenCode and Cursor.
- Context pack builder with provenance and budget fields.
Exit: repeatable roles and explicit context packs.

## Phase 3: Isolation and environments
- Devcontainer and compose env profiles.
- Resource limit controls and run isolation.
- Artifact storage per variant (logs, reports).
Exit: parallel variant runs without interference.

## Phase 4: Merge and verification
- Agentic merge assistance with explicit merge plans.
- Patch bundles with metadata and test evidence.
- Merge plan builder with gated verification.
- Conflict assistance workflow.
Exit: safe integration of parallel work.

## Phase 5: Workflow orchestration (north star)
- Declarative workflows with step outputs.
- Parallel worker dispatch to variants.
- UI dashboard for multi-run status.
Exit: repeatable pipelines across agent tools.

## Phase 6: Post-validation hardening (optional)
- Consider Go rewrite of daemon/sidecar if performance or packaging demands it.
