# Local-First Agent Workspace and Orchestration Layer

## Purpose
Build a local-first control plane that makes agent-driven development predictable, inspectable, and scalable.
It standardizes workspaces, environments, capability sets, context packs, and merges across multiple agent tools.

## Inputs
- Preliminary summary from past chat: https://chatgpt.com/c/69753cf8-4a14-8332-9432-240da68ff6eb
- Repo intent: spec sheets and feasibility planning for a startup idea.

## Current summary
- Spec-first repository; primary output lives under `plans/`.
- Python-first MVP with daemon + CLI core (Typer) and sidecar bridge; UI clients deferred.
- Daemon is the primary source of truth; CLI is primary client; UI clients optional later.
- Canonical env spec in `.agentplane/env/spec.json`; devcontainer configs generated.
- Memory uses Markdown with frontmatter; candidates require approval.
- Port allocation at startup; dynamic remap via proxy is future work.
- Project creation wizard (CLI-first) with project-local config and optional reusable templates.

## Problem statement (condensed)
- Capability sprawl: each tool has its own config and skill folders.
- Opaque context: users cannot see or control what an agent actually used.
- Environment friction: parallel runs collide in complex stacks.
- Merge anxiety: parallel agent work increases drift and conflict.

## North star
Provide a unified, local-first workspace layer that turns agentic coding into a disciplined, auditable workflow.

## Guiding principles
- Local-first and offline-capable by default; no cloud dependency required.
- Explicit over implicit: context, capabilities, and permissions are inspectable artifacts.
- Isolation by default for parallel runs; safe merging with verification gates.
- Tool-agnostic foundation: CLI works with any agent tool out of the box; UI clients optional later.
- Config/skill export over behavioral hooks: stable format conversion, not brittle integrations.
- Deep collaboration where it matters: OpenCode as first-class OSS partner.
- Minimal cognitive load: a single status surface (CLI summaries; optional dashboard later).
- CLI-first UX with fast keyboard-driven workflows; UI requirements assessed after CLI validation.
- Sidecar bridge for tool integration and container session status.

## Glossary
- Variant: a first-class workspace instance, usually backed by a git worktree.
- Role profile: a named set of skills, permissions, and allowed commands.
- Context pack: the exact files, snippets, and notes injected into a run.
- Patch bundle: a portable change set with metadata and test evidence.

## Scope framing
- "Max" vision: all planes (variant, runtime, capability, context, merge, workflow).
- MVP path: start with variants, context packs, and patch bundles, then layer isolation.
- MVP implementation: Python-first daemon + CLI; UI clients evaluated post-MVP; possible Go rewrite later.

## Integration strategy (layered)

**Key distinction:** Config/skill export (converting to `.cursorrules`, `opencode.json`, etc.) is stable and valuable. Deep behavioral integration (hooking into tool internals) is risky. We do the former liberally, the latter selectively.

```
┌─────────────────────────────────────────┐
│  Layer 4: Protocol/Standard             │  ← Emerges from usage
│  Documented schemas, integration guides │
├─────────────────────────────────────────┤
│  Layer 3: OpenCode Deep Integration     │  ← OSS collaboration
│  Upstream contributions, co-evolution   │
├─────────────────────────────────────────┤
│  Layer 2: Config/Skill Export Plugins   │  ← Works with many tools
│  .cursorrules, opencode.json, etc.      │
├─────────────────────────────────────────┤
│  Layer 1: CLI-first Foundation          │  ← Core platform
│  Workspace, env, context, merge         │
└─────────────────────────────────────────┘
```

### Layer 1: CLI-first Foundation (Core Platform)
The base platform that everything builds on:
- **CLI (primary)**: `lazyagent fork`, `run`, `merge`, `status`, etc.
- **UI clients (optional, later)**: simple GUI, TUI, or VS Code extension for dashboards/sessions
- **Daemon**: Source of truth for workspace state
- Works with ANY tool even without export plugins

### Layer 2: Config/Skill Export Plugins
"Adapt the project for work with X tool" - converting canonical configs to tool formats:
- Export to `.cursorrules` for Cursor
- Export to `opencode.json` and `.opencode/*` for OpenCode
- Export to Claude Code's skill format
- Community can contribute new export plugins
- **Stable**: these are documented config formats, not behavioral hooks

### Layer 3: OpenCode Deep Integration (OSS Collaboration)
Go beyond config export with one OSS project:
- Contribute features upstream that benefit both projects
- Co-design APIs where LazyAgent and OpenCode interact
- Shared skill/context formats understood natively
- **Collaboration, not fragile adaptation**

### Layer 4: Protocol/Standard (Emergent)
As Layers 1-3 mature, patterns stabilize into standards:
- Context pack format becomes documented schema
- Skill manifest becomes reusable specification
- Other tools can integrate by following specs
- We maintain docs, not behavioral adapters

## Document map
- plans/01-requirements.md
- plans/02-architecture.md
- plans/03-workflows.md
- plans/04-technical-validation.md
- plans/05-roadmap.md
- plans/06-risks-questions.md
- plans/07-architecture-diagram.md
- plans/08-storage-config.md
- plans/09-tech-stack.md
- plans/10-environments.md
- plans/11-ui-ux.md
- plans/12-capabilities-management.md
- plans/13-memory.md
- plans/14-merging.md
- plans/15-monetization.md
- plans/16-integration-strategy.md
