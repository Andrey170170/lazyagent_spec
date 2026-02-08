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

## 4) P2P collaboration mesh (serverless team sync)

Sources:
- https://raw.githubusercontent.com/automerge/automerge/main/README.md
- https://raw.githubusercontent.com/automerge/automerge-repo/main/README.md
- https://raw.githubusercontent.com/syncthing/syncthing/main/README.md
- https://raw.githubusercontent.com/hypercore-protocol/hypercore/main/README.md
- https://raw.githubusercontent.com/ipfs/kubo/master/README.md

Notes:
- Automerge provides CRDTs and sync protocols for local-first data.
- Automerge Repo includes pluggable networking, including websocket adapters.
- Syncthing demonstrates peer-to-peer file sync with safety and security goals.
- Hypercore and IPFS highlight content-addressed, append-only or DAG-based
  replication primitives.

Opportunity and fit:
- A P2P mesh supports small teams that want collaboration without a hosted
  server or centralized trust.
- It aligns with local-first workflows: each member has a complete copy and can
  work offline, then sync changes when peers are available.

Integration sketch:
- Represent workspace metadata (variants, runs, task state) as CRDT documents.
- Store patch bundles and artifacts as content-addressed blobs, replicated on
  demand (sparse sync for large repos).
- Provide a local "relay" mode for teams with NAT issues, but keep it optional.
- Use per-workspace keys for encryption and signing of updates.
- Add CLI flows for peer invites and trust onboarding (`lazyagent peer invite`).

Risks and open questions:
- NAT traversal and peer discovery are brittle without a relay.
- Eventual consistency can confuse users without strong conflict UX.
- Key management and revocation are hard in small teams.
- Large binary artifacts could overwhelm peer devices.

## 5) Hosted collaboration service (central server)

Sources:
- https://raw.githubusercontent.com/automerge/automerge-repo/main/README.md
- https://raw.githubusercontent.com/supabase/supabase/master/README.md

Notes:
- Automerge Repo includes websocket network adapters for client/server sync.
- Supabase demonstrates a hosted stack with Postgres, realtime subscriptions,
  auth, and storage services.

Opportunity and fit:
- A hosted service offers reliable presence, fast sync, and simpler onboarding.
- Central server can provide team-wide search, audit trails, and access control.

Integration sketch:
- Server stores workspace metadata, patch bundles, and run artifacts.
- Use websocket or realtime channels for presence, lock hints, and fast updates.
- Auth and org management (teams, roles, project access) live in the server.
- Object storage for large artifacts; database for metadata and indexes.
- Offline clients still work locally and sync when connected.

Risks and open questions:
- Hosting increases cost and complexity (SaaS surface area).
- Requires robust security model, rate limiting, and tenancy isolation.
- Must avoid coupling to a single cloud provider too early.

## 6) Kanban agent board (parallel work view)

Sources:
- https://en.wikipedia.org/wiki/Kanban_%28development%29
- https://www.atlassian.com/agile/kanban

Notes:
- A Kanban board emphasizes flow, WIP limits, and clear status transitions.
- It is a complementary view to a coding-first interface, not a replacement.

Opportunity and fit:
- Parallel agents map naturally to a board: each card is a task with status.
- Helps users coordinate multiple variants and avoid overloading agents.
- Provides a high-level view for planning and prioritization.

Integration sketch:
- Board columns: Backlog, Ready, In Progress, Review, Done (configurable).
- Cards represent tasks from Beads or a lightweight internal task model.
- Each card can spawn a variant/run and links to its patch bundles.
- WIP limits per column; show blockers and dependency links on cards.
- "Open" actions jump into the coding view for the active task/variant.
- Swimlanes by agent or by variant to show parallel execution.

Risks and open questions:
- Avoid duplicating existing task tools (Linear, Jira, Beads).
- Need reliable status syncing between runs and cards.
- Decide on the source of truth for tasks (Beads vs internal vs external).

## 7) Observer companion app (read-only remote progress)

Sources:
- https://raw.githubusercontent.com/tsl0922/ttyd/main/README.md
- https://raw.githubusercontent.com/yudai/gotty/master/README.md
- https://raw.githubusercontent.com/xtermjs/xterm.js/master/README.md
- https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API
- https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events

Notes:
- ttyd and GoTTY show that terminal output can be streamed to the web with a
  read-only default and optional write mode.
- xterm.js supports robust browser terminal rendering and has attach/serialize
  addons that help with reconnect and playback UX.
- Notifications API supports system-level notifications outside the active app.
- WebSocket and SSE are practical transports for pushing near-real-time updates.

Opportunity and fit:
- Matches the "camera over the workplace" need: monitor agent progress from a
  phone without disturbing execution.
- Reduces anxiety on long runs by surfacing current step, step count, and
  high-level status in a glanceable view.
- Complements the coding UI and Kanban view with a passive observation surface.

Integration sketch:
- Add a daemon progress stream (`run_started`, `step_started`, `step_done`,
  `log_chunk`, `run_failed`, `run_done`) with monotonic sequence IDs.
- Expose a companion web app (PWA first) with a compact status card: current
  task title, step progress (`5/7`), elapsed time, and health state.
- Push notification updates on significant events (step complete, blocked,
  failed, completed) with deep-link into full run view.
- Full run view includes read-only terminal stream (scrollback), progress
  summary, latest diff summary, and recent artifacts.
- Keep observer channel strictly read-only by default; any interactive control
  requires explicit elevation with short-lived tokens.

Design details (non-interference):
- Separate control plane and observer plane endpoints.
- Observer plane only reads append-only run events and terminal output buffers.
- Rate-limit observer polling/stream subscriptions to avoid impacting agent run.
- If observer disconnects, run continues unchanged; no coupling to lifecycle.

Risks and open questions:
- Remote access security (auth, device trust, token rotation, revocation).
- Mobile background limits can delay notifications or stream updates.
- Step counts are uncertain for open-ended agent tasks; need robust fallback UX.
- Deciding between local-only LAN mode, tunneled mode, or hosted relay.

## 8) Live config updates (no restart when possible)

Sources:
- https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration
- https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/runtime
- https://www.envoyproxy.io/docs/envoy/latest/start/quick-start/configuration-dynamic-filesystem.html
- https://nginx.org/en/docs/control.html
- https://caddyserver.com/docs/api
- https://context7.com/samuelcolvin/watchfiles
- https://context7.com/open-feature/spec

Notes:
- Envoy demonstrates a strong no-restart model via xDS and runtime config.
- Envoy dynamic filesystem docs emphasize atomic file replacement, not in-place
  edits, to preserve consistency.
- NGINX uses signal-based graceful config reload (`HUP`) as a proven daemon
  pattern.
- Caddy exposes an admin API for runtime config updates.
- watchfiles supports fast file-watch loops and process reload workflows.
- OpenFeature models runtime configuration change events via provider handlers.

Opportunity and fit:
- Reduces friction when a user tweaks skills, policies, or adapter config during
  active work.
- Keeps long-running sessions useful by applying safe changes immediately.
- Provides a cleaner UX than "edit config then restart everything".

Integration sketch:
- Add a Config Update Manager in the daemon with file watchers + apply pipeline.
- Introduce config classes with explicit apply semantics:
  - `hot`: apply immediately to active components.
  - `next_run`: apply only to newly started runs/sessions.
  - `restart_required`: stage update and show restart action.
- Maintain versioned config snapshots and an apply journal for rollback.
- Validate updates before activation (schema, adapter capability checks,
  permission rules).
- Perform two-phase apply: stage -> activate -> health check -> commit.
- Push `config_changed` events to UI/CLI with per-component status.

Design details (safety + non-interference):
- Use atomic writes and rename/swap semantics for generated files.
- Coalesce rapid file changes (debounce window) to avoid reload storms.
- Keep a capability matrix per adapter/tool so unsupported live updates
  gracefully downgrade to `next_run` or `restart_required`.
- Never interrupt active agent execution for `hot` updates unless policy says so.
- If activation fails, auto-rollback to last known-good config and raise alert.

Possible commands:
- `lazyagent config apply --live`
- `lazyagent config status`
- `lazyagent config rollback <version>`

Risks and open questions:
- Users may assume all changes are live when some are only `next_run`.
- Race conditions if tools read config while files are being regenerated.
- Tool-specific limitations can make behavior inconsistent across adapters.
- Need clear UX so "applied" always means "applied to what scope".
