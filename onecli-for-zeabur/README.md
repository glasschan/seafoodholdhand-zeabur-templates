# OneCLI for Zeabur

One-click deploy [OneCLI](https://github.com/onecli/onecli) on Zeabur — the open-source credential gateway and vault for AI agents.

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://seafoodholdhand.com/recommends/zeabur-onecli/)

## Architecture

```
[Zeabur LB] → OneCLI App (single image, two processes)
                  ├─ :10254  → Next.js Web Dashboard (UI + API)
                  └─ :10255  → Rust Gateway (credential injection proxy)
[PostgreSQL]  → OneCLI database (agents, secrets, audit logs)
```

### Services

| Service | Image | Purpose |
|---------|-------|---------|
| PostgreSQL 18 | `postgres:18-alpine` | Database for agents, encrypted secrets, audit logs |
| OneCLI App | `ghcr.io/onecli/onecli:latest` | All-in-one: Next.js dashboard + Rust gateway |

The OneCLI app image bundles:
- **Next.js web dashboard** on port 10254 (UI, agent management, secret management)
- **Rust gateway** on port 10255 (transparent credential injection proxy)
- **Prisma migrations** run automatically on container start
- **Auto-generated SECRET_ENCRYPTION_KEY** persisted to `/app/data/` (AES-256-GCM)

## Template Variables

| Variable | Type | Description |
|----------|------|-------------|
| PUBLIC_DOMAIN | Domain | Your OneCLI instance domain (auto-assigned by Zeabur) |
| NEXTAUTH_SECRET | String | NextAuth.js secret — empty = single-user mode, filled = Google OAuth mode |
| GOOGLE_CLIENT_ID | String | Optional — Google OAuth client ID for multi-user team mode |
| GOOGLE_CLIENT_SECRET | String | Optional — Google OAuth client secret for multi-user team mode |

## How It Works

1. **Open the dashboard** at `https://your-domain.zeabur.app`
2. **Create an agent** and get an access token
3. **Store real API keys** in OneCLI's encrypted vault (e.g. OpenAI, GitHub, Stripe)
4. **Configure your agent** to route HTTP traffic through `https://your-domain.zeabur.app:10255`
5. **Give your agent a placeholder** (e.g. `Bearer FAKE_KEY`) — when it makes a request, the gateway swaps it for the real credential

```
Your Agent → [FAKE_KEY] → OneCLI Gateway → [REAL_KEY] → External API (OpenAI, GitHub, etc.)
                                          ↓
                                   Audit log + rate limit
```

## First Login

1. Wait 2-3 minutes for all services to start (Prisma migrations run automatically)
2. Open your domain in a browser
3. You'll land directly on the dashboard (no login in single-user mode)
4. Create your first agent and grab its access token
5. Add secrets to the vault
6. Point your AI agent's HTTP client to the gateway URL

## Configure Your AI Agent

Once you have an agent token and at least one secret stored:

```bash
# Example: route OpenAI calls through OneCLI
curl https://your-domain.zeabur.app:10255/v1/chat/completions \
  -H "Authorization: Bearer <your-...n>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-4","messages":[{"role":"user","content":"Hello!"}]}'
```

In your agent code, just use a placeholder like `FAKE_OPENAI_KEY` and OneCLI swaps it for the real key.

## Post-Deployment Configuration

Customize these environment variables in the Zeabur Dashboard under the **onecli** service:

### Enable Google OAuth (multi-user mode)
- `NEXTAUTH_SECRET` → your NextAuth secret
- `GOOGLE_CLIENT_ID` → your Google OAuth client ID
- `GOOGLE_CLIENT_SECRET` → your Google OAuth client secret

### Custom Gateway Port
- `GATEWAY_PORT` → default 10255

## Notes

- The encryption key (`SECRET_ENCRYPTION_KEY`) is auto-generated on first start and persisted to `/app/data/`. **Don't lose this volume** — without it, all stored secrets are unrecoverable.
- The Rust gateway uses HTTPS interception (MITM) for credential injection. Your agent must trust the OneCLI CA cert (or set `--insecure-skip-verify` in development).
- Database migrations run automatically via the entrypoint script.
- The all-in-one image is ~500MB (Next.js + Rust gateway). Smaller than running separately.

---

Built by [SEAFOODHOLDHAND](https://seafoodholdhand.com) — [GitHub](https://github.com/glasschan/seafoodholdhand-zeabur-templates)
