# Workflows

## Workspace initialization
1. Initialize workspace metadata in repo root.
2. Detect existing git repo and main branch.
3. Create default role profile and env profile.
4. Register baseline facts (project purpose, constraints).

## Variant lifecycle
- Create: new variant from base ref (main or specific commit).
- Attach role and env profile.
- Work: run agent tasks and capture context packs.
- Archive: freeze variant metadata and store final patch bundle.
- Remove: clean worktree and metadata if safe.

## Example CLI flow (placeholder names)
```
agentctl init
agentctl variant create feat-context-pack --base main --role backend-worker
agentctl run --variant feat-context-pack --role backend-worker --cmd "uv run pytest"
agentctl context inspect --run <run-id>
agentctl bundle create --variant feat-context-pack
agentctl merge plan --bundle <bundle-id>
```

## Context pack flow
1. Select relevant docs and files (static + dynamic retrieval).
2. Score and rank candidates; apply budget caps.
3. Present pack for review with allow/deny controls.
4. Lock pack to run; record hash and provenance.

## Parallel work flow
1. Create N variants for parallel tasks.
2. Assign distinct role profiles per variant.
3. Run each task in isolated env profile.
4. Collect patch bundles and test results.
5. Build merge plan and integrate in order.

## Merge and verification flow
1. Generate agentic merge plan with stated steps.
2. Verify base ref matches expected.
3. Apply patch bundles in deterministic order.
4. Run verification gates (lint, test, build).
5. Require human approval for conflicts or failures.
6. Mark variants as merged and archive.

## UX expectations (dashboard)
- Variants list with status, diff summary, and last run.
- Context pack inspector with provenance and budget breakdown.
- Run history with command outputs and artifacts.
- Merge center with bundle list and verification status.
