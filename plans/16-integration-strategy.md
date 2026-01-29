# Integration Strategy

## Two Types of "Adapters" - Important Distinction

### Config/Skill Export Adapters (Good - We Build These)
Converting LazyAgent's canonical skills, prompts, and configs into tool-specific formats:
- Export to `.cursorrules` for Cursor
- Export to `opencode.json` and `.opencode/*` for OpenCode
- Export to Claude Code's skill format
- Export to Aider's conventions

**Why this is stable:**
- These are documented config file formats
- Tools expect users to provide these files
- Format changes are versioned and announced
- This is exactly the problem we're solving: one source, multiple outputs

### Deep Behavioral Integration (Risky - We Avoid This)
Hooking into tool internals, intercepting behavior, depending on undocumented APIs:
- Injecting into tool's runtime
- Depending on internal data structures
- Monkey-patching tool behavior
- Relying on undocumented CLI flags

**Why this is risky:**
- Breaks on minor tool updates
- No leverage over tool internals
- Maintenance nightmare

**Key insight:** Exporting a `.cursorrules` file is not the same as trying to intercept Cursor's behavior. The former is stable; the latter is fragile.

---

## The Layered Approach

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
│  Layer 1: CLI + TUI Foundation          │  ← Core platform
│  Workspace, env, context, merge         │
└─────────────────────────────────────────┘
```

---

## Layer 1: CLI + TUI Foundation

**The core platform that everything builds on.**

### Components
- **CLI**: `lazyagent fork`, `run`, `merge`, `status`, etc.
- **TUI**: Dashboard for variants, runs, context, and sessions
- **Daemon**: Source of truth for workspace state

### What it manages
- Workspace lifecycle (create, switch, archive, delete variants)
- Environment setup (devcontainers, compose, native)
- Context packs with provenance
- Memory (decisions, assumptions, invariants)
- Patch bundles and merge flows
- Port allocation and resource management

### Tool-agnostic by default
Even without any export plugins, users can:
```bash
lazyagent fork feature-x
cd $(lazyagent path feature-x)
# Run ANY tool - it just works in the prepared environment
opencode  # or cursor, or claude, or aider, or vim + copilot...
lazyagent merge feature-x
```

---

## Layer 2: Config/Skill Export Plugins

**Convert canonical LazyAgent configs to tool-specific formats.**

This is the "adapt the project for work with X tool" feature.

### How it works
1. LazyAgent maintains canonical skill/config definitions
2. Export plugins transform to tool-specific formats
3. Plugins are like format converters, not behavioral hooks

### Supported exports (MVP and beyond)
| Tool | Export Format | Notes |
|------|--------------|-------|
| OpenCode | `opencode.json`, `.opencode/*` | First-class |
| Cursor | `.cursorrules`, `.cursor/rules/*` | Stable format |
| Claude Code | Skills directory, CLAUDE.md | Documented |
| Aider | `.aider.conf.yml`, conventions | Stable format |
| Generic | Markdown prompt blocks | Fallback |

### Plugin architecture
```
canonical/
  skills/
    code-review.yaml
    testing.yaml
  policies/
    security.yaml

↓ Export plugins ↓

.cursorrules          (Cursor)
.opencode/skills/*    (OpenCode)
.claude/skills/*      (Claude Code)
```

### Why plugins, not hardcoded adapters
- Users can add new export targets
- Community can contribute plugins
- Each plugin is isolated - one breaking doesn't affect others
- Clear interface: canonical format in, tool format out

### What plugins DON'T do
- Hook into tool behavior
- Depend on tool internals
- Require tool-specific runtime integration

---

## Layer 3: OpenCode Deep Integration

**Go beyond config export - actual collaboration with an OSS project.**

### Why OpenCode specifically?
1. **Open source**: We can contribute, not just consume
2. **Extensible by design**: Skills, hooks, and config are first-class
3. **We use it daily**: Dogfooding drives real integration needs
4. **Aligned incentives**: Both projects benefit from collaboration

### What "deep integration" means here
- Contribute features upstream that benefit both projects
- Co-design APIs where LazyAgent and OpenCode interact
- Shared understanding of context/memory formats
- Native support in OpenCode for LazyAgent workspaces (potentially)

### This is collaboration, not adaptation
We're not building a fragile adapter that breaks on updates.
We're building a relationship where both tools evolve together.

### Upstream contribution examples
- If OpenCode needs better worktree support → we contribute that
- If LazyAgent's context format is useful → propose as OpenCode feature
- Shared skill format that both tools understand natively

---

## Layer 4: Emergent Protocol

**Standards emerge from usage, not committees.**

### How it happens
1. Layers 1-3 usage reveals stable patterns
2. We document what works as formal schemas
3. Other tools can adopt specs if they choose
4. We maintain docs, not behavioral adapters

### What gets standardized
- **Context pack format**: JSON schema for context snapshots
- **Variant metadata**: Schema for workspace state
- **Skill manifest**: Format for portable skill definitions
- **Memory format**: Markdown + YAML frontmatter spec

### Integration guide for tool authors
Published documentation:
- "How to read LazyAgent context packs"
- "How to export skills in LazyAgent format"
- "How to integrate with LazyAgent workspaces"
- Example implementations and reference plugins

---

## Implementation Timeline

| Phase | Layer 1 | Layer 2 | Layer 3 | Layer 4 |
|-------|---------|---------|---------|---------|
| 0 | CLI + TUI skeleton | - | - | - |
| 1 | Core features | OpenCode export | Begin collaboration | - |
| 2 | Containers, env | Cursor export | Deepen integration | - |
| 3 | Polish, stability | More plugins | Upstream contribs | Document schemas |
| 4+ | Maintenance | Community plugins | Co-evolution | Publish specs |

---

## Trade-offs

### Layer 2 (Export Plugins) trade-offs
- **Pro**: Works with many tools via stable config formats
- **Con**: Limited to what config files can express
- **Mitigation**: Layer 3 shows what's possible with deeper integration

### Layer 3 (OpenCode Deep) trade-offs
- **Pro**: Maximum integration depth with one tool
- **Con**: OpenCode-specific effort
- **Mitigation**: Layer 2 still works for other tools; Layer 4 opens the door

### Why not deep integration with every tool?
- Each deep integration is a significant investment
- Better to go deep with one OSS tool than shallow with many
- Config export (Layer 2) covers the common cases well
- If other tools want deep integration, they can follow Layer 4 specs

---

## Summary

| Layer | What | Stability | Effort |
|-------|------|-----------|--------|
| 1 | CLI + TUI | Core platform | High (we own it) |
| 2 | Config export | Stable (documented formats) | Medium per plugin |
| 3 | OpenCode deep | Collaborative (OSS) | High (relationship) |
| 4 | Protocol | Emergent (docs) | Low (documentation) |

The key insight: **config/skill export is not the same as behavioral integration**. We do the former liberally (Layer 2), the latter selectively (Layer 3 with OpenCode only), and let standards emerge naturally (Layer 4).
