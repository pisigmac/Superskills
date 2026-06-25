# CLAUDE.md — Production-Grade Shipping Protocol v3.0

These rules apply to every task unless explicitly overridden by a signed-off exception.
Bias: **correctness over speed. Zero tolerance for silent failures.**

---

## P0 — Hard Gates (Violate = Do Not Ship)

### Rule 1 — Think Before Coding
State assumptions explicitly. If uncertain, ask rather than guess.
Present multiple interpretations when ambiguity exists.
Push back when a simpler approach exists.
Stop when confused. Name what's unclear.

### Rule 2 — Simplicity First
Minimum code that solves the problem. Nothing speculative.
No features beyond what was asked. No abstractions for single-use code.
Test: would a senior engineer say this is overcomplicated? If yes, simplify.

### Rule 3 — Goal-Driven Execution
Define success criteria before writing code. Loop until verified.
Don't follow steps robotically. Define success and iterate.
Strong success criteria let you loop independently.

### Rule 4 — Production-Ready or Not at All
"It works on my machine" is not a success criterion.
Every feature must pass: unit tests, integration tests, security review, and performance baseline before merge.
No "we'll fix it in post." No known bugs in production.

### Rule 5 — Observability Is Not Optional
Every service, endpoint, and background job must emit:
- Structured logs (JSON, with trace IDs)
- Metrics (latency, throughput, error rate)
- Health checks (deep, not just `return 200`)
If you can't debug it in production without SSH, it's not done.

---

## P1 — Execution Discipline (Violate = Technical Debt)

### Rule 6 — Token Budgets Are Hard Caps
Per-task: 6,000 tokens. Per-session: 40,000 tokens.
If approaching budget, checkpoint, summarize, and start fresh.
Surface the breach. Do not silently overrun.
Complex features get split into sub-tasks before coding begins.

### Rule 7 — Surgical Changes Only
Touch only what you must. Clean up only your own mess.
Don't "improve" adjacent code, comments, or formatting.
Don't refactor what isn't broken. Match existing style.
Every change must be justified in the commit message.

### Rule 8 — AI Generates, Human Verifies
Use AI for: scaffolding, drafting, boilerplate, error analysis.
Use human review for: security logic, concurrency, data integrity, API contracts.
AI-generated code in hot paths or auth/payment flows gets mandatory human review.

### Rule 9 — Read Before You Write
Before adding code, read exports, immediate callers, shared utilities.
"Looks orthogonal" is dangerous. If unsure why code is structured a way, ask.
Understand the data flow before modifying it.

### Rule 10 — Tests Verify Intent, Not Just Behavior
Tests must encode WHY behavior matters, not just WHAT it does.
A test that can't fail when business logic changes is wrong.
Every bug fix ships with a regression test.
Coverage target: business logic 90%+, API contracts 80%+.

### Rule 11 — External Dependencies Are Untrusted
Every API call, SDK, and third-party service gets:
- Timeout (never default)
- Retry with exponential backoff + jitter
- Circuit breaker (fail fast when degraded)
- Graceful degradation (partial failure ≠ total failure)
Assume the API will fail during your busiest hour. Build accordingly.

### Rule 12 — Every Change Must Be Reversible
Feature flags, database migrations with rollbacks, or branch-based deploys.
If you can't turn it off in <2 minutes without a code deploy, it's not ready.
Database migrations: additive only in one deploy. Deletions happen in a second deploy after dual-write verification.

---

## P2 — Quality Assurance (Violate = Risk)

### Rule 13 — Code Review Is Mandatory
No code merges without review. Even solo projects: self-review after 24-hour cooldown.
Review checklist: logic correctness, error handling, test coverage, security, performance, observability.
Reject PRs that lack tests, docs, or rollback plan.

### Rule 14 — Documentation Ships With Code
Every API gets OpenAPI spec. Every function >10 lines gets a docstring.
Every architectural decision gets an ADR (Architecture Decision Record).
README is updated before merge. Changelog is updated before deploy.
Undocumented code is unmaintainable code.

### Rule 15 — Security by Default
- No secrets in code (use secret managers)
- Input validated at the edge, sanitized at persistence
- Output encoded to prevent XSS
- SQL injection impossible (parameterized queries only)
- AuthZ checked on every endpoint (not just authN)
- Dependencies scanned before every deploy

### Rule 16 — Performance Is a Feature
Every endpoint has a latency budget. Every query has an execution plan review.
N+1 queries are banned. Unbounded pagination is banned. Full table scans in production are banned.
Load test before launch. Profile before optimize.

---

## P3 — Operational Excellence (Violate = Incident)

### Rule 17 — Fail Loud, Fail Fast
"Completed" is wrong if anything was skipped silently.
"Tests pass" is wrong if any were skipped.
Default to surfacing uncertainty, not hiding it.
Alert on anomalies, not just failures. A 10x latency spike is a failure even if it returns 200.

### Rule 18 — Surface Conflicts, Don't Average Them
If two patterns contradict, pick one (more recent / more tested).
Explain why. Flag the other for cleanup.
Don't blend conflicting patterns. Don't leave TODOs in production without a ticket.

### Rule 19 — Checkpoint After Every Significant Step
Summarize what was done, what's verified, what's left.
Don't continue from a state you can't describe back.
If you lose track, stop and restate.

### Rule 20 — Match the Codebase's Conventions, Even If You Disagree
Conformance > taste inside the codebase.
If you genuinely think a convention is harmful, surface it via ADR. Don't fork silently.

---

## P4 — Incident Response (Violate = Downtime)

### Rule 21 — Rollback Is the First Response
When production breaks: rollback first, debug second.
Fix-forward is only allowed if rollback is impossible AND the fix is <5 minutes.
Every deploy has a tested rollback command before it goes live.

### Rule 22 — Post-Incident, Fix the Process
Every incident gets a post-mortem within 48 hours.
Action items are tracked to completion. "Human error" is not a root cause — fix the system that allowed it.

---

## Context-Specific Overrides

### For Security-Critical Code (Auth, Payment, PII)
- Rule 8: AI-generated code is prohibited. Write by hand.
- Rule 15: Mandatory security review by second pair of eyes.
- Rule 10: 100% branch coverage required.
- Rule 12: Feature flags mandatory. Kill switch must be <30 seconds.

### For Data-Intensive Systems
- Rule 16: Query plans reviewed in staging with production-like data volume.
- Rule 11: Backpressure handling mandatory. Queue depth monitored.
- Rule 5: Metrics include data freshness, processing lag, and error partitioning.

### For Multi-Tenant SaaS
- Rule 15: Tenant isolation verified in every test suite.
- Rule 11: Rate limiting per tenant, not just global.
- Rule 5: Metrics segmented by tenant tier.

---

## Emergency Escapes

**When stuck for >10 minutes:** Stop. State what's unclear. Ask for help.
**When scope creeps mid-task:** Push back. New feature = new task = new design doc.
**When a "quick fix" touches >3 files:** It's not quick. Stop and write a plan.
**When deleting a feature:** Use feature flag off → wait 7 days → delete code. Never comment-out and leave.
**When deploying on Friday:** Don't. Emergency fixes only. Saturday deploys require written justification.

---

## The Production Creed

> We do not ship to see if it works.
> We ship because we know it works.
> We know it works because we tested it, reviewed it, monitored it, and planned for when it breaks.

---

*Last updated: 2026-05-10 | Context: Production-grade shipping, zero-tolerance quality, enterprise SaaS, multi-tenant systems*
