# LearnHouse for Zeabur

One-click deploy [LearnHouse](https://github.com/learnhouse/learnhouse) on Zeabur — the next-gen open source learning platform.

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://seafoodholdhand.com/recommends/zeabur-learnhouse/)

## Architecture

```
[Zeabur LB] → LearnHouse App (:80, internal nginx)
                   ├─ /*           → Next.js Frontend (:8000)
                   ├─ /api/v1/*    → FastAPI Backend (:9000)
                   ├─ /api/auth/*  → NextAuth (via Frontend :8000)
                   ├─ /collab      → Collab WebSocket Server (:4000)
                   └─ /content/*   → Backend Static Content (:9000)
```

### Services

| Service | Image | Purpose |
|---------|-------|---------|
| PostgreSQL 16 | `pgvector/pgvector:pg16` | Database for courses, users, content (with vector extension for AI) |
| Redis 7.2 | `redis:7.2.3-alpine` | Caching and session management |
| LearnHouse App | `ghcr.io/learnhouse/app:latest` | All-in-one: frontend + backend + collab + nginx |

### Why No External Nginx?

The LearnHouse app image bundles an internal nginx (listening on port 80) that routes traffic between the Next.js frontend, FastAPI backend, and collaboration WebSocket server. Since Zeabur's load balancer already handles TLS termination and HTTP routing, there's no need for an additional reverse proxy layer. This keeps the template to just 3 services — simpler and more cost-effective.

## Template Variables

| Variable | Type | Description |
|----------|------|-------------|
| PUBLIC_DOMAIN | Domain | Your LearnHouse instance domain (auto-assigned by Zeabur) |
| ADMIN_EMAIL | String | Email for the initial admin account |
| ADMIN_PASSWORD | Password | Password for the initial admin account |

## First Login

1. Wait 2-3 minutes for all services to start (check LearnHouse service health in Zeabur Dashboard)
2. Open your domain in a browser
3. Click **Sign In**
4. Log in with the admin email and password configured during deployment
5. Start creating courses!

> **Note:** You may see a brief "Bad Gateway" error in the logs during startup — this is normal. The Next.js frontend starts faster than the FastAPI backend, so the first SSR request may hit the backend before it's ready. It resolves within a few seconds.

## Post-Deployment Configuration

Customize these environment variables in the Zeabur Dashboard under the **learnhouse** service:

### AI Features
- `LEARNHOUSE_IS_AI_ENABLED` → `True`
- `LEARNHOUSE_GEMINI_API_KEY` → your Gemini API key

### Google OAuth
- `LEARNHOUSE_GOOGLE_CLIENT_ID` → your Google OAuth client ID
- `LEARNHOUSE_GOOGLE_CLIENT_SECRET` → your Google OAuth client secret

### S3 Content Storage
- `LEARNHOUSE_CONTENT_DELIVERY_TYPE` → `s3api`
- `LEARNHOUSE_S3_API_BUCKET_NAME` → your S3 bucket name
- `LEARNHOUSE_S3_API_ENDPOINT_URL` → your S3-compatible endpoint

### Email Sending
- `LEARNHOUSE_EMAIL_PROVIDER` → `resend`
- `LEARNHOUSE_RESEND_API_KEY` → your Resend API key
- `LEARNHOUSE_SYSTEM_EMAIL_ADDRESS` → sender email address

### Unsplash Image Library
- `NEXT_PUBLIC_UNSPLASH_ACCESS_KEY` → your Unsplash API access key (for course cover images)

### Cookie Domain
- `LEARNHOUSE_COOKIE_DOMAIN` → your cookie domain (e.g., `.zeabur.app`) for cross-subdomain auth

### Multi-Organization Mode
- `NEXT_PUBLIC_LEARNHOUSE_MULTI_ORG` → `True`
- `NEXT_PUBLIC_LEARNHOUSE_TOP_DOMAIN` → your top-level domain (e.g., `example.com`)

## Notes

- The LearnHouse image uses a **runtime config injection** mechanism (`server-wrapper.js`) that reads `NEXT_PUBLIC_*` environment variables at startup and injects them into the Next.js frontend. This means you can change these values without rebuilding the image.
- Content uploads are stored in the container filesystem by default (`LEARNHOUSE_CONTENT_DELIVERY_TYPE=filesystem`). For production use with persistent storage, consider switching to S3-compatible storage.
- The collaboration server uses WebSocket via the `/collab` path. Zeabur's load balancer supports WebSocket upgrades, so real-time collaboration (boards, live editing) works out of the box.
- PostgreSQL uses `pgvector/pgvector:pg16` (not standard PostgreSQL) because LearnHouse requires the `vector` extension for AI features.

---

Built by [SEAFOODHOLDHAND](https://seafoodholdhand.com) — [GitHub](https://github.com/glasschan/seafoodholdhand-zeabur-templates)
