# Variant primitives beyond git worktrees (future direction)

This is a research deep dive into alternatives to git worktrees for managing
variants. It is not scheduled work. The goal is to design enough now that a
future implementation could move fast.

## Why explore alternatives
- Worktrees give isolation, but change sharing is manual (cherry-pick, patch,
  or rebasing).
- When two large variants depend on each other, you often want to share a
  subset of changes without merging the whole branch.
- Worktrees are tied to Git branches and the working copy, which can make
  cross-variant sync slow and conflict-prone.

## Evaluation criteria
- Fine-grained change mobility (move part of a feature to another variant).
- Deterministic re-application with conflict visibility.
- Compatibility with Git tooling and PR workflows.
- Low cognitive overhead for users.
- Works with isolation (containers) and parallel agents.

## Baseline: git worktrees (current)
Strengths:
- Simple mental model: one variant equals one working copy.
- Easy to run tools and tests in an isolated directory.
- Native Git support and compatibility.

Limitations:
- Change sharing is commit-centric and can be coarse (cherry-pick or rebase).
- Splitting a large change set is manual and error-prone.
- Syncing variants often means rebasing and resolving repeated conflicts.

## Option A: Patch stacks on top of Git

Software:
- StGit (Stacked Git) manages commits as a stack of patches.
- git-branchless adds patch-stack workflows and commit graph manipulation.

Sources:
- https://stacked-git.github.io
- https://raw.githubusercontent.com/stacked-git/stgit/master/README.md
- https://raw.githubusercontent.com/arxanas/git-branchless/master/README.md

What it is:
- Treats a series of commits as a patch stack that can be reordered, moved,
  and selectively applied.

How it helps the example:
- Move a single patch from variant 1 to variant 2, then restack.
- Keep the rest of the large feature unmerged.

Pros:
- Still Git underneath; easy to integrate with existing repos.
- Works with existing PR workflows.
- Gives a clearer model for partial change movement.

Cons:
- Still commit-based; granular change movement depends on good commit hygiene.
- Adds tooling complexity and extra commands.

## Option B: Change-centric VCS with Git storage (Jujutsu)

Software:
- Jujutsu (jj) uses Git as a backend while exposing a change-centric model.

Sources:
- https://www.jj-vcs.dev
- https://raw.githubusercontent.com/jj-vcs/jj/main/README.md

What it is:
- Changes are first-class, working copy is a commit, and history rewriting is
  built into the normal workflow.

How it helps the example:
- Split, move, or squash partial changes between stacks without checking out
  each branch.
- Automatic rebase propagates edits through dependent changes.

Pros:
- Git-compatible storage and remotes.
- Strong support for splitting and moving parts of changes.
- Undoable operation log reduces fear of rewriting history.

Cons:
- New toolchain for users and contributors.
- Still maturing; some Git features are incomplete.

## Option C: Patch-based VCS (Pijul)

Software:
- Pijul is patch-based and models change dependencies explicitly.

Sources:
- https://pijul.org/manual/why_pijul.html

What it is:
- Tracks changes (patches), not snapshots, and allows pulling only selected
  changes with dependency-aware application.

How it helps the example:
- Pull just the needed patches from variant 1 into variant 2.

Pros:
- Patch-level reasoning is native and explicit.
- Change commutation makes selective pulls more reliable.

Cons:
- Requires leaving Git for core workflows.
- Lower ecosystem compatibility and adoption risk.

## Option D: Filesystem snapshot or overlay (non-VCS primitive)

What it is:
- Use copy-on-write snapshots or overlayfs to create cheap working copies.

How it helps the example:
- Fast isolation, but change sharing still requires a diff/patch layer.

Pros:
- Very fast for creating many variants.
- Works well with large repos and local isolation.

Cons:
- Does not solve change mobility by itself.
- Needs a second layer for patch management and merge logic.

## Proposed LazyAgent primitive: Change graph + patch bundles

Design idea:
- Make a variant a selection of small, named change sets rather than a single
  branch. Each change set is a patch bundle with explicit dependencies.
- A worktree becomes a materialization step, not the source of truth.

Data model:
- ChangeSet: id, title, base_ref, patch_bundle, deps[], owner, tests, scope.
- Variant: base_ref + ordered change set selection + env profile.
- ChangeGraph: DAG of change sets with dependencies and status.

Key operations:
- `lazyagent change create`: capture a focused diff as a change set.
- `lazyagent change split`: split a large diff into multiple change sets.
- `lazyagent change apply`: apply a change set to another variant.
- `lazyagent change promote`: mark as shared dependency for multiple variants.
- `lazyagent variant materialize`: build a worktree/overlay from change sets.

Materialization flow:
- Start from base_ref.
- Apply change sets in dependency order using patch bundles.
- If conflicts occur, record a resolution patch and link it to the change set
  (similar to rerere).

How it solves the example:
- Variant 1 creates change sets A, B, C.
- Variant 2 needs B. Apply B (and its deps) to variant 2 without merging A or C.
- Both variants keep independent stacks, but shared dependencies are explicit.

Compatibility and export:
- Export a variant to a Git branch or PR by applying its change sets.
- Export individual change sets as patch bundles for review or CI.
- Continue to support worktrees as an optional materialization step.

Risks and costs:
- Requires excellent diff slicing and change set hygiene.
- Patch conflicts may repeat if dependency graphs are weak.
- Higher implementation complexity than simple worktrees.

## Suggested future exploration (not scheduled)
- Short spike on StGit or git-branchless to validate patch-stack workflows.
- Jujutsu evaluation for change movement and conflict handling.
- Prototype a minimal ChangeGraph layer using existing patch bundles.
