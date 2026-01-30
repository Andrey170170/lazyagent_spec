# Requirements

## Target users
- Solo builders using multiple agent tools across projects.
- Team leads who want consistent agent behavior and audit trails.
- Security-conscious teams needing strong isolation and clear permissions.

## Primary user goals
- Spin up parallel agent work without environment collisions.
- Know exactly what context an agent used for any run.
- Reuse skills and configs across tools and machines.
- Merge changes safely with clear intent and verification evidence.

## Functional requirements

### Project onboarding
- CLI-first flow to register or clone projects; UI optional.
- Optional Git URL input to clone into a managed projects root.
- Initialize workspace metadata automatically after clone.

### Project creation wizard (CLI-first)
- Create new projects via a guided wizard (language, package manager, location).
- Select runtime profile (native or Docker for MVP; more providers later).
- Pick tool packs (e.g., eza, fzf) and agent tools from a registry.
- Scaffold baseline docs/configs and record decisions for later recall.
- Store the initial project blueprint locally for reuse ("create as X").
- Wizard can start from a reusable template or an existing project.
- Prompt at the end to save as a reusable template (default: no).
- Provide a separate utility to save a named template from a project.

### Variants (project state plane)
- Create, switch, list, archive, and delete variants quickly.
- Variant metadata: owner/agent, base commit, purpose, timestamps.
- Compare variants by diff, affected modules, test status.
- Support short-lived and long-lived variants with clear lifecycle states.

### Environments (runtime and safety plane)
- Attach an environment profile to each variant.
- Support devcontainer or Docker Compose profiles.
- Enforce resource limits and network restrictions when configured.
- Capture command history, stdout/stderr, and artifacts per run.
- Sidecar bridge in container for PTY streaming and session status.

### Capabilities (capability plane)
- Canonical registry of skills, prompts, hooks, and policies.
- Role profiles that activate subsets of skills and permissions.
- Tool adapters that export capabilities into tool-specific formats.
- MVP first-class targets: OpenCode and Cursor; add Claude Code later.
- Visibility: show what is active for a given variant and role.

### Context and memory (context plane)
- Build context packs as explicit artifacts with provenance.
- Allow user review, pin, exclude, and budget allocation controls.
- Store long-term project facts and decisions in a versioned store.
- Link each run to its exact context pack.
- Generate candidate memories and require approval before promotion.

### Integration and merging (merge plane)
- Agentic merge assistance with stated, deterministic merge plans.
- Export changes as patch bundles with metadata and test evidence.
- Provide deterministic merge plans with clear ordering rules.
- Require verification gates before integration when configured.

### Orchestration (workflow plane)
- Declarative workflow steps: planner, worker, reviewer, tester.
- Run steps in parallel with isolated variants.
- Persist artifacts per step: patch bundle, context pack, run logs.

### UX and sessions
- CLI provides fork flow with prompts and non-blocking progress output.
- Optional UI client (GUI/TUI/extension) may add dashboards and quick switching after CLI validation.
- VS Code/Cursor extension integration for container attach and observability.

## Non-functional requirements
- Local-first: all core features work offline.
- Auditable: run logs and context packs are stored locally.
- Reproducible: variants and env profiles are deterministic.
- Secure by default: least privilege, explicit permissions.
- Extensible: new tools and role profiles are easy to add.
- Performant: fast variant creation and diff visualization.

## Constraints and assumptions
- Works with existing git repos without restructuring.
- Must not require cloud services for core functionality.
- Should tolerate mixed tool usage (CLI agents, IDE agents).
- Minimal setup friction: bootstrap with a single init step.

## Out of scope (initially)
- Cloud-hosted multi-tenant control plane.
- Fully autonomous agent swarms without user oversight.
- Replacing IDEs or editor workflows entirely.
