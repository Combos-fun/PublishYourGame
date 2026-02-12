# PublishYourGame Skill Pack

Chinese version: `README.zh.md`

## 1. What this is

`PublishYourGame` is a skill pack for GameAI Publisher, including:
- Cross-platform publish scripts (Python / Node.js)
- API contract documentation
- curl / PowerShell recipes

This repository is synced to:
`../game_backend/src/service/gameaipublisher`

## 2. Repository layout

- `SKILL.md`: skill entry and workflow
- `scripts/publish_game.py`: Python CLI
- `scripts/publish_game_node.mjs`: Node.js CLI
- `references/api-contract.md`: latest API contract
- `references/client-recipes.md`: runnable multi-client commands

## 3. Quick start

Upload ZIP:

```bash
python3 scripts/publish_game.py upload-zip \
  --base-url https://share.combos.fun \
  --zip ./my_game.zip \
  --title "My Game" \
  --platform desktop \
  --content-type game \
  --cover ./cover.png
```

Publish from directory:

```bash
python3 scripts/publish_game.py publish-files \
  --base-url https://share.combos.fun \
  --dir ./my_game_project \
  --title "My Game" \
  --platform mobile \
  --content-type code
```

Node.js version:

```bash
node scripts/publish_game_node.mjs upload-zip \
  --base-url https://share.combos.fun \
  --zip ./my_game.zip \
  --title "My Game"
```

## 4. Key options

- `--base-url`: publisher host
- `--title`: game title
- `--description`: optional description
- `--platform`: `desktop` or `mobile`
- `--content-type`: `game` or `code`
- `--cover`: cover image (upload-zip only, JPG/PNG/WEBP)
- `--header "Authorization: Bearer <token>"`: optional auth header

## 5. Important success fields

- `data.id`
- `data.pageUrl`
- `data.shareUrl`
- `data.cdnGameUrl`
- `data.ownershipType`
- response header `x-request-id`
