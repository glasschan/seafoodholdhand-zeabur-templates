# AGENTS.md

This repo is a **collection of Zeabur one-click-deploy templates** — static YAML files. There is no code to build, test, lint, or run. No `package.json`, no CI. The "source" is the YAML itself.

## Repo layout

Each template lives in its own folder at the root: `<thing>-for-zeabur/` containing:
- `<thing>.yaml` — the Zeabur Template definition (the actual deliverable)
- `README.md` — human-facing deploy notes + architecture diagram

Templates today: `multica-for-zeabur`, `n8n-w-worker-for-zeabur`, `learnhouse-for-zeabur`, `onecli-for-zeabur`.

**When adding a template**, also add a row to the table in the root `README.md`.

## Zeabur Template schema

Schema: `https://schema.zeabur.app/template.json` (referenced via the `# yaml-language-server: $schema=...` comment at the top of YAML files — keep it).

Top-level shape:
```yaml
apiVersion: zeabur.com/v1
kind: Template
metadata: { name: ... }
spec: { description, icon, coverImage, variables, tags, readme, services }
localization:   # sibling of spec:, NOT nested inside it
  zh-CN: { description, variables, readme }
  zh-TW: { ... }
  ja-JP: { ... }
```

### Critical: `dependencies` placement

`dependencies:` is a field on the **service object** (sibling of `name`, `template`, `spec`), NOT inside `spec:`. Putting it inside `spec:` breaks Zeabur's startup ordering. This was a hard-won fix (see n8n changelog 2026-03-23); get this right on new services.

```yaml
- name: Worker
  template: PREBUILT_V2
  dependencies:        # ← here, service level
    - N8N
    - PostgreSQL
  spec: { ... }
```

## Zeabur built-in variables

These are resolved by the Zeabur platform, not defined anywhere in the YAML. Use them directly in `env.*.default` values:

- `${PASSWORD}` — auto-generated secret per service (used for DB passwords, JWT secrets, auth tokens)
- `${ZEABUR_WEB_URL}` / `${ZEABUR_WEB_DOMAIN}` — the service's public URL/host (only on the service bound to `domainKey`)
- `${CONTAINER_HOSTNAME}` — the service's internal DNS hostname (expose this as `*_HOST`)
- `${DATABASE_PORT}` — internal port for the service's `database` port-id
- `${PORT_FORWARDED_HOSTNAME}` / `${DATABASE_PORT_FORWARDED_PORT}` — public-facing connect info for instructions/connection strings

Services reach each other over internal DNS by service name, e.g. `http://backend:8080`, `http://N8N:5679`.

### Env var wiring pattern

A canonical pattern repeats across templates — `expose: true` vars on the producing service become available as `${VAR_NAME}` to consuming services:
- Producer sets `POSTGRES_HOST: { default: ${CONTAINER_HOSTNAME}, expose: true }`
- Consumer references it as `${POSTGRES_HOST}`

When a secret needs to be shared between services (e.g. `N8N_ENCRYPTION_KEY`), expose it from the producer (`N8N_ENCRYPTION_KEY_WORKER: { default: ${N8N_ENCRYPTION_KEY}, expose: true }`) and consume it on the other side.

## Readme-in-YAML is localized content

The user-facing readme lives **inside the YAML** under `spec.readme` (English) and again under each `localization.<locale>.readme`. There is no separate changelog file — the n8n template keeps its "Update Log" inside `spec.readme`. When you change deploy behavior, update the readme in **every** locale block, not just English.

Supported locales vary per template: en (in `spec`), zh-CN, zh-TW, ja-JP, id-ID, es-ES.

## Conventions

- Maintained by SEAFOODHOLDHAND. Issues URL consistently points to `github.com/glasschan/seafoodholdhand-zeabur-templates` — keep that link in readme content.
- Commit style: conventional commits (`feat:`, `fix:`, `docs:`), often scoped (`feat(multica):`).
- `.DS_Store` files exist in the tree historically — do not add new ones.

# DOX framework

- DOX is highly performant AGENTS.md hierarchy installed here
- Agent must follow DOX instructions across any edits

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees
- Work products, source materials, instructions, records, assets, and durable docs must stay understandable from the nearest applicable AGENTS.md plus every parent AGENTS.md above it

## Read Before Editing

1. Read the root AGENTS.md
2. Identify every file or folder you expect to touch
3. Walk from the repository root to each target path
4. Read every AGENTS.md found along each route
5. If a parent AGENTS.md lists a child AGENTS.md whose scope contains the path, read that child and continue from there
6. Use the nearest AGENTS.md as the local contract and parent docs for repo-wide rules
7. If docs conflict, the closer doc controls local work details, but no child doc may weaken DOX

Do not rely on memory. Re-read the applicable DOX chain in the current session before editing.

## Update After Editing

Every meaningful change requires a DOX pass before the task is done.

Update the closest owning AGENTS.md when a change affects:

- purpose, scope, ownership, or responsibilities
- durable structure, contracts, workflows, or operating rules
- required inputs, outputs, permissions, constraints, side effects, or artifacts
- user preferences about behavior, communication, process, organization, or quality
- AGENTS.md creation, deletion, move, rename, or index contents

Update parent docs when parent-level structure, ownership, workflow, or child index changes. Update child docs when parent changes alter local rules. Remove stale or contradictory text immediately. Small edits that do not change behavior or contracts may leave docs unchanged, but the DOX pass still must happen.

## Hierarchy

- Root AGENTS.md is the DOX rail: project-wide instructions, global preferences, durable workflow rules, and the top-level Child DOX Index
- Child AGENTS.md files own domain-specific instructions and their own Child DOX Index
- Each parent explains what its direct children cover and what stays owned by the parent
- The closer a doc is to the work, the more specific and practical it must be

## Child Doc Shape

- Create a child AGENTS.md when a folder becomes a durable boundary with its own purpose, rules, responsibilities, workflow, materials, or quality standards
- Work Guidance must reflect the current standards of the project or user instructions; if there are no specific standards or instructions yet, leave it empty
- Verification must reflect an existing check; if no verification framework exists yet, leave it empty and update it when one exists

Default section order:
- Purpose
- Ownership
- Local Contracts
- Work Guidance
- Verification
- Child DOX Index

## Style

- Keep docs concise, current, and operational
- Document stable contracts, not diary entries
- Put broad rules in parent docs and concrete details in child docs
- Prefer direct bullets with explicit names
- Do not duplicate rules across many files unless each scope needs a local version
- Delete stale notes instead of explaining history
- Trim obvious statements, repeated rules, misplaced detail, and warnings for risks that no longer exist

## Closeout

1. Re-check changed paths against the DOX chain
2. Update nearest owning docs and any affected parents or children
3. Refresh every affected Child DOX Index
4. Remove stale or contradictory text
5. Run existing verification when relevant
6. Report any docs intentionally left unchanged and why

## User Preferences

When the user requests a durable behavior change, record it here or in the relevant child AGENTS.md

## What this root owns vs. children

This root owns **repo-wide** rules: template-folder layout, the Zeabur Template schema, built-in variables, the `expose: true` wiring pattern, `dependencies:` placement, readme/localization conventions, and commit/URL conventions.

Each `*-for-zeabur/` child owns its **per-template contracts**: service inventory, image choices, internal architecture, env-var wiring specific to that template, login flow, and known gotchas. Read the child AGENTS.md before editing that template's YAML; read this root for anything that spans templates or introduces a new one.

## Child DOX Index

| Folder | Covers |
|--------|--------|
| [`multica-for-zeabur/`](./multica-for-zeabur/AGENTS.md) | 4-service Caddy-gateway template; `APP_URL` expose/consume wiring; email-verification login; Multica CLI post-deploy |
| [`n8n-w-worker-for-zeabur/`](./n8n-w-worker-for-zeabur/AGENTS.md) | 6-service n8n v2 + worker + external task runners; per-instance runner split; `N8N_ENCRYPTION_KEY` / `TASK_RUNNERS_AUTH_TOKEN` secret sharing; in-readme Update Log |
| [`learnhouse-for-zeabur/`](./learnhouse-for-zeabur/AGENTS.md) | 3-service all-in-one LMS; bundled nginx routing; Valkey-not-Redis (`valkey` DNS hostname); runtime `NEXT_PUBLIC_*` injection; SSR backend bypass; Gemini AI integration |
| [`onecli-for-zeabur/`](./onecli-for-zeabur/AGENTS.md) | 2-service credential vault; single-image two-process (:10254 dashboard + :10255 Rust gateway); auto-generated encryption-key volume is load-bearing; NextAuth single-user vs. Google OAuth |

All children are leaf folders (no nested DOX files).
