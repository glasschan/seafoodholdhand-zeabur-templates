# AGENTS.md — LearnHouse template

## Purpose

One-click Zeabur deploy template for [LearnHouse](https://github.com/learnhouse/learnhouse) — open-source LMS. Deliverable is `learnhouse.yaml`; `README.md` is deploy notes + architecture diagram.

## Ownership

Owned by SEAFOODHOLDHAND. Repo-wide rules live in the root `AGENTS.md`.

## Local Contracts

### Services (3)

| Service | Image | Role |
|---------|-------|------|
| PostgreSQL | `pgvector/pgvector:pg16` | DB (pgvector required for AI features — **not** standard postgres) |
| Valkey | `valkey/valkey:9-alpine` | Cache/sessions (Redis-compatible, BSD-3 fork) |
| Learnhouse | `ghcr.io/learnhouse/app:latest` | All-in-one app; `domainKey: PUBLIC_DOMAIN` |

### All-in-one app image

`ghcr.io/learnhouse/app:latest` bundles Next.js frontend, FastAPI backend, Collab WebSocket server, and an **internal nginx** (listens on port 80). Only the app's port 80 is exposed. No external reverse proxy — Zeabur's LB already terminates TLS.

Internal routing (inside the container):
```
/*           → Next.js Frontend (:8000)
/api/v1/*    → FastAPI Backend (:9000)
/api/auth/*  → NextAuth (via Frontend :8000)
/collab      → Collab WebSocket Server (:4000)
/content/*   → Backend Static Content (:9000)
```

### Valkey, not Redis — DNS hostname matters

Redis is intentionally replaced with **Valkey**. The service is named `Valkey` and its internal DNS hostname is `valkey`. Connection strings must use `redis://valkey:6379` — **not** `redis://redis:6379`. Both `LEARNHOUSE_REDIS_CONNECTION_STRING` and `LEARNHOUSE_REDIS_URL` use `redis://valkey:6379`.

### Runtime NEXT_PUBLIC_* injection

The image runs `server-wrapper.js` which reads `NEXT_PUBLIC_*` env vars at container start and injects them into the frontend. These can be changed in the Zeabur dashboard **without** rebuilding the image. All frontend-facing URLs are derived from `${ZEABUR_WEB_DOMAIN}`:
- `NEXT_PUBLIC_LEARNHOUSE_API_URL` → `https://${ZEABUR_WEB_DOMAIN}/api/v1/`
- `NEXT_PUBLIC_COLLAB_URL` → `wss://${ZEABUR_WEB_DOMAIN}/collab`

### SSR backend bypass

`LEARNHOUSE_API_URL: http://localhost:9000` — server-side rendering hits the backend directly inside the container, bypassing the internal nginx. Keep this as localhost, not the public domain.

### Admin seeding

First-run admin is seeded from `LEARNHOUSE_INITIAL_ADMIN_EMAIL` (`${ADMIN_EMAIL}`) and `LEARNHOUSE_INITIAL_ADMIN_PASSWORD` (`${ADMIN_PASSWORD}`) template vars. Default org slug is `default`.

### Startup behavior

A brief "Bad Gateway" in logs at startup is normal — the Next.js frontend boots faster than FastAPI, so the first SSR request may precede the backend. Self-resolves in seconds.

### Known doc drift

The English `spec.readme` mentions "OpenAI integration" for AI features, but the env vars (`LEARNHOUSE_GEMINI_API_KEY`) and the localized readmes (zh-CN/zh-TW/ja-JP/id-ID/es-ES) correctly reference **Gemini**. Treat Gemini as the real integration; the English readme's "OpenAI" wording is stale.

### Localizations present

en (in `spec`), zh-CN, zh-TW, ja-JP, id-ID, es-ES.

## Work Guidance

- When changing cache/session config, remember the DNS hostname is `valkey`, not `redis`.
- AI feature flag is `LEARNHOUSE_IS_AI_ENABLED=True` + `LEARNHOUSE_GEMINI_API_KEY`.

## Verification

None — static YAML, no build/test step.

## Child DOX Index

None — leaf folder.
