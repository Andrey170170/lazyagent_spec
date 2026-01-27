# Workflows

## Workspace initialization
1. Initialize workspace metadata in repo root.
2. Detect existing git repo and main branch.
3. Create default role profile and env profile.
4. Register baseline facts (project purpose, constraints).

## Project onboarding (TUI)
1. Select "Add project" and enter a Git URL or local path.
2. Clone into managed projects root (if URL).
3. Run workspace init and detect tooling.
4. Show project in sidebar with status.

## Variant lifecycle
- Create: new variant from base ref (main or specific commit).
- Attach role and env profile.
- Work: run agent tasks and capture context packs.
- Archive: freeze variant metadata and store final patch bundle.
- Remove: clean worktree and metadata if safe.

## Fork and connect flow (container)
1. Trigger Fork in TUI and provide variant name.
2. Create worktree and attach env profile.
3. Start container with bind-mounted worktree.
4. Launch sidecar bridge in container.
5. Attach to tool session via PTY stream.
6. Sidebar shows a spinner while build runs without blocking main view.

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

## Session switching
1. Sidebar lists active variants with status indicators.
2. Select a variant to attach its tool session.
3. Background sessions continue running with status updates.

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
