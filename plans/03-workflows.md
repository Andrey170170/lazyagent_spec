# Workflows

## Workspace initialization
1. Initialize workspace metadata in repo root.
2. Detect existing git repo and main branch.
3. Create default role profile and env profile.
4. Register baseline facts (project purpose, constraints).

## Project onboarding (CLI-first; UI optional)
1. Select "Add project" and enter a Git URL or local path.
2. Clone into managed projects root (if URL).
3. Run workspace init and detect tooling.
4. Show project in status output (UI dashboard optional later).

## Project creation wizard (CLI-first)
1. Choose source: new project, clone repo, register existing, create from template, or create as another project.
2. For templates or "create as": select base and decide "use as-is" or "edit choices".
3. Choose language, package manager, project location, and runtime (native or Docker for MVP).
4. Select tool packs (e.g., eza, fzf) and agent tools to install/configure.
5. Scaffold repo structure, baseline docs, and initial configs.
6. Write project-local blueprint and env spec under `.lazyagent/`.
7. Register project and summarize choices.
8. Prompt to save as reusable template (default: no).

## Environment initialization
1. Detect project type (pyproject.toml, package.json, etc.).
2. Suggest a template (Python, Node, Rust, etc.).
3. User selects template and optional features.
4. Generate environment spec + devcontainer config.
5. Build the image in background; show sidebar spinner.

## Variant lifecycle
- Create: new variant from base ref (main or specific commit).
- Attach role and env profile.
- Work: run agent tasks and capture context packs.
- Archive: freeze variant metadata and store final patch bundle.
- Remove: clean worktree and metadata if safe.

## Fork and connect flow (container)
1. Trigger Fork via CLI (UI optional later) and provide variant name.
2. Create worktree and attach env profile.
3. Start container with bind-mounted worktree.
4. Launch sidecar bridge in container.
5. Allocate ports for the variant; set PORT/BASE_URL env vars.
6. Attach to tool session via PTY stream.
7. Optional UI dashboard shows a spinner while build runs without blocking main view.

## Environment drift and promotion
1. Sidecar detects package installs (apt/pip/npm).
2. Record changes in an env journal.
3. Prompt to promote changes into env spec or keep as variant overlay.
4. If promoted, regenerate Dockerfile/devcontainer config and rebuild.

## Example CLI flow (placeholder names)
```
lazyagent init
lazyagent variant create feat-context-pack --base main --role backend-worker
lazyagent run --variant feat-context-pack --role backend-worker --cmd "uv run pytest"
lazyagent context inspect --run <run-id>
lazyagent bundle create --variant feat-context-pack
lazyagent merge plan --bundle <bundle-id>
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
1. Optional UI dashboard lists active variants with status indicators.
2. Select a variant to attach its tool session (CLI or UI).
3. Background sessions continue running with status updates (or via CLI status output).

## Memory formation flow
1. Run produces a summary.
2. System generates candidate memories with provenance.
3. User approves or rejects candidates.
4. Approved memories are promoted to durable files.
5. Prompt to restart sessions to apply updates (optional auto-restart).

## Merge and verification flow
1. Generate agentic merge plan with stated steps.
2. Verify base ref matches expected.
3. Reconcile environment specs and overlays.
4. Apply patch bundles in deterministic order.
5. Run verification gates (lint, test, build).
6. Require human approval for conflicts or failures.
7. Mark variants as merged and archive.

## Post-merge refactor (optional)
1. Create a refactor variant from the merged integration state.
2. Apply pattern merge or targeted cleanup.
3. Run gates; if green, merge refactor patch.
4. If failed, discard without blocking the merge.

## UX expectations (optional UI dashboard)
- Variants list with status, diff summary, and last run.
- Context pack inspector with provenance and budget breakdown.
- Run history with command outputs and artifacts.
- Merge center with bundle list and verification status.
