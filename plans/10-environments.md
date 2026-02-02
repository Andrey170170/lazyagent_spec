# Environments, templates, and drift handling

## Goals
- Make environment setup fast and repeatable.
- Avoid manual Dockerfile editing for common cases.
- Track environment drift and make it easy to persist.

## Concepts
- Template: reusable base image + feature set.
- Environment spec: project-level, reproducible definition of tooling.
- Overlay: variant-level, temporary changes not yet promoted.
- Journal: append-only record of runtime installs and changes.

## Template store
- Built-in templates (Python, Node, Rust, Go, ML/GPU).
- Local templates stored under `~/.config/lazyagent/templates/env/`.
- Remote templates from a registry (optional later).

Template metadata fields (example):
- `id`, `name`, `base_image`, `features[]`, `defaults`, `labels`.

## Template builder (CLI-first; UI optional)
- Detect project type from repo files or wizard selections.
- Offer a recommended template.
- Optional checkboxes for features: git, uv, node, docker-in-docker, GPU.
- Generate env spec + devcontainer/Dockerfile from selections.

## Environment spec
Canonical environment definition stored at `.lazyagent/env/spec.json`.

Proposed fields:
- `base_image`: image name + digest.
- `features`: list of features (devcontainer features or internal).
- `apt`: packages list.
- `pip`: packages list or `uv` constraints.
- `node`: npm/pnpm packages list (optional).
- `env_vars`: key/value pairs.
- `tools`: sidecar/tooling versions.

## Drift detection
Drift is any change inside the container that is not in `spec.json`.

Signals we use:
- Package manager shims: wrap apt, pip, npm, uv to record installs/removals.
- Filesystem diff: use `docker diff <container>` to list changed files.
- Targeted watchers: monitor `/etc`, `/usr/local`, `/opt`, tool config dirs.

Each detected change is recorded to `journal.jsonl` with:
- timestamp
- source (apt/pip/npm/uv/docker-diff/watch)
- operation (install/remove/modify)
- payload (package name, file path, hash)

Variants can keep changes as overlays or promote to spec.

## Promote flow
1. Show detected changes in CLI (with optional UI view).
2. User chooses to persist or discard.
3. Persisted changes update `spec.json`.
4. Regenerate devcontainer config/Dockerfile.
5. Rebuild image and update base digest.

Promotion rules:
- Package installs go into `spec.json` lists.
- Config file changes are copied into `.lazyagent/env/overrides/` and added
  as Dockerfile `COPY` steps in the generated image.
- Unknown changes remain overlays with a warning.

## Forking with envs
- New variants reuse the project env spec by default.
- Overlays apply on top for experimental installs.
- Forks stay fast because base images are cached locally.

## Merging env changes
Env spec is treated like code and merged with an explicit plan.

Stable point strategy:
- Start from the base spec at fork time.
- Apply promoted changes from each variant.
- Compute a stable spec candidate:
  - Always keep base spec.
  - For added packages, default to union with warnings.
  - For removed packages, require explicit choice.
  - For conflicting file overrides, require explicit choice.

Merge plan output includes:
- `spec.json` diff (added/removed packages, env vars).
- Overlay diff (file overrides and conflicts).
- Suggested resolution and impact notes.

## Notes
- For MVP, keep devcontainer.json as generated output, not the source of truth.
- Optionally support devcontainer.json as canonical later if users prefer it.
