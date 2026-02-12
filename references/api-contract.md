# GameAI Publisher API Contract (Latest)

This reference is aligned to:
`../game_backend/src/service/gameaipublisher` (API routes + Prisma model)

## 1) POST /api/upload

Purpose: publish from a zip package.

Auth:
- Optional `Authorization: Bearer <token>`.
- If token is valid, publish result is immediately owned by that user.
- If no token, result is community-owned and includes a one-time `onceToken` in `shareUrl`.

Request:
- Content-Type: `multipart/form-data`
- Fields:
  - `file` (required): `.zip` file
  - `title` (required)
  - `description` (optional)
  - `platform` (optional): `desktop` or `mobile`, default `desktop`
  - `contentType` (optional): `game` or `code`, default `game`
  - `cover` (optional): JPG/PNG/WEBP image, max 8MB

Server validation:
- `file` and `title` are required.
- `platform/contentType` must be valid if provided.
- `file.name` must end with `.zip`.
- `cover` (if provided) must pass type/size validation.
- zip must be parseable and contain `index.html`.

Zip parsing behavior:
- Skips directories and metadata (`__MACOSX`, `.DS_Store`).
- Normalizes path separators to `/`.
- Strips one shared top-level folder when all files are under it.
- Supports compression methods:
  - `0` (store)
  - `8` (deflate)
- Unsupported entries are skipped.

Success response:
- HTTP `201`
- Body:

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "My Game",
    "description": "optional",
    "platform": "desktop",
    "contentType": "game",
    "thumbnailUrl": "https://.../games/<id>/cover.jpg",
    "gameUrl": "https://host/games/<id>",
    "pageUrl": "https://host/games/<id>",
    "shareUrl": "https://host/games/<id>?onceToken=...",
    "cdnGameUrl": "https://cdn.example.com/games/<id>/index.html",
    "onceToken": "uuid-or-null",
    "ownershipType": "community",
    "ownerName": "optional",
    "ownerEmail": "optional",
    "ownerPicture": "optional",
    "ossPrefix": "games/<id>/",
    "fileCount": 12,
    "totalSize": 123456,
    "status": "published",
    "viewCount": 0,
    "likeCount": 0,
    "createdAt": "...",
    "updatedAt": "..."
  }
}
```

Notes:
- `gameUrl` points to the game detail page (same as `pageUrl`).
- `cdnGameUrl` points to OSS/CDN `index.html`.
- Anonymous publish returns usable claim link in `shareUrl` (`onceToken` included).

Common failures:
- `400` `file and title are required`
- `400` `platform must be mobile/desktop and contentType must be game/code`
- `400` `Only zip files are supported`
- `400` `Cover file exceeds 8MB limit` / `Cover must be JPG, PNG, or WEBP`
- `400` `Invalid zip file`
- `400` `Zip file is empty`
- `400` `index.html not found in zip`
- `401` Auth token errors (if bad/expired token is sent)
- `500` `Failed to upload game`

## 2) POST /api/publish

Purpose: publish from JSON payload (`files[]`).

Auth:
- Optional `Authorization: Bearer <token>`, same ownership behavior as `/api/upload`.

Request:
- Content-Type: `application/json`
- Body:

```json
{
  "title": "My Game",
  "description": "optional",
  "platform": "desktop",
  "contentType": "game",
  "files": [
    {
      "path": "index.html",
      "content": "<html>...</html>"
    },
    {
      "path": "assets/sprite.png",
      "contentBase64": "iVBORw0KGgo..."
    }
  ]
}
```

Server validation:
- `title` required.
- `files` required and non-empty.
- `platform/contentType` must be valid if provided.
- At least one file path is `index.html` or `./index.html`.

Upload behavior:
- Leading `./` is removed from each path.
- `contentBase64` has higher priority than `content`.
- Missing both `contentBase64` and `content` => file is skipped.

Success response:
- HTTP `201`
- Body shape is the same as `/api/upload` success body.

Common failures:
- `400` `title and files are required`
- `400` `platform must be mobile/desktop and contentType must be game/code`
- `400` `index.html is required as entry point`
- `401` Auth token errors (if bad/expired token is sent)
- `500` `Failed to publish game`

## 3) GET /api/games

Purpose: paginated list query.

Query params:
- `page` (default `1`)
- `pageSize` (default `12`, max `50`)
- `sortBy`: `createdAt|latest|likeCount|likes|viewCount|views|commentCount|comments`
- `order`: `asc|desc` (default `desc`)
- `platform`: `desktop|mobile|all`
- `contentType` or `type`: `game|code|all`

Response:
- `200` with `data.items[]`, `total`, `page`, `pageSize`, `totalPages`, `sortBy`, `order`, `platform`, `contentType`.
- Each item includes: `ownershipType` and `commentCount`.

Failures:
- `400` invalid `sortBy`
- `400` invalid platform/type filter

## 4) GET /api/games/random

Purpose: random game feed.

Query params:
- `count` (default `12`, max `50`)
- `platform`
- `contentType` or `type`

Response:
- `200` with `data.items[]`, `count`, `requestedCount`, `total`, `platform`, `contentType`.

## 5) GET /api/games/:id

Purpose: game detail (also increments view count).

Query params:
- `fingerprint` (optional): used to compute `isLiked`.
- `onceToken` (optional): returns `claimTokenState`.

Response fields include:
- `ownershipType`
- `isLiked`
- `claimTokenState`: `none|claimable|invalid|consumed`
- `commentCount`
- `comments[]`

Failures:
- `404` `Game not found`

## 6) POST /api/games/:id/like

Purpose: toggle like by fingerprint (no auth required).

Request:

```json
{ "fingerprint": "stable-device-id" }
```

Response:
- `200` `{ "success": true, "data": { "liked": true|false } }`

Failures:
- `400` `fingerprint is required`

## 7) POST /api/games/:id/comments

Purpose: create comment.

Auth:
- Required `Authorization: Bearer <token>`.

Request:

```json
{ "content": "Great game!" }
```

Validation:
- `content` required
- author name max `50` chars
- content max `1000` chars

Response:
- `200` comment object (`id`, `nickname`, `authorName`, `authorAvatar`, `content`, `createdAt`)

## 8) POST /api/games/:id/claim

Purpose: consume one-time ownership token.

Auth:
- Required only when `action=claim`.

Request:

```json
{
  "onceToken": "token-from-share-url",
  "action": "claim"
}
```

`action`:
- `claim` (default): attach to logged-in user
- `community`: keep community ownership

Failures:
- `400` invalid JSON/action/missing token
- `404` game not found
- `409` token consumed or invalid

## 9) POST /api/games/:id/cover

Purpose: upload/update cover image after publish.

Request:
- Content-Type: `multipart/form-data`
- Fields:
  - `cover` (required)
  - `source` (optional, default `manual`)

Response:
- `200` with `data.gameId` and `data.thumbnailUrl`.

## 10) GET/HEAD /api/games/:id/play/:assetPath

Purpose: proxy OSS assets via backend route.

Behavior:
- Default asset is `index.html`.
- Rejects path traversal (`.` / `..` segments).
- For HTML assets, cache header defaults to `no-cache`.
