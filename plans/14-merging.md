# Merging and agentic merge modes

## Goals
- Merge parallel work safely while preserving intent.
- Support idea-level merges when multiple solutions exist.
- Keep guardrails so merges remain auditable and testable.

## Inputs and artifacts
- Patch bundles (diffs or per-commit patches).
- Environment spec diffs and overlays.
- Memory changes (decisions/invariants/candidates).
- Test and gate results.

Outputs:
- Merge plan artifact (ordered steps + rationale).
- Integration variant (merged state).
- Optional refactor variant (cleanup or pattern merge).

## Patch bundle strategy
We support two patch bundle shapes:
- Commit-based bundle: generated with `git format-patch` and applied with `git am`.
- Diff-based bundle: generated with `git diff` and applied with `git apply`.

3-way fallbacks:
- `git am --3way` for commit bundles when clean apply fails.
- `git apply --3way` for diff bundles when clean apply fails.

References:
- git format-patch: https://git-scm.com/docs/git-format-patch
- git am: https://git-scm.com/docs/git-am
- git apply: https://git-scm.com/docs/git-apply

## Baseline merge (deterministic)
1. Create an integration variant from main.
2. Reconcile environment specs and overlays (stable point).
3. Apply patch bundles in order using 3-way apply if needed.
4. Run gates (lint/test/build) in the integration environment.
5. If conflicts remain, open a conflict variant and resolve with a merge agent.

Conflict handling is aligned with Git’s merge conflict model and uses the
working tree conflict markers when 3-way apply is needed.

References:
- git merge (conflict presentation model): https://git-scm.com/docs/git-merge

## Mode 1: Rebase-and-repair (stale variants)
Use when a variant’s base commit is behind the integration base.

Flow:
1. Detect base mismatch and compute a new base target.
2. Replay patch bundles onto the new base (same as baseline apply).
3. Use 3-way fallbacks if needed and open conflict variant on failure.
4. Merge agent resolves only conflict regions unless scope is expanded.
5. Run gates and finalize.

Notes:
- Semantically similar to `git rebase`, but using patch bundles.
- `git range-diff` can compare old vs rebased patch series for review.

References:
- git rebase: https://git-scm.com/docs/git-rebase
- git range-diff: https://git-scm.com/docs/git-range-diff

## Mode 2a: Intent-slice merge (code units)
Use when you want specific parts from different solutions.

Flow:
1. Preprocess each variant into intent units (file/component/task).
2. User selects units from each variant.
3. Merge agent stitches selected units into integration variant.
4. Run gates and present diff summary for approval.

Guardrails:
- Default scope: touched files only.
- Escalation requires approval and a plan.

## Mode 2b: Pattern merge (idea-level)
Use when the merge is about architecture or style rather than files.

Pattern Pack extraction:
- Small model + heuristics summarize:
  - architecture patterns
  - style/convention patterns
  - UI composition patterns
  - trade-offs and risks
- Each pattern includes evidence and affected modules.

Intent-driven selection:
- User describes desired outcome in plain words.
- System pre-selects matching patterns.
- User reviews and confirms before execution.

Application:
- Selected patterns become a refactor plan.
- Refactor agent applies the plan in a separate refactor variant.
- Gates validate; merge into integration variant when green.

## Scope ladder and guardrails
Scope ladder:
1. Conflict-only (default for merge agent).
2. Touched files (fallback).
3. Module boundary (default for refactor agent).
4. Dependency radius (requires approval).
5. Project-wide (explicit approval).

Additional guardrails:
- Change budget (max files/lines).
- Decisions/invariants always injected.
- No new deps without env spec approval.
- Plan + rationale required before execution.

## Post-merge refactor (optional)
Run after baseline merge and gates, never blocking shipping.

Flow:
1. Create a refactor variant from the merged integration state.
2. Apply pattern merge or targeted cleanup.
3. Run gates; if green, merge refactor patch into integration state.
4. If failed, discard without blocking the merge.

Refactor outputs:
- Cleanup patches.
- Updated memory candidates for decisions/invariants.
- Optional env refactor suggestions.

## Environment refactor (optional)
Stable point envs are safe but may be bloated.

Flow:
1. Compute a minimal spec candidate from project manifests.
2. Compare with current `spec.json` and propose a prune plan.
3. Require explicit approval for removals.
4. Rebuild image and run gates.

If gates fail, revert to the previous spec.

## Memory reconciliation
After merge or refactor:
- Propose candidate memories summarizing new decisions.
- Detect conflicts with existing decisions/invariants.
- Require approval before promotion.

## Idea grading (future)
- Optional scoring of candidate patterns/solutions:
  - complexity, duplication, test impact, performance hints
- Used to guide selection, not to auto-merge by default.
