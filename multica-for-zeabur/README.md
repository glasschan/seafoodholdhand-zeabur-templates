# Multica for Zeabur

One-click deploy [Multica](https://github.com/multica-ai/multica) on Zeabur.

[**Deploy to Zeabur →**](https://seafoodholdhand.com/recommends/zeabur-multica/)

## Architecture

```
[Zeabur LB] → Caddy Gateway (:3000)
                   ├─ /health     → backend:8080
                   ├─ /readyz     → backend:8080
                   ├─ /ws         → backend:8080
                   ├─ /api/*      → backend:8080
                   ├─ /auth/*     → backend:8080
                   ├─ /uploads/*  → backend:8080
                   └─ /*          → frontend:3000
```

4 services: PostgreSQL 17 (pgvector) → Backend (Go) → Frontend (Next.js) → Caddy Gateway

## Deploy

Test:
```bash
npx zeabur@latest template deploy -f multica.yaml
```

Publish to Zeabur Marketplace:
```bash
npx zeabur@latest template create -f multica.yaml
```

## Why Caddy?

Prebuilt frontend images can't bake runtime env vars like `NEXT_PUBLIC_WS_URL`. Caddy solves this by putting everything behind one domain — WebSocket, API, and web all share the same origin. No CORS issues, no cross-origin WebSocket problems.

## Login

1. Enter your email on the login page
2. Get verification code:
   - **With Resend:** code sent to your email (set `RESEND_API_KEY`)
   - **Without Resend:** check Backend service logs for `[DEV] Verification code for`

## After Deployment

```bash
# Install CLI
brew install multica-ai/tap/multica

# Configure — both URLs point to the same domain
multica setup self-host \
  --server-url https://your-multica-domain.zeabur.app \
  --app-url https://your-multica-domain.zeabur.app
```

## Create by SEAFOODHOLDHAND

[seafoodholdhand.com](https://seafoodholdhand.com)
