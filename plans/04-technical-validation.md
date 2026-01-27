# Technical validation (doc-checked)

## Variant management via git worktrees
Source: https://git-scm.com/docs/git-worktree
- Git supports multiple linked worktrees with add/list/remove commands.
- Worktrees can be locked to prevent prune; list supports porcelain output.
- Worktree metadata lives under .git/worktrees; do not edit directly.

Implication:
- Variants can be first-class using worktree add/list/remove.
- Workspace registry should track worktree paths and branch refs.

## Patch bundles and application
Sources:
- https://git-scm.com/docs/git-format-patch
- https://git-scm.com/docs/git-apply
- https://git-scm.com/docs/git-am

Notes:
- git format-patch generates patch files per commit.
- git apply applies patches without creating commits.
- git am applies patches and creates commits from mail-style patches.

Implication:
- Patch bundles can be stored as format-patch output plus metadata.
- Merge plans can choose apply (no commit) or am (commit) flows.

## Devcontainer-based environments
Source: https://containers.dev/implementors/json_reference/
- devcontainer.json supports image or Dockerfile builds and compose usage.
- runArgs, mounts, and hostRequirements provide runtime controls.
- Lifecycle scripts support consistent setup and reproducibility.

Implication:
- Environment profiles can map to devcontainer configs per variant.
- Multi-service environments can rely on dockerComposeFile.

## Container resource limits
Source: https://docs.docker.com/engine/containers/resource_constraints/
- Docker supports CPU and memory limits via runtime flags (for cgroups).
- Resource limits are enforceable at container run time.

Implication:
- Isolation plane can enforce CPU/memory caps per variant.

## Tool adapter: OpenCode
Source: https://opencode.ai/docs/config/
- Project config lives at opencode.json and supports tool/agent settings.
- .opencode directories can store agents/commands/skills.

Implication:
- Capability manager can export to opencode.json and .opencode/.

## Gaps and follow-ups
- Verify Cursor and Claude Code config formats for adapters.
- Confirm compatibility across git versions (worktree config extensions).
- Confirm podman support if Docker is not available.
