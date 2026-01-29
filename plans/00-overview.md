# Local-First Agent Workspace and Orchestration Layer

## Purpose
Build a local-first control plane that makes agent-driven development predictable, inspectable, and scalable.
It standardizes workspaces, environments, capability sets, context packs, and merges across multiple agent tools.

## Inputs
- Preliminary summary from past chat: https://chatgpt.com/c/69753cf8-4a14-8332-9432-240da68ff6eb
- Repo intent: spec sheets and feasibility planning for a startup idea.

## Current summary
- Spec-first repository; primary output lives under `plans/`.
- Python-first MVP with Textual TUI, FastAPI daemon, Typer CLI, sidecar bridge.
- Daemon is the primary source of truth; clients attach from any directory.
- Canonical env spec in `.agentplane/env/spec.json`; devcontainer configs generated.
- Memory uses Markdown with frontmatter; candidates require approval.
- Port allocation at startup; dynamic remap via proxy is future work.

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
- Tool-agnostic core with adapters for popular agent tools.
- Minimal cognitive load: a single dashboard of variants, runs, and context.
- TUI-first UX with fast keyboard-driven workflows.
- Sidecar bridge for tool integration and container session status.

## Glossary
- Variant: a first-class workspace instance, usually backed by a git worktree.
- Role profile: a named set of skills, permissions, and allowed commands.
- Context pack: the exact files, snippets, and notes injected into a run.
- Patch bundle: a portable change set with metadata and test evidence.

## Scope framing
- "Max" vision: all planes (variant, runtime, capability, context, merge, workflow).
- MVP path: start with variants, context packs, and patch bundles, then layer isolation.
- First-class tool adapters in MVP: OpenCode and Cursor (add others later).
- MVP implementation: Python-first (Textual TUI), with possible Go rewrite later.

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
