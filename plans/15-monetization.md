# Monetization plan and assumptions

## Goals
- Keep the core workflow free and useful.
- Monetize the high‑value, compute‑heavy features.
- Align pricing with real costs (LLM usage, analysis, and orchestration).

## Technical analysis (what costs more)
Based on the current architecture:
- Agentic merge modes (intent‑slice + pattern merge) require extra analysis and LLM passes.
- Post‑merge refactor and idea grading require additional model calls and gates.
- Memory curation (auto‑suggest + curator mode) adds ongoing analysis overhead.
- Remote registries (templates/skills) require hosting and distribution.
- Dynamic port proxying adds a long‑running host service.

Local‑first features have low marginal cost:
- Variants (git worktrees), context packs, env specs, and baseline merges are local.
- Sidecar and daemon run locally without cloud infra.

## Proposed tiering

### Free (core)
- CLI + daemon; optional UI client.
- Project onboarding + variants (worktrees).
- Environment specs and templates (local only).
- Context packs (manual selection + review).
- Memory files with manual approval.
- Baseline merge and Mode 1 (rebase‑and‑repair) with gates.
- Port allocation at startup (static mapping).

### Pro add‑on (agentic merge + intelligence)
- Mode 2a intent‑slice merge.
- Mode 2b pattern/idea merge (Pattern Packs).
- Post‑merge refactor agent with scope ladder.
- Idea grading (complexity/duplication/test impact signals).
- Memory curator mode + auto‑suggested candidates.

### Team/Enterprise (future)
- Shared registries for templates/skills/policies.
- Org‑wide policy enforcement and audit exports.
- Remote template registry + approved images.
- Optional hosted merge compute (if offered).

## Assumptions
- Users bring their own LLM keys in free tier (no hosted inference).
- Pro features can still use user‑provided keys, but add higher‑value automation.
- If hosted inference is added later, it can bundle usage credits.

## Feature gating (technical)
- Daemon enforces feature flags from license.
- CLI (primary) and optional UI clients hide or lock paid actions when unavailable.
- All artifacts remain local; gating controls behavior, not data access.

## Pricing signals (non‑final)
- Pro pricing aligned to LLM usage frequency and merge automation value.
- Team pricing aligned to number of seats + shared registries.

## Risks and mitigations
- Risk: gating too much reduces adoption.
  - Mitigation: keep baseline merge, memory, and env management free.
- Risk: pro features feel opaque.
  - Mitigation: require plans, rationale, and gates for all agentic merges.
