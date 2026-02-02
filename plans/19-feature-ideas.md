# Feature ideas (researched)

This doc is a parking lot for feature ideas backed by quick research.
These items are not scheduled yet.

## 1) Beads task management integration

Sources:
- https://github.com/steveyegge/beads
- https://raw.githubusercontent.com/steveyegge/beads/master/README.md
- https://raw.githubusercontent.com/steveyegge/beads/master/docs/COMMUNITY_TOOLS.md

Notes:
- Beads is a distributed, git-backed graph issue tracker for agents.
- Tasks live as JSONL under `.beads/` with hash IDs to avoid merge collisions.
- The CLI is agent-optimized with JSON output, dependency tracking, and ready-task
  detection.
- Beads keeps a local SQLite cache and uses a background daemon for auto-sync.
- It supports workflows like stealth mode and protected-branch metadata.
- There is already an OpenCode plugin (`opencode-beads`) and a rich ecosystem of
  TUI/UI tools.

Opportunity and fit:
- A Beads adapter would make the task plane dependency-aware while staying
  local-first and git-native.
- The task graph can feed LazyAgent's context packs and merge planning with
  explicit blockers and ownership.

Integration sketch:
- Add an optional task backend: `lazyagent tasks init --backend beads`.
- Run `bd init` during project setup, with toggles for `--stealth` or
  protected-branch metadata.
- Provide wrapper commands that call Beads with `--json` and surface results in
  CLI or UI (ready list, blockers, ownership, status).
- Store a mapping between LazyAgent variants/runs and Beads issue IDs in
  `.lazyagent/` metadata.
- When a plan is created, auto-seed Beads issues for steps and link dependencies.
- Inject a short Beads summary (ready tasks, in-progress, blockers) into context
  packs for agent runs.

Risks and open questions:
- Should `.beads/` be committed in the main repo, a protected branch, or kept
  in stealth mode by default?
- How to avoid Beads auto-commit behavior conflicting with worktree-based flows?
- Do we need a minimal Beads schema adapter for non-git repos?

## 2) "Open in X" editor launcher (inspired by Codex app)

Sources:
- https://developers.openai.com/codex/app/features
- https://developers.openai.com/codex/app/local-environments
- https://developers.openai.com/codex/app/commands
- https://ss64.com/osx/open.html
- https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/start
- https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process
- https://manpages.ubuntu.com/manpages/jammy/en/man1/xdg-open.1.html

Notes:
- The Codex app includes multi-project management, worktree support, built-in
  Git tools, and an integrated terminal.
- Local environments allow defining per-project "actions" that appear as quick
  buttons in the app and run commands in the terminal.
- The app exposes an "Open folder" command (Cmd+O) and syncs with the IDE
  extension. The docs do not explicitly mention an "open in editor" button, so
  this may be a newer or undocumented UI feature.

Opportunity and fit:
- A fast "Open in Editor" action helps users jump from the control plane to
  their preferred editor (VS Code, Cursor, Windsurf, JetBrains, etc.).
- This aligns with the UI optionality: it can live in CLI first, then in a GUI
  or extension later.

Integration sketch:
- Add `lazyagent project open --editor <name>` and `lazyagent variant open`
  commands.
- Detect installed editors per OS and provide a short menu with fallbacks.
- Allow per-project defaults and custom launch commands in
  `.lazyagent/workspace.json`.
- Use platform-specific launchers (macOS `open -a`/`open -b`, Windows
  `start`/`Start-Process`, Linux `xdg-open` or direct exec paths).
- Always open the exact worktree path for a variant, not just the repo root.

Design details (editor detection and launch):
- Maintain a small editor registry with: `id`, display name, detection signals,
  launch commands per OS, and supported open capabilities (folder only vs
  file:line:column).
- Prefer explicit launch commands for known editors; fallback to system default
  opener when detection fails.
- Allow per-project overrides and custom commands for niche editors.

macOS detection/launch:
- Detection signals: `.app` bundles in `/Applications` or `~/Applications`,
  bundle IDs via Spotlight metadata, or CLI binaries in PATH.
- Launch: `open -b <bundle-id> <path>` (stable) or `open -a "App Name" <path>`.
- Fallback: use editor CLI binaries if present (e.g., `code`, `cursor`, `zed`).

Windows detection/launch:
- Detection signals: editor CLI binaries in PATH; registry entries under
  `HKCU/HKLM\Software\Microsoft\Windows\CurrentVersion\App Paths\*.exe`;
  JetBrains Toolbox install paths.
- Launch: `powershell Start-Process -FilePath <exe> -ArgumentList <path>` or
  `cmd /c start "" "<exe>" "<path>"`.
- Fallback: open in default editor via `cmd /c start "" "<path>"`.

Linux detection/launch:
- Detection signals: `which` for known editor binaries; desktop entries in
  `~/.local/share/applications` or `/usr/share/applications`.
- Launch: `gtk-launch <desktop-id>` for desktop entries, or exec the binary
  directly with the path.
- Fallback: `xdg-open <path>`.

Risks and open questions:
- Reliable editor detection varies across macOS, Windows, and Linux.
- Need a clear policy for host vs container paths when a variant runs in Docker.
- Should this be a core CLI command, a UI button, or both?

## 3) Codex-inspired Git surface (diff/commit/PR)

Sources:
- https://developers.openai.com/codex/app/features
- https://developers.openai.com/codex/app/commands

Notes:
- The Codex app provides a built-in Git diff pane with inline comments.
- It supports staging or reverting chunks/files, committing, pushing, and PR
  creation directly in the app.
- The UI includes a diff panel toggle and an integrated terminal for advanced
  Git tasks.
- Codex overlaps with LazyAgent on project/worktree management, but does not
  emphasize the same isolation guarantees (e.g., containerized envs).

Opportunity and fit:
- A Git surface in LazyAgent reduces context switching: users can inspect diffs
  and commit from the control plane while preserving isolation.
- The Git UI can be a thin layer on top of existing patch bundles and merge
  plans, not a separate workflow.

Integration sketch:
- CLI-first: `lazyagent diff` and `lazyagent status` summarize variant changes
  and patch bundles.
- UI (if built): a right-side diff panel with stage/revert actions and a commit
  form scoped to the active variant/worktree.
- Add a "Create PR" action that exports patch bundles + merge plan metadata.
- Keep an "Open in editor" escape hatch for complex edits or conflicts.

Design details (Git surface):
- Diff panel shows a file list with status (modified/untracked/staged) and a
  split view for before/after with inline comment threads.
- Inline comments are stored as variant-local review notes and can be converted
  into tasks or patch bundle annotations.
- Commit button is variant-scoped and checks merge gates before enabling.
- PR flow assembles a summary from patch bundle metadata, run logs, and the
  merge plan, then hands off to CLI/gh when needed.

Risks and open questions:
- Avoid duplicating Git UIs that users already trust in their editor.
- Decide how much Git functionality should live in CLI vs UI.
- Ensure commits in a variant never bypass merge-plan verification gates.
