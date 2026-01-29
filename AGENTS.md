# AGENTS.md
# Guidance for agentic coding assistants in this repository.

## Project snapshot
- Language: Python 3.12+ (see `pyproject.toml`)
- Entry point: `main.py`
- Primary goal: produce a spec sheet/description for a startup idea.
- Python is mainly for running small scripts; keep tooling light.
- Current codebase is intentionally small; no frameworks detected.

## Build, run, lint, test
### Run
- Run the app: `uv run python main.py`
- Run as module (if you add a package): `uv run python -m <package>`

### Build
- No build system configured.
- If you introduce packaging, use `python -m build` (add `build` to dependencies first).

### Lint/format
- Lint: `uv run ruff check .`
- Format: `uv run ruff format .`
- Auto-fix lint issues: `uv run ruff check . --fix`

### Tests
- Test runner: `pytest` (add tests under `tests/`)
- Run all tests: `uv run pytest`
- Run a single test file: `uv run pytest tests/test_example.py`
- Run a single test by node id: `uv run pytest tests/test_example.py::test_name`
- Add new tests under `tests/` and keep names `test_*.py`.

## Code style guidelines
### General
- Keep the codebase simple and readable; avoid premature abstractions.
- Prefer small, composable functions over deep nesting.
- Use explicit control flow; avoid clever one-liners when clarity suffers.
- Keep side effects localized and easy to identify.

### Imports
- Use standard library imports first, then third-party, then local imports.
- One import per line; avoid wildcard imports.
- Prefer absolute imports; avoid relative imports unless the package layout demands it.
- Keep import order stable; if you add tooling, enforce via formatter/linter.

### Formatting
- Follow PEP 8 spacing and line length guidelines.
- Use 4 spaces per indent; no tabs.
- Prefer double quotes for strings unless single quotes avoid escapes.
- Keep line length ~88-100 characters; break long expressions across lines.

### Types
- Type hints are not currently used; introduce them when adding non-trivial logic.
- If you add type hints, be consistent and annotate function signatures.
- Use `typing` constructs for collections (e.g., `list[str]`, `dict[str, int]`).
- Type checking: `uv run ty check .`

### Naming conventions
- Functions and variables: `snake_case`.
- Constants: `UPPER_SNAKE_CASE`.
- Classes: `PascalCase`.
- Modules: short, lowercase filenames.
- Avoid single-letter names except for trivial loop indices.

### Error handling
- Use exceptions for exceptional cases; do not swallow errors silently.
- Prefer explicit error messages when raising custom errors.
- Validate inputs at function boundaries, especially for user input.
- Keep error handling close to the source; do not over-broaden `try/except`.

### Logging/printing
- The sample app prints to stdout; keep output minimal and informative.
- If you add structured logging, choose a single approach and document it.

### CLI behavior
- If you expand CLI features, use `argparse` unless another framework is added.
- Provide `--help` output and clear error messages for invalid arguments.

## File organization
- `main.py` is the current entry point.
- If the project grows, add a `src/` or package directory and update `pyproject.toml`.
- Keep tests in `tests/` and mirror package structure when possible.

## Dependency management
- Use `pyproject.toml` for dependencies.
- Manage environments with `uv`; keep dependencies minimal.
- Add a runtime dependency: `uv add <package>`
- Add a dev dependency: `uv add --dev <package>`
- Sync the environment: `uv sync`
- Avoid adding heavyweight frameworks unless the user requests it.

## Documentation
- Update `README.md` with any new run/test instructions you add.
- Keep docstrings concise; document public functions and modules.
- Prefer usage examples over exhaustive API descriptions.
- If you add a CLI flag or environment variable, document it here and in `README.md`.
- Keep `plans/` docs in sync with decisions and changes; update summary sections when needed.

## Configuration and I/O
- Keep configuration explicit; prefer function parameters or argparse flags.
- Avoid reading environment variables deep in the call stack; keep them near the entry point.
- Use UTF-8 for text files unless a file already uses another encoding.
- When writing files, make output paths explicit and avoid hidden side effects.

## Data handling
- Validate external inputs early (CLI args, files, stdin).
- Preserve input ordering unless there is a clear need to sort.
- Avoid mutating inputs in place unless performance requires it; document when you do.

## Reliability and performance
- Favor clear, correct implementations over micro-optimizations.
- If adding caching, include invalidation rules in comments or docstrings.
- Keep functions deterministic where possible to simplify testing.

## Change safety checklist
- Ensure `main.py` remains a thin entry point that delegates to helpers.
- Add tests for new behavior when it is non-trivial.
- Update `pyproject.toml` if new dependencies or tooling are introduced.
- Avoid global state unless there is a strong reason; document any globals.

## Security and secrets
- Do not commit credentials, tokens, or API keys.
- Avoid logging sensitive data; redact if needed.
- Prefer parameterized inputs over string concatenation when building commands.

## Git hygiene
- Do not rewrite history unless explicitly requested.
- Keep commits scoped to the requested change.
- If the worktree is dirty, avoid reverting unrelated changes.

## OpenCode config
- Project config lives at `opencode.json` in the repo root.
- Python LSP uses Ruff via `uv run ruff server` and disables Pyright.
- Update the LSP command if the environment or tooling changes.

## Cursor/Copilot rules
- No Cursor rules found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions found in `.github/copilot-instructions.md`.

## Agent workflow expectations
- Before making changes, scan `main.py` and `pyproject.toml` for context.
- Keep edits focused on the user request; avoid unrelated refactors.
- When adding new files, ensure they are referenced in documentation.
- Do not introduce generated files or lockfiles unless requested.
- Primary role: feasibility research, technical planning, and creative exploration.

## Safe defaults for new code
- Write code that runs on Python 3.12+.
- Avoid non-ASCII characters unless a file already uses them.
- Keep modules importable without side effects at import time.
- Guard execution with `if __name__ == "__main__":` in scripts.

## Suggested testing strategy (if needed)
- Start with unit tests for new functions.
- Use fixtures sparingly; prefer explicit inputs.
- Keep tests deterministic and isolated from the environment.

## Repository limitations
- This repository currently has no CI, formatting, or linting automation.
- All guidance above is based on the current minimal codebase.

## Quick reference
- Run app: `uv run python main.py`
- Lint: `uv run ruff check .`
- Format: `uv run ruff format .`
- Type check: `uv run ty check .`
- Dependencies: `pyproject.toml`
- Entry point: `main.py`
