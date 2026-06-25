---
name: production-shipping-protocol
description: Use before shipping code to production to enforce hard-gate checks.
---

# Production Shipping Protocol

Run these gates before any production deploy. Ship only when every gate passes. Project-specific thresholds, commands, and checklists live in `skills/feature-factory/PROJECT_RULES.md`.

## P0 — Hard Gates (Ship Blockers)

Stop the deploy if any gate fails. Fix first. Ship never.

**Think before shipping.** State assumptions explicitly. If uncertain, ask before guessing. Present multiple interpretations when ambiguity exists. Push back when a simpler approach exists. Stop when confused and name what's unclear.

**Ship the minimum correct change.** No speculative code. No features beyond the requested scope. No abstractions for single-use code. If a senior engineer would call it overcomplicated, simplify it.

**Define success criteria first.** Loop until verified. Do not follow steps robotically. Strong success criteria let verification run independently.

**Production-ready or not at all.** "Works on my machine" is not a success criterion. Every change must pass unit tests, integration tests, security review, and performance baseline before merge. No "we'll fix it in post." No known production bugs.

**Observability is mandatory.** Every service, endpoint, and background job must emit:

- Structured logs (JSON, with trace IDs)
- Metrics (latency, throughput, error rate)
- Deep health checks (not just `return 200`)

If you cannot debug it in production without SSH, it is not done.

## P1 — Execution Discipline

**Treat token budgets as hard caps.** Per-task: 6,000 tokens. Per-session: 40,000 tokens. If approaching budget, checkpoint, summarize, and start fresh. Surface the breach. Do not silently overrun. Split complex features into sub-tasks before coding begins.

**Make surgical changes only.** Touch only what you must. Clean up only your own mess. Do not "improve" adjacent code, comments, or formatting. Do not refactor what isn't broken. Match existing style. Justify every change in the commit message.

**AI generates; human verifies.** Use AI for scaffolding, drafting, boilerplate, and error analysis. Require human review for security logic, concurrency, data integrity, and API contracts. AI-generated code in hot paths or auth/payment flows gets mandatory human review.

**Read before you write.** Before adding code, read exports, immediate callers, and shared utilities. "Looks orthogonal" is dangerous. If unsure why code is structured a certain way, ask. Understand the data flow before modifying it.

**Tests verify intent, not just behavior.** Tests must encode WHY behavior matters, not just WHAT it does. A test that cannot fail when business logic changes is wrong. Every bug fix ships with a regression test. Coverage targets: business logic 90%+, API contracts 80%+.

**Treat external dependencies as untrusted.** Every API call, SDK, and third-party service gets:

- Timeout (never default)
- Retry with exponential backoff + jitter
- Circuit breaker (fail fast when degraded)
- Graceful degradation (partial failure ≠ total failure)

Assume the API will fail during your busiest hour. Build accordingly.

**Every change must be reversible.** Use feature flags, database migrations with rollbacks, or branch-based deploys. If you cannot turn it off in under two minutes without a code deploy, it is not ready. Database migrations: additive only in one deploy. Deletions happen in a second deploy after dual-write verification.

## P2 — Quality Assurance

**Code review is mandatory.** No code merges without review. Even solo projects require self-review after a 24-hour cooldown. Review checklist: logic correctness, error handling, test coverage, security, performance, observability. Reject PRs that lack tests, docs, or a rollback plan.

**Documentation ships with code.** Every API gets an OpenAPI spec. Every function longer than ten lines gets a docstring. Every architectural decision gets an ADR. Update the README before merge. Update the changelog before deploy. Undocumented code is unmaintainable code.

**Security by default:**

- No secrets in code (use secret managers)
- Validate input at the edge; sanitize at persistence
- Encode output to prevent XSS
- Make SQL injection impossible (parameterized queries only)
- Check AuthZ on every endpoint (not just AuthN)
- Scan dependencies before every deploy

**Performance is a feature.** Every endpoint has a latency budget. Every query has an execution plan review. Ban N+1 queries, unbounded pagination, and full table scans in production. Load test before launch. Profile before optimizing.

## P3 — Operational Excellence

**Fail loud, fail fast.** "Completed" is wrong if anything was skipped silently. "Tests pass" is wrong if any were skipped. Default to surfacing uncertainty, not hiding it. Alert on anomalies, not just failures. A 10x latency spike is a failure even if it returns 200.

**Surface conflicts; do not average them.** If two patterns contradict, pick one (more recent or more tested). Explain why. Flag the other for cleanup. Do not blend conflicting patterns. Do not leave TODOs in production without a ticket.

**Checkpoint after every significant step.** Summarize what was done, what's verified, and what's left. Do not continue from a state you cannot describe. If you lose track, stop and restate.

**Match the codebase conventions, even if you disagree.** Conformance beats taste inside the codebase. If a convention is genuinely harmful, surface it via ADR. Do not fork silently.

## P4 — Incident Response

**Rollback is the first response.** When production breaks: rollback first, debug second. Fix-forward is allowed only if rollback is impossible and the fix takes under five minutes. Every deploy has a tested rollback command before it goes live.

**Fix the process after every incident.** Every incident gets a post-mortem within 48 hours. Track action items to completion. "Human error" is not a root cause — fix the system that allowed it.

## Context-Specific Overrides

For security-critical code (auth, payment, PII):

- AI-generated code is prohibited. Write by hand.
- Mandatory security review by a second pair of eyes.
- 100% branch coverage required.
- Feature flags mandatory. Kill switch must be under 30 seconds.

For data-intensive systems:

- Review query plans in staging with production-like data volume.
- Backpressure handling mandatory. Monitor queue depth.
- Metrics include data freshness, processing lag, and error partitioning.

For multi-tenant SaaS:

- Verify tenant isolation in every test suite.
- Rate limit per tenant, not just globally.
- Segment metrics by tenant tier.

## Emergency Escapes

- Stuck for more than 10 minutes: stop, state what's unclear, ask for help.
- Scope creeps mid-task: push back. New feature = new task = new design doc.
- A "quick fix" touches more than three files: stop and write a plan.
- Deleting a feature: feature flag off → wait seven days → delete code. Never comment-out and leave.
- Deploying on Friday: don't. Emergency fixes only. Saturday deploys require written justification.

## Shipping Checklist

Before marking ready for production:

1. All P0 hard gates pass.
2. All tests pass; none skipped.
3. Lint and type checks are clean.
4. No secrets, credentials, or debug tokens in code or config.
5. No console errors, debug logging, or dead code paths.
6. Observability (logs, metrics, health checks) is in place.
7. Security checklist complete (input validation, output encoding, AuthZ, dependency scan).
8. Rollback plan documented and tested.
9. README, changelog, and OpenAPI/ADR updates are complete.
10. Code review approved.

## Integration

**Required workflow skills:**

- **superskills:writing-plans** — Define the shipping plan and success criteria before execution.
- **superskills:test-driven-development** — Verify intent with tests before merging.
- **superskills:requesting-code-review** — Get mandatory review before shipping.
- **superskills:finishing-a-development-branch** — Complete the branch after all gates pass.
- **superskills:verification-before-completion** — Run verification commands and confirm output before claiming production-ready.
