# AGENTS.md — OneCLI template

## Purpose

One-click Zeabur deploy template for [OneCLI](https://github.com/onecli/onecli) — open-source credential gateway and vault for AI agents. Agents get placeholder keys; the gateway swaps in real credentials on outbound HTTP calls. Deliverable is `onecli.yaml`; `README.md` is deploy notes + architecture diagram.

## Ownership

Owned by SEAFOODHOLDHAND. Repo-wide rules live in the root `AGENTS.md`.

## Local Contracts

### Services (2)

| Service | Image | Role |
|---------|-------|------|
| PostgreSQL | `postgres:18-alpine` | DB (agents, encrypted secrets, audit logs) |
| onecli | `ghcr.io/onecli/onecli:latest` | App; `domainKey: PUBLIC_DOMAIN` |

### Single image, two processes, two ports

`ghcr.io/onecli/onecli:latest` bundles:
- **Next.js dashboard** on port `10254` (UI, agent/secret management) — port-id `web`
- **Rust gateway** on port `10255` (transparent credential-injection proxy, MITM HTTPS) — port-id `gateway`
- **Prisma migrations** run automatically via entrypoint on container start
- **`SECRET_ENCRYPTION_KEY`** auto-generated on first start, persisted to `/app/data/` (AES-256-GCM)

Both ports are HTTP and exposed by the onecli service. Volume `onecli-data` mounts `/app/data`.

### Critical: the encryption-key volume

`SECRET_ENCRYPTION_KEY` lives only in `/app/data/`. **Losing this volume makes every stored secret unrecoverable.** Do not remove or change the `onecli-data` volume.

### Auth modes (driven by NEXTAUTH_SECRET)

- Empty `NEXTAUTH_SECRET` → single-user mode, no login (dashboard opens directly)
- `NEXTAUTH_SECRET` set → Google OAuth multi-user mode (also needs `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET`)

`AUTH_TRUST_HOST=true` is required because the app sits behind Zeabur's proxy.

### URL env wiring

All public URLs derive from `${ZEABUR_WEB_DOMAIN}`:
- `APP_URL` / `NEXTAUTH_URL` → `https://${ZEABUR_WEB_DOMAIN}`
- `GATEWAY_API_URL` → `https://${ZEABUR_WEB_DOMAIN}:10255` (note the explicit gateway port)

Agents call the gateway at `https://<domain>:10255`, e.g. `/v1/chat/completions`, with `Authorization: Bearer <agent-access-token>`.

### DB URL

`DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}` — all four consumed from the PostgreSQL service's exposed vars.

### Localizations present

en (in `spec`), zh-TW, zh-CN, ja-JP, id-ID, es-ES.

## Work Guidance

- The gateway port (10255) is a real, separately-exposed HTTP port — keep it in `GATEWAY_API_URL` and in agent-facing docs.
- Prisma migrations are automatic; never add a manual migration step.

## Verification

None — static YAML, no build/test step.

## Child DOX Index

None — leaf folder.
