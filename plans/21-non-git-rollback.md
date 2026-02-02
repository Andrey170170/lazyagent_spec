# Non-git rollback primitives (future direction)

This deep dive explores rollback options for ad-hoc tasks where Git is
overkill (e.g., editing `~/.bashrc` or running a one-off upload script).
It proposes a lightweight snapshot layer that does not require a repo.

## Problem framing
- Git is excellent for codebases, but heavy for one-off tasks or system files.
- We still need safe rollback and auditability for agent-driven changes.
- Many tasks operate on paths outside repos (home directory, `/etc`, server
  directories), where "init a repo" is a non-starter.

## Requirements for a better solution
- Works on arbitrary paths without initializing a VCS repo.
- Fast snapshot + restore, with minimal user setup.
- Supports selective restore (one file or subset of a directory).
- Stores enough metadata for audit and diff summaries.
- Local-first and offline; optional encryption for sensitive paths.

## Existing approaches (survey)

### Backup snapshot tools
These are strong for directories and one-off tasks, but not integrated with
code-centric flows.

Sources:
- https://raw.githubusercontent.com/restic/restic/master/README.md
- https://raw.githubusercontent.com/kopia/kopia/master/README.md

Notes:
- Restic and Kopia create deduplicated snapshots and support encryption.
- They work well for "capture state before run, restore if needed."
- Not tailored for diffing or patch-level change sharing.

### Dotfile managers
Great for config files but still Git-centric or specialized.

Sources:
- https://raw.githubusercontent.com/twpayne/chezmoi/master/README.md
- https://raw.githubusercontent.com/TheLocehiliosan/yadm/master/README.md

Notes:
- chezmoi is purpose-built for dotfiles across machines.
- yadm is Git-based with dotfile-focused features.
- These help in the "bashrc" case, but are not general rollback primitives.

### Filesystem snapshots (ZFS/Btrfs/APFS)
Best for speed, but depend on filesystem support and admin permissions.

Notes:
- Near-instant snapshots and restores at the filesystem level.
- Great for large trees; not portable across all user machines.
- Hard to expose at the app layer without privileged access.

## Proposed LazyAgent primitive: Snapshot & rollback layer

Design goal:
Enable Git-like rollback for any path without requiring a repo.

### Concept
- Treat a "session" as a set of watched paths.
- Snapshots capture file content and metadata into a local store.
- Restores can be full or selective, with a diff summary.

### Data model (local store)
- SnapshotSet: id, label, created_at, root_paths[], notes.
- SnapshotEntry: path, type, mode, size, mtime, content_hash.
- Blob store: content-addressed file chunks for deduplication.

### Core commands (future)
- `lazyagent snapshot create --paths <...> [--label]`
- `lazyagent snapshot diff <snapshot-id> --paths <...>`
- `lazyagent snapshot restore <snapshot-id> --paths <...> [--dry-run]`
- `lazyagent snapshot list`

### Auto-safety flow
- Before running a risky command, take a snapshot of the target paths.
- After the run, show a diff summary and offer rollback.
- Keep snapshots in `~/.config/lazyagent/snapshots/` by default.

### Storage backends
- **Internal CAS**: content-addressed blobs + manifests.
- **External**: optional restic/kopia integration if installed.
- Backend selection via `--backend` with internal as default.

### Selective restore
- Restore a single file or subdirectory without rolling back everything.
- Provide a diff preview (text-only) before applying changes.

### Security and safety
- Require explicit path allowlist; never auto-snapshot `/` or `/home`.
- Allow sensitive-path redaction and optional encryption.
- Keep a small audit log: snapshot id, command, actor, timestamp.

## How this solves the "one-off" cases

Example: update `~/.bashrc`
1. `lazyagent snapshot create --paths ~/.bashrc --label "before bashrc"`
2. Apply changes.
3. `lazyagent snapshot diff <id>` to review.
4. `lazyagent snapshot restore <id> --paths ~/.bashrc` if needed.

Example: upload scripts in a server folder
1. Snapshot the target directory.
2. Run upload script.
3. Restore only the damaged files if necessary.

## Integration with variants
- Git-backed variants still use worktrees and patch bundles.
- Non-git tasks use SnapshotSets as the rollback primitive.
- Both surfaces should share UX (diff/restore, audit log).

## Risks and open questions
- Snapshot size and retention policy for large directories.
- How to handle binary diffs and large media files.
- Whether to store snapshots per-user or per-project.
- UX for partial restores without confusing users.

## Suggested future exploration (not scheduled)
- Prototype internal CAS snapshot store on a single-file path.
- Add restic/kopia backend adapters for large directory snapshots.
- Define a minimal audit log format that mirrors run logs.
