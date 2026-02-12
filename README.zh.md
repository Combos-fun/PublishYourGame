# PublishYourGame 技能包

English version: `README.md`

## 1. 这是什么

`PublishYourGame` 是一个面向 GameAI Publisher 的技能包，包含：
- 跨平台发布脚本（Python / Node.js）
- API 契约文档
- curl / PowerShell 调用示例

该仓库内容已对齐同层目录：
`../game_backend/src/service/gameaipublisher`

## 2. 目录结构

- `SKILL.md`: 技能主说明（触发条件、流程、参数）
- `scripts/publish_game.py`: Python CLI
- `scripts/publish_game_node.mjs`: Node.js CLI
- `references/api-contract.md`: 最新接口契约
- `references/client-recipes.md`: 多客户端命令示例

## 3. 快速开始

上传 ZIP：

```bash
python3 scripts/publish_game.py upload-zip \
  --base-url http://localhost:3000 \
  --zip ./my_game.zip \
  --title "My Game" \
  --platform desktop \
  --content-type game \
  --cover ./cover.png
```

目录发布：

```bash
python3 scripts/publish_game.py publish-files \
  --base-url http://localhost:3000 \
  --dir ./my_game_project \
  --title "My Game" \
  --platform mobile \
  --content-type code
```

Node.js 版本：

```bash
node scripts/publish_game_node.mjs upload-zip \
  --base-url http://localhost:3000 \
  --zip ./my_game.zip \
  --title "My Game"
```

## 4. 关键参数

- `--base-url`: 服务地址
- `--title`: 游戏标题
- `--description`: 描述（可选）
- `--platform`: `desktop` 或 `mobile`
- `--content-type`: `game` 或 `code`
- `--cover`: 封面图（仅 upload-zip，JPG/PNG/WEBP）
- `--header "Authorization: Bearer <token>"`: 可选鉴权头

## 5. 成功返回重点字段

- `data.id`
- `data.pageUrl`
- `data.shareUrl`
- `data.cdnGameUrl`
- `data.ownershipType`
- 响应头 `x-request-id`
