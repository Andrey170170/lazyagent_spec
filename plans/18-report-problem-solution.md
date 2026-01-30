# LazyAgent Product Report (Problem/Solution)

## Problem: What is the problem? Why is it an important problem?

Agent-driven development is powerful but increasingly chaotic in real projects:

- Capability sprawl: each agent tool has its own skill/config conventions, leading to duplicated setup and inconsistent behavior across repos and machines.
- Opaque context: users often cannot see (or later reproduce) what files, snippets, and assumptions an agent used for a given run.
- Environment friction: parallel agent work collides on ports, dependencies, and state; teams struggle to make runs reproducible.
- Merge anxiety: parallel variants produce drift and conflicts; automation without guardrails reduces trust.
- Audit and safety gaps: logs/context can leak secrets; permissions are often implicit; runs are hard to inspect after the fact.

This is important because the leverage of agent tools increases with parallelism and repeatability; without an orchestration layer, teams either slow down to stay safe or move fast and accept increasing integration risk.


## Solution: What is your solution? Why does it matter?

LazyAgent is a local-first control plane that turns agentic coding into a disciplined workflow by standardizing:

- Variants: parallel workspaces backed by git worktrees, with explicit lifecycle and metadata.
- Environments: native/devcontainer/compose profiles with drift detection, promotion to a canonical env spec, and predictable port allocation.
- Project creation: CLI wizard with runtime/tool selection, project-local blueprints, and reusable templates.
- Capabilities: a canonical registry of skills/prompts/policies and role profiles, exported into multiple tool formats via stable config/skill export plugins.
- Context and memory: context packs as explicit artifacts with provenance/budgets, and durable memory (decisions/assumptions/invariants) with approval-gated candidates.
- Merging: patch bundles + deterministic merge plans + verification gates; optional advanced merge modes remain opt-in and auditable.

It matters because it makes agent-driven work reproducible, inspectable, and safe to integrate while remaining tool-agnostic and offline-capable.


## Vision: If your product is successful, what does the future look like?

If successful, LazyAgent becomes the default “workspace layer” for agentic development and (increasingly) a standard entry point for running agent swarms:

- “Want a horde of agents?” becomes a solved problem: you launch many workers against one repo/task, each in an isolated variant (worktree + env), without manually managing branches, ports, or environment collisions.
- As agent tools add swarm-style execution (and as model/compute costs continue to fall), large parallelization becomes a normal workflow instead of a novelty: many cheap specialists explore different approaches or implement competing solutions.
- Runs become replayable and auditable by construction: every worker produces a provenance trail (variant + env spec + context pack + run logs + patch bundle + gates).
- Smart merge becomes the differentiator at scale: the merge system can compare multiple candidate implementations, surface trade-offs, run verification gates, and (optionally) “grade” solutions to recommend the best path with clear evidence.
- Execution backends expand: local-first remains the default, but additional backends (including cloud execution) can scale concurrency to hundreds of isolated environments per user/team while reducing local setup burden.
- Team workflows emerge naturally: shared role profiles/policies, shared capability registries, and collaboration surfaces (review, merge, audit exports) fit into how companies already ship code.
- Canonical formats become a standard: LazyAgent’s capability manifests, context packs, and variant metadata are adopted as import/export targets so tools interoperate through stable schemas rather than one-off adapters.


## Competitive Landscape: What are existing alternatives?

Alternatives typically solve only part of the problem:

- Sculptor (Imbue) (https://imbue.com/sculptor/): a UI for running parallel Claude Code agents in isolated Docker containers, with “Pairing Mode” to bring an agent’s branch into your IDE and merge/conflict-help tooling.
- Conductor (https://www.conductor.build/): a Mac app to run multiple Claude Code (and Codex) agents in isolated local workspaces (git worktrees), with diff/PR/merge flows and the ability to open a workspace in your IDE.
- Superconductor (https://www.superconductor.com/): a cloud platform to run multiple coding agents in parallel with isolated environments and live previews, plus collaboration/workflow integrations (GitHub PRs, GitHub comment triggers, Slack).
- Graphite (https://graphite.dev/) (adjacent): an AI code review platform focused on PR workflow (stacked PRs, merge queue, PR inbox, AI reviews/agent) rather than being a local orchestration layer for agent workspaces.
- Individual agent tools (IDE agents, CLI agents): provide prompting/skills and basic session UX, but rarely standardize cross-tool behavior, explicit context provenance, or deterministic merge plans.
- Devcontainers/Docker Compose + git branching/worktrees: provide pieces of environment and workspace parallelism, but not an end-to-end system for capabilities, context/memory artifacts, run logs, and merge workflows.
- CI gating: helps after the fact, but doesn’t make the agent run itself reproducible or the merge plan explainable.


## Insights and Differentiation: What is unique about your work?

Key differentiators (especially relative to Sculptor/Conductor/Superconductor/Graphite):

- Control plane, not just a UI: daemon-backed system that standardizes variants, runs, artifacts, and config precedence across repos.
- Truly tool-agnostic (workspace-first): instead of embedding a single “agent harness” as the primary way you work, LazyAgent gives you a prepared variant (worktree + env) you can enter with a shell and open in any IDE, then run your preferred tools directly (e.g., Claude Code, Codex, OpenCode, Aider, Cursor/VS Code workflows) at full fidelity.
- Artifact-first transparency: context packs (with provenance and budgets), memory (decisions/assumptions/invariants with approval-gated candidates), run logs, patch bundles, and merge plans are explicit, inspectable files.
- Local-first and offline-capable by default: core workflows run entirely on the developer machine (in contrast to cloud-first orchestration models).
- Capability/config management as a first-class product surface: canonical registry of skills/prompts/policies with role profiles, tags/groups, per-project overrides, and on/off toggles, plus export plugins to generate tool-specific configs (rather than “install/list/remove” only).
- Environment drift becomes a first-class workflow: some competitors focus on “containers per agent” or “snapshots per run”; LazyAgent additionally treats the environment definition as code, journals drift, and supports promoting/reconciling changes into a stable-point `spec.json` (with overlays when changes are intentionally temporary).
- Merge safety as a product surface: competitors generally provide merge UX (diff viewers, conflict help, PR creation, merge queues); LazyAgent emphasizes deterministic merge plans as artifacts (patch bundles + ordering + rationale) and verification gates by default, with advanced/agentic merge modes kept opt-in and bounded by a scope ladder.
- Protocol direction: the long-term goal is to document stable schemas (context packs, variant metadata, skill manifests, memory format) so tool integration can converge on shared formats rather than bespoke adapters.


## Your prototype: Tech Stack, architecture (expand in Technical Requirements section below)

### Tech stack (MVP)
- Python 3.12+ (uv-managed), ruff, pytest, ty.
- Daemon/API: FastAPI + uvicorn (local RPC; Unix socket preferred).
- UI client (optional, later): simple GUI or TUI for dashboards/PTY.
- CLI (primary): Typer.
- Sidecar bridge (containers): lightweight Python process for PTY streaming + status.
- Integrations: git CLI for worktrees/patches; Docker/devcontainer/compose for environments; optional VS Code/Cursor extension in TypeScript.

### Architecture (high level)
- Daemon is the source of truth for workspace state (projects, variants, runs, artifacts).
- CLI is the primary client; UI clients are optional later and attach via the same daemon.
- Variants are git worktrees with metadata.
- Context packs and memory live in `.agentplane/` and are linked to runs.
- Capabilities are canonical and exported to tool-specific formats via plugins.
- Merge planner produces patch bundles + merge plan + gate evidence.


## Product Metrics: How will you evaluate progress towards building your product?

Metrics should track whether LazyAgent makes agent work safer and more repeatable without slowing users down.

### Adoption and activation
- Time-to-first-success: minutes from install to “fork -> run tool -> capture patch bundle”.
- Activation rate: % of onboarded repos that create at least one variant and record at least one run.

### Repeatability and auditability
- Provenance coverage: % of runs with a locked context pack and stored run summary.
- Repro rate: % of runs that can be replayed in the same variant/env profile without manual fixes.

### Merge confidence
- Gate pass rate on integration variants (lint/test/build).
- Conflict rate and resolution time: frequency of conflicts when applying patch bundles + median time to resolution.
- Merge plan acceptance rate: % of merges applied without manual plan edits.

### Environment stability
- Drift incidence: frequency of detected drift events per variant.
- Drift promotion success: % of promoted env changes that rebuild and pass gates.

### Capability portability
- Export success rate per tool plugin (OpenCode/Cursor/etc.).
- Role reuse rate: # of repos using shared role profiles without per-repo customization.

### User-perceived value
- Qualitative: user feedback on trust ("I know what the agent used") and speed ("parallel work doesn’t break my env").
- Retention proxy: repeated weekly usage (variants created, runs executed, merges completed).
