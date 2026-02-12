# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A skill pack for GameAI Publisher containing zero-dependency CLI tools (Python and Node.js) that publish web games via two API endpoints:
- `POST /api/upload` — publish from a `.zip` package (multipart/form-data)
- `POST /api/publish` — publish from a project directory as JSON file payload

The backend this targets lives at `../game_backend/src/service/gameaipublisher`. When backend behavior changes, re-sync scripts and docs against that path.

## Validation Commands

There is no test framework. Minimum checks before any PR:

```bash
python3 -m py_compile scripts/publish_game.py      # Python syntax check
node --check scripts/publish_game_node.mjs          # Node syntax check
python3 scripts/publish_game.py --help               # Verify CLI entry
node scripts/publish_game_node.mjs --help            # Verify CLI entry
```

## Repository Layout

- `SKILL.md` — Skill entry point (trigger, workflow, parameters). This is a Claude Code skill definition.
- `scripts/publish_game.py` — Python CLI (stdlib only, no third-party deps)
- `scripts/publish_game_node.mjs` — Node.js ESM CLI (built-in modules only)
- `references/api-contract.md` — Full API contract aligned to backend
- `references/client-recipes.md` — Runnable multi-client examples (curl, PowerShell, Python, Node)

## Architecture Notes

Both CLIs implement identical behavior with two subcommands:

- **`upload-zip`** — reads a `.zip` file, builds multipart/form-data, POSTs to `/api/upload`. Optionally attaches a cover image.
- **`publish-files`** — walks a project directory, encodes files as Base64 (or UTF-8 text with `--prefer-text`), POSTs JSON to `/api/publish`.

Both CLIs generate an `x-request-id` header per request for tracing. Output goes to stdout (JSON response body) and stderr (summary fields like `game_id`, `page_url`, `share_url`).

Authorization is optional: with a Bearer token the game is user-owned; without it, the game is community-owned and a one-time `onceToken` claim link is returned in `shareUrl`.

## Conventions

- **Code changes** go in `scripts/`, **documentation changes** go in `references/` or root docs.
- CLI flags use kebab-case (`--content-type`); JSON fields follow backend contract (`contentType`, `onceToken`).
- Python: 4-space indent, type hints for new logic.
- Node: ESM `.mjs`, `camelCase` for variables/functions.
- Commits: short, imperative, lowercase (e.g., `add ...`, `fix ...`, `simplify ...`). Keep commits scoped (scripts vs docs).
