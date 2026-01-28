# Memory system

## Goals
- Prevent "patchwork" assumptions by keeping decisions explicit.
- Keep memory seamless and low-friction for daily work.
- Make memory transparent and tool-agnostic.

## Memory layers
- Durable knowledge: decisions, assumptions, invariants.
- Active context: per-run context packs.
- Run summaries: what changed and why.
- Candidate memories: pending items awaiting approval.

## Storage layout
- `.agentplane/memory/decisions.md`
- `.agentplane/memory/assumptions.md`
- `.agentplane/memory/invariants.md`
- `.agentplane/memory/candidates.md`
- `.agentplane/context/active.md`
- `.agentplane/runs/<run-id>/summary.md`

## Memory format (Markdown + frontmatter)
Each entry uses frontmatter for structure and a short body for readability.

```md
---
id: DEC-001
type: decision
status: accepted
tags: [ui, tui]
source: run-2026-01-27-3
updated: 2026-01-27
---
Decision: Use Textual for the MVP TUI.
Rationale: Fast iteration and team familiarity.
```

## Lifecycle
1. Run produces a summary.
2. System proposes candidate memories with provenance.
3. User approves or rejects candidates.
4. Approved items are promoted into durable memory files.
5. Prompt to restart tool session to apply latest memory.

## Seamless UX
- Non-blocking badge when new candidates exist.
- One-click approve or bulk approve.
- Restart prompt after promotion with optional auto-restart toggle.

## Conflict detection
- New candidate memories are checked against existing decisions.
- Conflicts are flagged for explicit resolution before promotion.

## Tool integration
- Adapters inject memory files into tool instructions.
- Context pack references `active.md` for each run.
- If tools do not reload instructions, prompt user to restart.

## Curator mode
- On-demand mode to review memory health.
- Finds stale or conflicting assumptions.
- Suggests consolidations and deprecations.

## Advanced retrieval (post-MVP)
- Retrieval layer is pluggable.
- Start with keyword + embeddings.
- Later option: HippoRAG2 or similar for richer retrieval.
