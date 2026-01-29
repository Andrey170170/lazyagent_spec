# Local-First Agent Workspace and Orchestration Layer

This repository contains the product and technical specification for a local-first control plane that makes agentic software development predictable, inspectable, and scalable. The system standardizes variants (git worktrees), environments, capabilities, context packs, memory, and merge plans across multiple agent tools.

## What this is
- A spec-first project: the primary output is the documentation under `plans/`.
- A local-first workflow manager with a TUI, daemon, and sidecar model.
- Tool-agnostic core with first-class adapters for OpenCode and Cursor.

## Key decisions (current)
- Python-first MVP (Textual TUI, FastAPI daemon, Typer CLI); Go rewrite later if needed.
- Daemon is the source of truth; clients attach from any directory.
- Env specs are canonical (`.agentplane/env/spec.json`), devcontainer configs are generated.
- Memory uses Markdown with frontmatter; candidate memories require approval.
- Port management allocates ports at startup; dynamic remap via proxy is future work.

## Spec index
- `plans/00-overview.md`
- `plans/01-requirements.md`
- `plans/02-architecture.md`
- `plans/03-workflows.md`
- `plans/04-technical-validation.md`
- `plans/05-roadmap.md`
- `plans/06-risks-questions.md`
- `plans/07-architecture-diagram.md`
- `plans/08-storage-config.md`
- `plans/09-tech-stack.md`
- `plans/10-environments.md`
- `plans/11-ui-ux.md`
- `plans/12-capabilities-management.md`
- `plans/13-memory.md`
- `plans/14-merging.md`
- `plans/15-monetization.md`

## Working notes
- This repo is intentionally small; code will be added after the spec is validated.
- See `AGENTS.md` for coding guidelines and tool usage.
