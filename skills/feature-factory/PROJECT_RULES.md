---
name: project-rules
description: Core project context loaded at the start of every AI session. This is the single source of truth for stack, conventions, and guardrails.
---

# Project Rules

## Identity
- Project name: {{PROJECT_NAME}}
- Primary language: {{PRIMARY_LANG}} (e.g., TypeScript, Python, Go, Rust)
- Framework: {{FRAMEWORK}} (e.g., Next.js, Django, FastAPI, Axum)
- Database: {{DB}} (e.g., PostgreSQL, MongoDB, SQLite)
- ORM / Query builder: {{ORM}} (e.g., Prisma, SQLAlchemy, Diesel)
- Job queue: {{QUEUE}} (e.g., BullMQ, Celery, Sidekiq)
- Email / Notifications: {{EMAIL}} (e.g., Resend, SendGrid, AWS SES)

## Commands
- Dev server: {{DEV_CMD}} (e.g., `npm run dev`, `python manage.py runserver`, `cargo run`)
- Test suite: {{TEST_CMD}} (e.g., `npm test`, `pytest`, `cargo test`)
- Lint / Typecheck: {{LINT_CMD}} (e.g., `npm run lint`, `ruff check`, `cargo clippy`)
- Database migrate: {{MIGRATE_CMD}} (e.g., `npx prisma migrate dev`, `alembic upgrade head`)
- Build for prod: {{BUILD_CMD}} (e.g., `npm run build`, `docker build -t app .`)

## Architecture Rules
1. Business logic lives in {{SERVICE_LAYER}}. API routes / controllers stay thin (< 50 lines).
2. All database access goes through {{DB_LAYER}}. No raw queries in handlers.
3. Background jobs use {{QUEUE}}. No cron scripts inside application code.
4. Authentication middleware runs before every protected route. Never skip it.
5. Tenant isolation check must exist on every data query in multi-tenant mode.
6. API errors return structured JSON: `{ error: string, code: string }`. Never expose raw stack traces.
7. All mutations must be idempotent where possible. Design for retries.

## What Not To Do
- Do not add cron jobs — use {{QUEUE}}.
- Do not log raw payment payloads, PII, or auth tokens.
- Do not expose raw errors or stack traces to clients.
- Do not store IDs in memory for long-lived processes.
- Do not skip migration files. Every schema change must be versioned.
- Do not add dependencies without explicit approval.
- Do not bypass the service layer from controllers / components.

## File Conventions
- Services: {{SERVICE_PATH}} (e.g., `/src/services`, `/app/services`)
- API routes / controllers: {{API_PATH}} (e.g., `/src/app/api`, `/app/routes`)
- Components / UI: {{UI_PATH}} (e.g., `/src/components`, `/app/templates`)
- Tests: {{TEST_PATH}} (e.g., alongside source as `*.test.ts`, or `/tests`)
- Shared utilities: {{UTIL_PATH}} (e.g., `/src/lib`, `/app/utils`)
- Database schema / migrations: {{DB_PATH}} (e.g., `/prisma`, `/migrations`)

## Pointers to Deeper Docs
- `docs/architecture.md` — system design and data flow
- `docs/billing.md` — payment and subscription logic
- `docs/security.md` — auth, RBAC, and audit rules
- `docs/deployment.md` — infra, env vars, and secrets
