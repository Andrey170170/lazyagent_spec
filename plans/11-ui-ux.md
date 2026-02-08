# UI/UX design notes (deferred; UI TBD)

CLI is the primary interface for all workflows. UI work is deferred until after CLI validation.
This doc sketches a potential dashboard experience; the eventual client may be a simple GUI,
TUI, or a VS Code/Cursor extension.

## Mock

![main project page mockup](../images/main_page.png)

![pty session mockup](../images/pty_session.png)

[figma make mockup](https://www.figma.com/make/EeKVsngadEeoWTRRdP5CRB/TUI-Project-Management-App?t=t56ebnKEv8vVGpIe-1)

## Goals
- CLI-first: every workflow is available in the CLI; UI focuses on visibility and speed if built.
- Fast, keyboard-driven workflow for managing projects and variants.
- Keep main screen responsive; long tasks show in sidebar spinner.
- Make state visible: variants, context packs, env status, merge plan.
- Provide a clear path to open tool sessions (embedded PTY).

## Personas
- Solo builder: wants quick forking, visible context, low overhead.
- Tech lead: wants audit trail, repeatability, and merge confidence.
- Experimenter: likes rapid switching between variants and runs.

## UX principles
- Keyboard first, mouse optional.
- CLI parity: never require a UI for core workflows.
- Single source of truth: sidebar lists projects and variants.
- Avoid modal lockups; show non-blocking progress.
- Explicit state labels: RUNNING, READY, ERROR, BUILDING.
- Optimistic UI with clear rollback paths.

## Information architecture
- Projects
  - Overview
  - Variants
  - Runs
  - Context packs
  - Environments
  - Merge plans
  - Settings

## Layout
- Left sidebar: projects and variants tree, status indicators.
- Main pane: detail view for the selected item.
- Right drawer (optional): logs, context pack, or diff preview.
- Status bar: active role, env profile, tool session, hints.

## Status indicators (ASCII)
- [READY] variant built and idle.
- [RUNNING] tool session active.
- [BUILD] initializing env or container.
- [ERROR] failed build or run.
- [WARN] drift detected or env overlay active.
- Spinner: "[BUILD] ..." shown in sidebar without blocking main view.

## Key views

### Project list
- List registered projects with last-opened timestamp.
- Quick actions: open, fork, settings, remove.
- Search: fuzzy project search with filters.

### Project overview
- Summary: default role, env profile, active variants, last run.
- Buttons: Fork, New Variant, Add Role, Add Env.

### Variant list
- Table of variants with status, base ref, role, env, last run.
- Actions: open session, archive, delete, build env.
- Search: fuzzy variant search with filters (status, role, env).

### Session view (embedded PTY)
- Full-height PTY pane for OpenCode/Cursor CLI.
- Header shows active variant and tool.
- Toggle: detach to external terminal if needed.

### Context pack inspector
- List of items with provenance (path, reason, size).
- Actions: pin, exclude, reorder, rebuild pack.

### Memory view
- Tabs: Decisions, Assumptions, Invariants, Candidates.
- Candidate approval with one-click promote.
- Conflict indicator when new entries collide with existing decisions.
- Restart prompt after promotion (with auto-restart toggle).

### Environment view
- Env spec summary: base image, features, packages.
- Drift summary: detected changes with promote/discard actions.
- Build history and last image digest.

### Ports view
- Logical to actual port mappings per variant.
- Copy-to-clipboard URLs.
- Quick switch for side-by-side comparisons (future proxy remap).

### Merge plan view
- Ordered list of patch bundles.
- Env spec merge summary (added/removed packages, file overrides).
- Run gates: lint/test/build status.
- Final action: apply merge or abort.
- Optional pattern selection for idea-level merges.

## Key flows

### Create project (UI, optional)
1. Create Project -> choose source: new, from template, or create as project.
2. If template or create-as: review prefilled choices and optionally edit.
3. Select runtime (native/Docker), tool packs, and agent tool configs.
4. Confirm and scaffold; project is registered automatically.
5. Prompt to save as reusable template (default: no).

### Add project (UI, optional)
1. Add Project -> enter Git URL or local path.
2. If URL, clone into managed projects root.
3. Initialize workspace metadata.
4. Sidebar shows project with [BUILD] spinner if env init runs.

### Fork variant
1. Select project -> Fork.
2. Enter variant name.
3. Create worktree and start env build.
4. Sidebar shows [BUILD] spinner; main view remains usable.
5. On ready, user can open session (embedded PTY).

### Promote env changes
1. Env view shows drift summary.
2. Select changes to promote.
3. Regenerate spec + rebuild image in background.
4. Sidebar shows [BUILD] spinner; state updates on completion.

### Approve memories
1. Badge indicates new candidates.
2. Open Memory view and review candidates.
3. Approve/reject; conflicts must be resolved.
4. Prompt to restart sessions to apply updates.

## Keyboard shortcuts (initial set)
- `p`: project switcher.
- `v`: variant list.
- `f`: fork variant.
- `s`: open session.
- `c`: context pack inspector.
- `e`: environment view.
- `m`: merge plan view.
- `?`: help.

## Error handling
- Show errors inline with short reason and retry action.
- Persist error logs in run artifacts and link to them.
- Provide safe fallback: switch to native run if container fails.

## Accessibility
- High-contrast theme option.
- Configurable keybindings.
- Avoid color-only status signals; always use labels.

## VS Code/Cursor extension UX
- Sidebar tree mirrors projects/variants.
- Actions: fork, open in devcontainer, view status.
- Uses same daemon API; no duplicate state.
- Consider this as the first UI surface if a standalone GUI/TUI is deferred.

## MVP scope
- CLI provides full workflow coverage (init, fork, run, context, merge).
- No UI required for MVP; UI decisions happen after CLI validation.
- If a UI is built later, start with read-only status and session switching.

## Future directions (Codex-inspired UI/UX, not scheduled)
These are optional, resource-heavy ideas to revisit after core CLI value is
proven. They should not block MVP milestones.

### Open in editor (multi-editor launcher)
- Add a compact "Open in" control on the variant header and project overview.
- Show a short menu of detected editors (VS Code, Cursor, JetBrains, Zed, etc.)
  plus a fallback "System default".
- Remember the last editor used per project in `.lazyagent/workspace.json`.
- If a variant uses a worktree, always open that worktree path, not repo root.
- Show a small "external" indicator to communicate that the UI is handing off
  control to the editor.

### Git surface (diff, stage, commit, PR)
- Right-side diff panel with file list, status (modified, untracked, staged),
  and inline comments for agent fixes.
- Stage/revert chunk actions should map directly to the worktree, not the base
  branch.
- Commit form should be scoped to the current variant and respect merge gates
  (if gates fail, show a clear block + link to logs).
- "Create PR" action should export patch bundles + merge plan metadata and
  open a lightweight PR form (or hand off to CLI/gh).
- Treat diff/commit as a convenience layer on top of patch bundles, not a
  separate workflow.

### Action buttons (quick commands)
- Allow per-project action buttons ("Run", "Test", "Lint", "Build") that
  launch in the embedded terminal and stream logs to the UI.
- Actions should be defined once in `.lazyagent/workspace.json` and run within
  the active variant's environment.
- Show status badges (running, success, failed) tied to run artifacts.

### Live update controls (hot config)
- Show config update scope per change: hot, next-run, or restart-required.
- Display current config version and last applied timestamp in project settings.
- Provide one-click rollback to last known-good config snapshot.
- Use non-blocking toasts for apply success/failure with links to apply logs.

### Codex-inspired polish
- Command palette for fast navigation (projects, variants, actions).
- Diff panel toggle and integrated terminal toggle on the main header.
- Optional "review mode" that highlights uncommitted changes for inspection.

### Kanban board view (parallel agents)
- Alternate view that surfaces tasks as cards with status columns.
- Cards link to variants/runs and expose quick open actions into the coding UI.
- WIP limits and blocker badges to keep parallel work manageable.
- Swimlanes by agent or by variant to show concurrency at a glance.

### Companion observer view (mobile-first, read-only)
- Optional companion web app with glanceable run status and step progress.
- Notification-style summary card (for example, "indexing workspace 5/7").
- Tap-through into a read-only terminal stream with scrollback and run summary.
- Observer mode must never interrupt active runs; lifecycle stays daemon-owned.
