# Workflows

## Workspace initialization
1. Initialize workspace metadata in repo root.
2. Detect existing git repo and main branch.
3. Create default agent/runtime preferences and env profile.
4. Register baseline facts (project purpose, constraints).

## Project onboarding (CLI-first; UI optional)
1. Select "Add project" and enter a Git URL or local path.
2. Clone into managed projects root (if URL).
3. Run workspace init and detect tooling.
4. Show project in status output (UI dashboard optional later).

## Project creation wizard (CLI-first)
1. Choose source: new project, clone repo, register existing, create from template, or create as another project.
2. For templates or "create as": select base+modules and decide "use defaults" or "customize".
3. Choose language, package manager, project location, and runtime profile.
4. Select tool packs and preferred agents.
5. Show defaults-first review with optional targeted overrides.
6. Run compatibility/auth/runtime preflight before apply.
7. Scaffold repo structure and write project blueprint/state under `.lazyagent/`.
8. If runtime is Docker, prompt build-now (default yes in interactive mode).
9. Set explicit readiness state (`ready` or `needs_action`) and summarize next actions.
10. Prompt to save as reusable template (default: no).

## Environment initialization
1. Detect project type (pyproject.toml, package.json, etc.).
2. Suggest a template (Python, Node, Rust, etc.).
3. User selects template and optional features.
4. Generate environment spec + devcontainer config.
5. Build the image in background; show sidebar spinner.

## Variant lifecycle
- Create: new variant from base ref (main or specific commit).
- Attach runtime profile, agent defaults, and optional credential additions.
- Work: run agent tasks and capture context packs.
- Archive: freeze variant metadata and store final patch bundle.
- Remove: clean worktree and metadata if safe.

## Variant + agent connect flow (runtime profile)
1. Create variant via CLI and provide variant name/base.
2. Resolve runtime profile and reuse or create backend resource.
3. Run start preflight (runtime/auth/binary/config/drift).
4. Start dedicated agent session and attach by default (TTY).
5. Allocate ports for the variant; set PORT/BASE_URL env vars.
6. Optional UI dashboard shows build/start progress without blocking main view.

## Environment drift and promotion
1. Sidecar detects package installs (apt/pip/npm).
2. Record changes in an env journal.
3. Prompt to promote changes into env spec or keep as variant overlay.
4. If promoted, regenerate Dockerfile/devcontainer config and rebuild.

## Example CLI flow (placeholder names)
```
lazyagent project create --template python-cli --runtime-profile docker
lazyagent variant create --name feat-context-pack --base-branch main
lazyagent agent start opencode --variant feat-context-pack
lazyagent run exec uv run pytest --variant feat-context-pack
lazyagent project reconcile --project <project>
lazyagent runtime drift --project <project> --domain auth
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
2. Select a variant and use `agent attach` (or low-level `session attach`) to reconnect.
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
