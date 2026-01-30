# Capability and config management

## Goals
- Full transparency of active skills/configs per session.
- Managed entities for skills, plugins, and MCP servers.
- Searchable, taggable, and groupable capability sets.
- Canonical format with export plugins for multiple tools.

## Integration strategy alignment

This capability system follows the 4-layer integration approach.

### Key distinction
**Config/skill export** (converting to `.cursorrules`, `opencode.json`, etc.) is stable and valuable - it's exactly the problem we're solving. **Deep behavioral integration** (hooking into tool internals) is risky and we avoid it.

### Layer 1: CLI-first Foundation (UI optional)
- Capabilities stored in canonical format (YAML/JSON).
- CLI exposes capabilities via environment variables and file paths.
- Optional UI shows "Active Config" panel per session.
- Works with any tool even without export plugins.

### Layer 2: Config/Skill Export Plugins
- "Adapt project for work with X tool" feature.
- Export to `.cursorrules` for Cursor.
- Export to `opencode.json` and `.opencode/*` for OpenCode.
- Export to Claude Code's skill format.
- Plugin architecture: canonical in → tool format out.
- Community can contribute new export plugins.

### Layer 3: OpenCode Deep Integration
- OSS collaboration beyond config export.
- Skill sync: bidirectional where it makes sense.
- Hook integration: LazyAgent events ↔ OpenCode events.
- Upstream contributions: features that belong in OpenCode go there.

### Layer 4: Protocol/Standard
- Document capability manifest schema.
- Publish skill pack format specification.
- Integration guide for tool authors.

## Managed entities
- Skill: prompt + docs + tools/policies.
- Plugin: tool extensions or automations.
- MCP server: external capability provider.
- Policy: constraints (allowed tools, commands, paths).

Each entity has:
- `id`, `name`, `version`, `tags[]`, `group`, `source` (global/project).
- `enabled` flag (per project or per role).

## Capability registry
- Stored in global and project stores.
- Role profiles reference a list of entity IDs.
- Adapters export active entities into tool configs.

## Visibility and transparency
- UI shows "Active Config" panel per session:
  - role profile
  - enabled skills/plugins/MCPs
  - source layer (global/project/variant)
- Each item links to its definition and last updated time.

## Search and grouping
- Fuzzy search by name, tag, or group.
- Toggle individual items on/off.
- Toggle entire groups on/off.
- Filter to show only active items.

## Import and discovery (MVP)
- Local directory import.
- Git URL import (reads a manifest file).

## Import and discovery (later)
- Registry search for skills/plugins.
- Version pinning and auto-update options.

## Tool integration approach

### Any tool (Layer 1 - foundation)
- Capabilities exposed via file paths in environment.
- `LAZYAGENT_SKILLS_PATH`, `LAZYAGENT_CONTEXT_PATH`, etc.
- Tools that support custom prompts/context can read these paths.
- Works even without any export plugins installed.

### Export plugins (Layer 2 - config/skill export)

| Tool | Export Format | Notes |
|------|--------------|-------|
| OpenCode | `opencode.json`, `.opencode/*` | First-class |
| Cursor | `.cursorrules`, `.cursor/rules/*` | Stable format |
| Claude Code | Skills directory, CLAUDE.md | Documented |
| Aider | `.aider.conf.yml`, conventions | Stable format |
| Generic | Markdown prompt blocks | Fallback |

**Plugin architecture:**
- Canonical skill/config → export plugin → tool-specific format
- Each plugin is isolated; one breaking doesn't affect others
- Users/community can add new export targets

### OpenCode (Layer 3 - deep collaboration)
- Beyond config export: actual OSS collaboration.
- Memory files injected into OpenCode's context automatically.
- Role profiles map to OpenCode permission sets.
- We dogfood this daily; integration grows from real usage.
- Contribute features upstream when they belong in OpenCode.

### Protocol (Layer 4 - documentation)
- Document the capability manifest format.
- Publish integration guide with examples.
- Tools can adopt the spec if they choose.
- We maintain docs, not behavioral adapters.
