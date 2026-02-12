# Repository Guidelines

## Project Structure & Module Organization
- `SKILL.md`: entry point for the skill behavior and usage workflow.
- `scripts/publish_game.py`: Python CLI for `/api/upload` and `/api/publish`.
- `scripts/publish_game_node.mjs`: Node.js CLI with equivalent behavior.
- `references/api-contract.md`: backend-aligned API contract.
- `references/client-recipes.md`: runnable examples (Python/Node/curl/PowerShell).
- `README.md`: default English quick start and usage summary.
- `README.zh.md`: Chinese quick start and usage summary.

Keep code changes in `scripts/` and documentation changes in `references/` or root docs.

## Build, Test, and Development Commands
- `python3 scripts/publish_game.py --help`: verify Python CLI entry and flags.
- `node scripts/publish_game_node.mjs --help`: verify Node CLI entry and flags.
- `python3 -m py_compile scripts/publish_game.py`: Python syntax check.
- `node --check scripts/publish_game_node.mjs`: Node syntax check.
- Example smoke run:
  - `python3 scripts/publish_game.py upload-zip --base-url http://localhost:3000 --zip ./my_game.zip --title "My Game"`

## Coding Style & Naming Conventions
- Python: 4-space indentation, type hints for new logic, small helper functions.
- Node: ESM (`.mjs`), `camelCase` for variables/functions, clear error messages.
- Markdown: concise sections, command examples that can run as-is.
- Naming:
  - CLI flags use kebab-case (`--content-type`).
  - JSON fields follow backend contract (`contentType`, `onceToken`).

## Testing Guidelines
- No formal test framework is configured in this repository.
- Minimum validation before PR:
  1. Run both syntax checks (`py_compile`, `node --check`).
  2. Run both `--help` commands.
  3. If CLI behavior changed, include one successful sample command and output snippet in PR description.

## Commit & Pull Request Guidelines
- Follow current history style: short, imperative, lowercase summaries (e.g., `add ...`, `move ...`, `simplify ...`).
- Keep commits scoped (scripts vs docs).
- PRs should include:
  - what changed,
  - why it changed,
  - affected files,
  - validation commands executed.

## Security & Sync Notes
- Never commit tokens, credentials, or private endpoints.
- Prefer env vars in examples (e.g., `$AUTH_TOKEN`).
- When backend behavior changes, re-sync docs/scripts against:
  - `../game_backend/src/service/gameaipublisher`
