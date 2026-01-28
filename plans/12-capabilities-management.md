# Capability and config management

## Goals
- Full transparency of active skills/configs per session.
- Managed entities for skills, plugins, and MCP servers.
- Searchable, taggable, and groupable capability sets.

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
