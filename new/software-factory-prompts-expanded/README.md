# Software Factory Prompts

Production-grade, stack-agnostic prompts for the 7-agent software factory architecture.

## What's Included

| File | Role |
|------|------|
| `PROJECT_RULES.md` | Template for your project's single source of truth |
| `agents/01_Researcher.md` | Maps codebase before building (read-only) |
| `agents/02_Story_Writer.md` | Turns ideas into approved user stories (read-only) |
| `agents/03_Spec_Writer.md` | Turns stories into technical briefs (read-only) |
| `agents/04_Backend_Builder.md` | Builds backend: API, services, DB, jobs, tests |
| `agents/05_Frontend_Builder.md` | Builds UI: components, pages, hooks, tests |
| `agents/06_Test_Verifier.md` | Writes acceptance tests, reports pass/fail |
| `agents/07_Validator.md` | Audits implementation against story & brief (read-only) |
| `skills/ORCHESTRATOR.md` | Wires the chain, manages checkpoints & loop-backs |

## How to Use

1. Copy `PROJECT_RULES.md` to your repo root. Fill in the `{{PLACEHOLDERS}}`.
2. For each agent, copy the prompt into a new AI session / subagent / custom GPT.
3. Feed the Orchestrator prompt to your main session to manage the chain.
4. Run one small feature end-to-end. Tune the prompts based on where it stumbles.

## Customization

Every prompt uses `{{PLACEHOLDER}}` syntax for stack-specific values. Replace these once:
- `{{PRIMARY_LANG}}`, `{{FRAMEWORK}}`, `{{DB}}`, `{{ORM}}`, `{{QUEUE}}`
- `{{DEV_CMD}}`, `{{TEST_CMD}}`, `{{LINT_CMD}}`, `{{MIGRATE_CMD}}`
- `{{API_PATH}}`, `{{SERVICE_PATH}}`, `{{UI_PATH}}`, `{{TEST_PATH}}`, `{{DB_PATH}}`

## License
Use freely. Modify for your team. No attribution required.

## Optional Add-On Agents (Insert Where Needed)

| # | Agent | When to Insert | What It Guards Against |
|---|-------|----------------|------------------------|
| 08 | **Security Auditor** | After Test Verifier, before Validator | Auth bypass, injection, secrets leaks, XSS, CSRF |
| 09 | **Performance Engineer** | After Backend Builder | N+1 queries, missing indexes, memory leaks, unbounded lists |
| 10 | **Database Safety Reviewer** | After Spec Writer approval, before Backend Builder | Destructive migrations, table locks, data loss |
| 11 | **Documentation Writer** | After Validator approval | Missing API docs, stale README, no runbooks |
| 12 | **Observability Engineer** | After Backend + Frontend Builders | Blind production — no logs, metrics, or alerts |
| 13 | **DevOps Deployer** | After Documentation | Broken deployments, missing env vars, no rollback plan |
| 14 | **Accessibility Auditor** | After Frontend Builder | WCAG violations, keyboard traps, screen reader failures |
| 15 | **API Contract Guardian** | After Backend Builder | Breaking API changes, stale OpenAPI, inconsistent errors |
| 16 | **Analytics Events Tracker** | After Frontend Builder | No product data, unmeasured features, broken funnels |
| 17 | **Regression Guardian** | After Test Verifier | New code breaking old features, bundle bloat, perf degradation |

## Expanded Pipeline

```
Researcher → Story Writer → [Human] → Spec Writer → [Human]
    ↓
Database Safety Reviewer → [Human]
    ↓
Backend Builder → Performance Engineer → API Contract Guardian
    ↓
Frontend Builder → Accessibility Auditor → UX Copywriter → Analytics Tracker
    ↓
Test Verifier → Regression Guardian → Security Auditor → Validator
    ↓
Observability Engineer → Documentation Writer → DevOps Deployer
    ↓
[Human: Approve PR] → Merge & Deploy
```

Pick the agents that matter for your project. A solo founder might only need 08 + 10 + 14. A team serving enterprise clients probably wants all of them.
