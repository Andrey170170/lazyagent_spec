# LazyAgent Project Report

## System Description and Architecture

LazyAgent is a local-first control plane for agent-driven development that makes parallel work predictable, inspectable, and safe to integrate. It standardizes how projects are onboarded, how parallel workspaces ("variants") are created and isolated, how tool capabilities are managed and exported across agent tools, how context/memory is captured as explicit artifacts, and how changes are merged with deterministic plans and verification gates.

### High-level components
- Daemon (local source of truth): manages workspace registry, variants, runs, adapters, and artifacts.
- CLI + TUI clients: attach to the daemon from any directory; TUI is keyboard-first and can embed tool sessions via PTY.
- Variant manager: uses git worktrees to create parallel workspaces with metadata and lifecycle states.
- Run executor: runs commands in native or containerized environments; captures stdout/stderr and artifacts.
- Environment manager + sidecar bridge (containers): builds/attaches devcontainers/compose, streams PTY, detects drift, allocates ports.
- Capability manager: canonical registry of skills/prompts/policies; role profiles select subsets; export plugins generate tool-specific configs.
- Context + memory managers: context packs with provenance and budgets; durable memory files with approval-gated candidates.
- Merge planner: patch bundles + deterministic merge plans; optional agentic merge modes with guardrails.

### Storage model (restartable, local)
- Global store: `~/.config/agentplane/` (projects registry, reusable skills/profiles/adapters).
- Per-project store (git-ignored): `<repo>/.agentplane/` (workspace metadata, variants, env specs/journal, context packs, memory, run logs).
- Generated tool configs (per variant worktree root): e.g. `opencode.json`, `.opencode/*`, `.cursor/*`.

### Integration strategy
- Layer 1 (foundation): CLI/TUI + daemon work with any agent tool via standard workspace/env/context artifacts.
- Layer 2 (export plugins): convert canonical configs/skills to tool formats (OpenCode, Cursor, Claude Code, Aider, generic).
- Layer 3 (deep OpenCode collaboration): co-evolve with one OSS tool where deeper integration is worth it.
- Layer 4 (protocol): document stable schemas (context packs, variant metadata, skill manifests, memory format).


## Functional Requirements: User Stories, Walkthroughs, Feature List

### Target users
- Solo builders using multiple agent tools across projects.
- Team leads who want consistent agent behavior and audit trails.
- Security-conscious teams needing isolation and explicit permissions.

### User stories (representative)
- As a user, I can fork multiple variants for parallel agent work without environment collisions.
- As a user, I can see exactly what context an agent used for any run (files/snippets/notes, provenance, budgets).
- As a user, I can reuse skills/configs across tools and machines and know what is active for a session.
- As a user, I can export changes as patch bundles with metadata and test evidence.
- As a user, I can merge parallel work safely using an explicit merge plan and verification gates.

### Walkthroughs (end-to-end)
- Workspace init: detect repo + main branch; create default role/env; register baseline facts.
- Onboard project (TUI): add Git URL/local path; clone (if needed); init; show in sidebar.
- Fork variant (TUI/CLI): create git worktree from base ref; attach role/env; build env in background.
- Run work: execute commands; capture run logs and artifacts; lock a context pack for the run.
- Memory formation: generate candidate memories from runs; user approves/rejects; promote to durable memory; restart prompt.
- Merge: generate merge plan; reconcile env specs/overlays; apply patch bundles in order (3-way fallback); run gates; archive variants.

### Feature list (by area)
- Variants: create/switch/list/archive/delete; metadata (base ref, purpose, role/env, timestamps); diff/test status comparisons.
- Environments: native/devcontainer/compose profiles; optional resource/network restrictions; port allocation mapping; drift detection + promotion.
- Capabilities: canonical registry (skills/plugins/MCPs/policies); role profiles; active-config visibility; export plugins to tools.
- Context packs: select/rank/review/lock pipeline; provenance (path/reason/time); budgets; run linkage by hash.
- Memory: decisions/assumptions/invariants/candidates; approval gate; conflict detection and resolution.
- Runs: command history; stdout/stderr capture; artifact retention; session manager with embedded PTY.
- Merging: patch bundles (diff or commit-based); deterministic merge plans; verification gates; optional agentic merge modes with guardrails.


## Non-functional Requirements: Performance, Reliability, Security, Usability, Quality

### Performance
- Fast variant creation via git worktrees.
- Incremental diff/indexing and lazy loading to handle large repos and many variants.
- Background env builds; UI remains responsive (sidebar spinner) and avoids blocking.

### Reliability
- Restartable daemon: reconcile registry/projects/worktrees on startup; flag stale entries.
- Durable local artifacts: runs/logs/context/memory stored in `.agentplane/` for post-mortem and audit.
- Clear lifecycle states for variants and runs; explicit archive/remove flows.

### Security
- Local-first operation; no cloud dependency required for core functionality.
- Least privilege via role profiles and command allowlists; optional network/resource restrictions.
- Run logs/context packs treated as sensitive: secret redaction rules + optional secret scanning.
- Local auth model: prefer Unix socket + file permissions for daemon access.

### Usability
- Keyboard-first TUI with quick session switching and non-blocking progress.
- Explicit state labels and inspectable artifacts (active config, context pack inspector, merge plan view).
- Tool-agnostic default behavior; export plugins add convenience without locking users in.

### Quality
- Spec-first development: plans live under `plans/` and drive implementation.
- Guardrails: merge/refactor automation requires plans, rationale, and gates.
- Deterministic configuration precedence (global -> project -> role -> variant -> generated).


## System Operations: Maintenance, Monitoring, Logging, Backup, Recovery

### Maintenance
- Version export plugins and keep a compatibility matrix per tool.
- Provide explicit cleanup/prune operations for archived variants, old runs, and large artifacts.
- Document supported git edge cases (submodules, sparse checkout) for the MVP.

### Monitoring (local)
- Daemon health endpoints (and/or a `lazyagent status` command) for:
  - running variants and sessions
  - recent run outcomes
  - env build state and drift warnings
  - storage usage/retention warnings

### Logging
- Per-run logs captured under `.agentplane/runs/<run-id>/` (stdout/stderr, summaries, artifacts).
- Daemon logs stored locally with clear log levels and optional structured JSON.
- Redaction rules applied before logs are persisted when configured.

### Backup and recovery
- Primary recovery mechanism: git remains the canonical code history.
- Backup scope for LazyAgent state:
  - Global: `~/.config/agentplane/`
  - Per project: `<repo>/.agentplane/` (git-ignored, but important for audit artifacts)
- Recovery flow: restore stores; daemon reconciles projects/worktrees/containers; stale entries flagged for repair/removal.


## Deployment, Testing, Validation

### Deployment model (local)
- Distribution targets: local daemon + CLI + TUI; optional container-side sidecar.
- Environment support: native first; devcontainers/compose when available.
- Editor integration: optional VS Code/Cursor extension to attach to variant environments.

### Testing strategy
- Unit tests for core managers (variants, context pack builder, merge planner) and schema validation.
- Integration tests for git worktree operations and patch bundle apply flows.
- End-to-end manual validation loop per roadmap phase (fork -> run -> bundle -> merge -> gates).

### Validation references
- Git worktrees, patch formats, and apply/am flows are validated against upstream git docs.
- Devcontainer behavior validated against the containers.dev specification; resource limits validated against Docker docs.
- OpenCode export target validated against OpenCode config docs.


## Legal and Compliance Requirements

This project is designed to minimize compliance burden by keeping core operation local-first and offline-capable; however, several requirements still apply depending on usage.

### Data privacy and handling
- Treat run logs, context packs, and memory files as potentially sensitive (may include proprietary code, secrets, or personal data).
- Provide clear retention controls (time-based and size-based) and redaction/secret scanning options.
- If telemetry is ever added, default to local-only and require explicit opt-in for any outbound data.

### Third-party services and credentials
- BYO keys assumption: users supply their own LLM/provider keys; never store keys in plaintext logs; avoid echoing secrets in outputs.
- Document where secrets can appear (env vars, tool configs, run logs) and provide guidance for safe usage.

### Open source licensing
- Track licenses of bundled dependencies (Python libs, Textual, FastAPI, Typer, etc.).
- If shipping templates/snippets, ensure their licenses are compatible and attribution is included where required.

### Enterprise compliance considerations (future)
- Policy enforcement and audit exports for regulated environments.
- Optional redaction profiles aligned to common controls (SOC 2-style operational practices, not a certification claim).


## Risk: Identified Risks, Mitigation, Contingencies

### Key risks
- Tool adapter drift: tools change config formats.
- Git worktree edge cases: submodules/sparse checkout complexity.
- Environment parity: devcontainers/compose do not match all stacks.
- Performance: large repos + many variants slow diffing/indexing.
- Trust: users hesitate to apply automation-generated patches.
- Agentic merge correctness: conflict handling and plan quality.
- Security: logs/context packs leak secrets.
- Sidecar complexity: PTY streaming compatibility.
- Editor integration drift: VS Code/Cursor API changes.
- Python performance/packaging limits as scope grows.

### Mitigations
- Adapter versioning + compatibility matrix; keep Layer 1 tool-agnostic fallback viable.
- Document MVP limits; add explicit checks and safe failure modes.
- Native env fallback when containers are unavailable.
- Incremental diff/indexing + lazy loading; background processing.
- Verification gates + dry-run modes + explainable merge plans.
- Redaction rules + optional secret scanning for artifacts.

### Contingencies
- If a tool export breaks: disable that plugin version, keep canonical config intact, ship a fixed exporter.
- If containers are unreliable on a host: run in native mode and defer isolation features.
- If Python becomes a bottleneck: keep RPC contracts stable and migrate daemon/sidecar to Go post-MVP.
- If advanced merge modes are not trusted: keep deterministic baseline merge as default; make agentic modes opt-in.


## Timeline and Milestones

Milestones follow the layered integration strategy and the phased roadmap.

### Phase 0: Scaffolding and vertical slice (Layer 1)
- Deliver: `init`, `fork`, `list`; worktree-backed variants; native run executor with logs; diff-based patch bundle export/apply; daemon skeleton.
- Exit criteria: fork a variant, run any tool inside it, export a patch bundle.

### Phase 1: MVP user loop (Layer 1 complete, Layer 2 starts)
- Deliver: TUI onboarding; embedded PTY (native); baseline merge with gates; static port allocation mapping; minimal memory store; OpenCode export plugin.
- Exit criteria: onboard -> fork -> run agent -> merge safely with gates.

### Phase 2: Containers + sidecar + env spec
- Deliver: container runner (bind-mount worktrees); sidecar for PTY + status; env spec/journal/overlays; drift detection + promotion; template selection.
- Exit criteria: containerized variants with auditable env changes.

### Phase 3: Capabilities, context, and memory UX (Layer 2 matures; Layer 3/4 begin)
- Deliver: capability registry + role toggles; context pack builder (provenance/budgets); memory candidate approvals; more export plugins; begin OpenCode deep collaboration; start documenting schemas.
- Exit criteria: repeatable roles, transparent context, managed skills across tools.

### Phase 4: Agentic merge modes + refactor
- Deliver: rebase-and-repair; intent-slice merge; pattern merge via refactor variant; scope ladder guardrails; env refactor suggestions.
- Exit criteria: advanced merges add value without sacrificing auditability.

### Phase 5: Workflow orchestration
- Deliver: declarative workflow runner (planner/worker/reviewer/tester); parallel dispatch to variants; multi-run dashboard; policy enforcement.
- Exit criteria: repeatable multi-agent pipelines with stored artifacts.

### Phase 6: Hardening and scale
- Deliver: crash recovery and pruning; packaging/distribution improvements; optional local-only telemetry hooks; evaluate Go rewrite targets.
- Exit criteria: stable daily-driver experience on real repos.
