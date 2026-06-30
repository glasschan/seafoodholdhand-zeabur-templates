# AGENTS.md — Multica template

## Purpose

One-click Zeabur deploy template for [Multica](https://github.com/multica-ai/multica) — managed agents platform (turn coding agents into teammates). Deliverable is `multica.yaml`; `README.md` is deploy notes + architecture diagram.

## Ownership

Owned by SEAFOODHOLDHAND. Sibling to the other `*-for-zeabur/` template folders; repo-wide rules live in the root `AGENTS.md`.

## Local Contracts

### Services (4) and wiring

```
[Zeabur LB] → Caddy Gateway (:3000, domainKey: PUBLIC_DOMAIN)
                   ├─ /health /readyz /ws /api/* /auth/* /uploads/*  → backend:8080
                   └─ /*                                                        → frontend:3000
PostgreSQL 17 (pgvector) ← backend
```

Dependency chain (note: `dependencies:` stays at service level, never inside `spec:`):
- `backend` depends on `PostgreSQL`, `gateway`
- `frontend` depends on `backend`, `gateway`
- `gateway` carries `domainKey: PUBLIC_DOMAIN` and exposes `APP_URL`

### Why Caddy exists

Prebuilt frontend images can't bake runtime `NEXT_PUBLIC_*`. Caddy puts everything behind one origin so there are no CORS / cross-origin WebSocket issues. Do not remove the gateway or try to expose backend/frontend directly.

### APP_URL propagation (the common break point)

Gateway sets and exposes:
```yaml
APP_URL: { default: ${ZEABUR_WEB_URL}, expose: true }
```
Backend consumes `${APP_URL}` for `FRONTEND_ORIGIN`, `CORS_ALLOWED_ORIGINS`, `MULTICA_APP_URL`, `MULTICA_PUBLIC_URL`. If the deployed URL fails to load after deploy, manually set `APP_URL` on the Gateway service and restart the Gateway.

### Login flow

Email login → 6-digit verification code. Code delivery:
- **Resend configured** (`RESEND_API_KEY` + `RESEND_FROM_EMAIL` template vars) → emailed
- **No Resend** → search Backend service logs for `[DEV] Verification code for`

### Post-deploy CLI

```bash
brew install multica-ai/tap/multica
multica setup self-host --server-url https://<domain> --app-url https://<domain>
```
Both URLs point to the same gateway domain.

### Localizations present

en (in `spec`), zh-CN, zh-TW, ja-JP, id-ID, es-ES. Update `readme` + `variables` in every locale block when behavior changes.

## Work Guidance

- When editing env vars, preserve the `APP_URL` expose/consume wiring — it is load-bearing for CORS and WebSocket.
- Caddyfile lives inline under the `gateway` service `configs:` (path `/etc/caddy/Caddyfile`). Route additions go there.

## Verification

None — static YAML, no build/test step.

## Child DOX Index

None — leaf folder.
