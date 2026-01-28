# Tech stack (MVP)

## Summary
MVP is Python-first to maximize team speed and iteration. Rewrites to Go can happen after product validation.

## Core runtime
- Language: Python 3.12+.
- Package manager: uv.
- Lint/format: ruff.
- Type check: ty.
- Test: pytest.

## Services and UI
- Daemon/API: Python async service (HTTP over Unix socket preferred).
- TUI: Textual (Python) with embedded PTY support for tool sessions.
- CLI: Typer (Python) for a thin command surface that calls the daemon.
- Sidecar bridge: Python process inside containers for PTY streaming + status.
 - Daemon runs continuously; TUI/CLI attach from any directory.

## Python libraries (planned)
- Async runtime: `asyncio` (stdlib).
- API server: `FastAPI` + `uvicorn` (ASGI) with WebSocket support.
- TUI: `textual` (built on `rich`).
- CLI: `typer` (+ `rich` for output).
- PTY streaming: `pty` (Unix stdlib) + asyncio streams; Windows later via `pywinpty`.
- Validation/config models: `pydantic` (or `dataclasses` + `jsonschema` fallback).
- File watching: `watchfiles` for config/skill updates.
- Logging: `structlog` or stdlib `logging` with JSON format.
- Async tests: `pytest` + `pytest-asyncio`.

## Tooling integrations
- Git operations: git CLI (worktrees, diff, status, patch bundles).
- Container control: Docker CLI (devcontainer and compose orchestration).
- Devcontainer lifecycle: devcontainer CLI (when available) or direct docker/compose.
- Editor integration: VS Code/Cursor extension in TypeScript.

## Integration approach
- Prefer CLI invocations for git and docker to match user environments.
- Use `subprocess` with structured capture and exit-code handling.

## Storage formats
- Metadata: JSON/JSONL in `.agentplane/` and `~/.config/agentplane/`.
- Logs/artifacts: text and structured JSON summaries per run.
- Context packs: JSON with provenance metadata.

## Transport and protocols
- Local RPC: HTTP + WebSocket for PTY streams.
- Authentication: local-only, socket file permissions.

## Rewrite path (post-MVP)
- Keep RPC contracts stable so services can be swapped.
- Target Go for daemon + sidecar if performance/packaging becomes a bottleneck.
- Leave TUI in Python or replace with Go TUI later.
