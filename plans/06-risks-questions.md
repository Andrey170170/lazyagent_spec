# Risks and open questions

## Key risks
- Tool adapter drift: agent tools change config formats frequently.
- Git worktree edge cases: submodules and sparse checkout complexity.
- Environment parity: devcontainers may not map to all stacks.
- Performance: large repos + many variants could slow diffing.
- Trust: users may hesitate to let automation apply patches.
- Agentic merge correctness: plan quality and conflict handling.
- Security: storing logs and context packs may leak secrets.
- Sidecar complexity: PTY streaming and tool compatibility.
- Editor integration drift: VS Code/Cursor APIs can change.
- Python performance or packaging limitations at scale.

## Mitigations
- Adapter versioning and compatibility matrix per tool.
- Clear limits: document unsupported git features in MVP.
- Allow native env fallback when containers are not available.
- Incremental diff/indexing and lazy loading of variant data.
- Merge gates and human approval before apply.
- Provide dry-run mode and explainable merge steps.
- Redaction rules and secret scanning for logs.

## Open product questions
- Primary user persona for MVP: solo devs or small teams?
- Which agent tools must be first-class in v1?
- Should the initial UI be CLI-only or include a lightweight dashboard?
- How strict should permission enforcement be by default?

## Open technical questions
- Preferred workspace metadata location and naming (.agentplane/ vs other).
- Long-term fact store format: JSONL, Markdown, or SQLite?
- Merge bundle format: raw patches, commits, or both?
- How to capture context budget consistently across providers.

## What's left to discuss
- Daemon + sidecar API protocol (endpoints, auth, reconnects).
- Data schemas for workspace/variants/runs/specs.
- Adapter mapping details for OpenCode/Cursor.
- Context pack selection and scoring rules.
- Agentic merge plan generation logic and failure modes.
- Environment build pipeline and caching strategy.
- VS Code/Cursor extension spec and attach flow.
- Permissions model (commands, paths, network).
- Storage retention/cleanup policies.
- Platform support and distribution strategy.
