# AGENTS.md — n8n with Worker template

## Purpose

One-click Zeabur deploy template for **n8n v2** with dedicated worker nodes and external task runners for secure Code-node execution. Deliverable is `n8n-w-worker.yaml`; the readme + Update Log live **inside** `spec.readme` (no separate changelog file).

## Ownership

Owned by SEAFOODHOLDHAND. Repo-wide rules live in the root `AGENTS.md`.

## Local Contracts

### Services (6)

| Service | Image | Role |
|---------|-------|------|
| PostgreSQL | `postgres:17` | DB |
| Redis | `redis:7-alpine` | Queue (BullMQ) |
| N8N | `n8nio/n8n:stable` | Main instance — `domainKey: PUBLIC_DOMAIN` |
| Worker | `n8nio/n8n:stable` (`n8n worker`) | Background task processing |
| N8N-Runners | `n8nio/runners:stable` | Task runner for main instance |
| Worker-Runners | `n8nio/runners:stable` | Task runner for worker |

N8N exposes two ports: `web` (5678 HTTP) and `taskbroker` (5679 TCP). Worker also exposes `taskbroker` (5679).

### Task runner architecture (hard-won split)

Each n8n instance needs its **own** dedicated runner — do not collapse back into a single `Runners` service:
- `N8N-Runners` → `N8N_RUNNERS_TASK_BROKER_URI: http://N8N:5679`, depends on `N8N`
- `Worker-Runners` → `N8N_RUNNERS_TASK_BROKER_URI: http://Worker:5679`, depends on `Worker`

Both runners share `N8N_RUNNERS_AUTH_TOKEN` (sourced from `${TASK_RUNNERS_AUTH_TOKEN}`) and auto-shutdown after 15s idle.

### Shared-secret wiring

- `N8N_ENCRYPTION_KEY` is generated on the N8N service (`default: ${PASSWORD}`). It is re-exposed as `N8N_ENCRYPTION_KEY_WORKER: { default: ${N8N_ENCRYPTION_KEY}, expose: true }` so the Worker can consume `${N8N_ENCRYPTION_KEY_WORKER}`. Never let Worker generate its own key — mismatched keys break credential decryption.
- `N8N_RUNNERS_AUTH_TOKEN` (on N8N) is re-exposed as `TASK_RUNNERS_AUTH_TOKEN: { default: ${N8N_RUNNERS_AUTH_TOKEN}, expose: true }`; both runner services consume `${TASK_RUNNERS_AUTH_TOKEN}`.

### Forced env flags

- `N8N_RUNNERS_MODE=external` — external runner containers (production-recommended)
- `EXECUTIONS_MODE=queue` — Bull via Redis
- `OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true` — future-compat for scaling mode
- Redis reached via `${REDIS_HOST}` / `${REDIS_PORT}` / `${REDIS_PASSWORD}`

### Changelog

The "Update Log" is maintained inside `spec.readme` (English) and mirrored in each locale's `readme`. Add a dated entry there when deploy behavior changes — do not create a separate changelog file.

### Localizations present

en (in `spec`), zh-CN, zh-TW, ja-JP. **Not** localized to id-ID / es-ES (unlike Multica/LearnHouse/OneCLI).

## Work Guidance

- When touching runner/env config, update the readme Update Log in every locale block that exists.
- Keep `dependencies:` at the service level (parent rule) — this template's 2026-03-23 fix was specifically about that.

## Verification

None — static YAML, no build/test step.

## Child DOX Index

None — leaf folder.
