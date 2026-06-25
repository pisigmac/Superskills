---
name: testing-protocol
description: Use when defining or enforcing testing standards for a project.
---

# Testing Protocol

## Overview

Define, enforce, and verify testing standards for any project. Use project-specific thresholds from `skills/feature-factory/PROJECT_RULES.md` where they exist. Where no project rules are defined, use the defaults in this skill.

Untested code is undeployable code. A green build without proof is a lie.

## When to Use

- Defining testing standards for a project.
- Reviewing test coverage, gates, or CI configuration.
- Adding or updating tests.
- Investigating flaky tests or regressions.
- Preparing to merge or deploy.

## The Production Test Pyramid

Maintain tests at these layers. If any layer fails, stop the pipeline.

| Layer | Default Minimum | Purpose | Gate |
|-------|-----------------|---------|------|
| Unit | 90% business logic, 80% utilities, 100% auth/payment/PII | Pure functions, algorithms, state machines | Block PR |
| Integration | 80% API contracts, 100% external service wrappers | Databases, APIs, queues | Block merge |
| Contract | 100% public API surface | Schemas, backward compatibility | Block merge |
| Security | 100% auth/payment/PII paths | AuthZ, injection, secrets leakage | Block merge |
| Performance | 100% critical paths | Latency budgets, throughput | Block deploy |
| E2E / Smoke | 100% business-critical journeys | Sign-up → payment → core action → logout | Block deploy |
| Chaos / Resilience | 100% failure modes | Circuit breakers, retries, degradation | Monthly |
| Compliance | 100% regulated flows | Audit trails, retention, PII handling | Block deploy |

**Override values in `skills/feature-factory/PROJECT_RULES.md`.**

## Unit Tests

### Coverage

| Code Type | Minimum Line Coverage | Minimum Branch Coverage |
|-----------|----------------------:|------------------------:|
| Business logic | 90% | 85% |
| API controllers / handlers | 80% | 75% |
| Utility functions | 70% | 65% |
| Auth / payment / PII handling | 100% | 100% |
| Configuration / constants | 0% | 0% |

### What to Test

- All public methods and functions.
- Every conditional branch.
- Error handling paths.
- Boundary values: empty, null, max, malformed, overflow.
- Race conditions in concurrent code.
- Idempotency in retry logic.

### What NOT to Test

- Framework internals.
- Third-party SDKs (test your adapter/wrapper, not their code).
- Generated code (test the generator, not the output).
- Private methods. If they need testing, make them package-internal or test via the public API.

### Structure and Naming

Use `given-when-then` structure. Name tests:

```
test_<unit>_<condition>_<expected_outcome>
```

Examples:

- `test_auth_service_when_token_expired_then_raises_unauthorized`
- `test_payment_gateway_when_network_timeout_then_retries_3_times`
- `test_user_repository_when_email_duplicate_then_raises_conflict`

### Mutation Testing

Run mutation testing for auth, payment, and PII code. If mutants survive, the tests are too weak.

## Integration Tests

### Coverage

- 100% of API endpoints (every route, every method).
- 100% of database query paths, including pagination, filtering, sorting.
- 100% of external service integrations with contract stubs.
- 100% of message queue producers and consumers.

### Mocking Rules

- Mock external APIs only. Never mock your own code.
- Verify real API behavior with contract tests. Use Pact, WireMock, or recorded VCR cassettes.
- Fail the build if the real API contract changes.
- Use Testcontainers for databases. Never use in-memory databases for integration tests.

### Database Rules

- Use real database engines (PostgreSQL, MySQL, Redis) in Testcontainers. Do not use SQLite or H2.
- Run each test in a transaction and roll it back.
- Apply schema migrations once per suite, not per test.
- Seed data via factories. Never use raw SQL.

## Security Tests

Run these categories every PR. Failure blocks merge or deploy as noted.

| Category | Frequency | Gate |
|----------|-----------|------|
| SAST | Every PR | Block merge |
| Dependency scanning | Every PR | Block merge |
| Secrets detection | Every commit | Block merge |
| DAST | Weekly | Block deploy if critical |
| Container scanning | Every build | Block deploy if critical |
| Fuzzing | Monthly | Track findings |

### Required Coverage

- 100% line and branch coverage for auth and authZ code.
- 100% line and branch coverage for payment code.
- 100% line and branch coverage for PII handling code.

### Injection Tests

Test every query parameter for:

- SQL injection
- NoSQL injection
- Command injection
- XPath/LDAP injection if applicable
- Template injection

### PII Tests

- Verify PII is encrypted at rest.
- Verify PII is masked in logs.
- Verify PII is not returned in API responses unless explicitly requested.
- Verify audit logs capture access to PII.

## Performance Tests

### Latency Budgets (Defaults)

| Endpoint Type | P50 | P95 | P99 |
|---------------|----:|----:|----:|
| Read (GET) | <50ms | <100ms | <200ms |
| Write (POST/PUT) | <100ms | <200ms | <500ms |
| Report / Export | <1s | <3s | <10s |
| Auth / Payment | <100ms | <200ms | <300ms |

**Override in `skills/feature-factory/PROJECT_RULES.md`.**

### Test Types

| Type | Purpose | Frequency | Gate |
|------|---------|-----------|------|
| Load | Throughput under expected load | Before major release | Block deploy if P95 exceeds budget |
| Soak | Stability over time | Monthly | Track degradation |
| Stress | Find breaking point | Quarterly | Document limits |
| Spike | Sudden traffic bursts | Before campaigns | Block deploy if failure rate > 0.1% |

## E2E / Smoke Tests

### Coverage

- 100% of business-critical user journeys.
- 100% of revenue-critical flows.
- 100% of admin-critical flows.

### Isolation

- Each test creates its own user, data, and state.
- No shared state between tests.
- Reset database before each suite, not per test.

## Chaos and Resilience Tests

Run monthly. Verify:

- Killed database connections mid-transaction roll back correctly.
- Dropped external API responses trigger circuit breakers.
- Disk saturation triggers graceful degradation.
- Killed pods/containers restart with zero downtime.

## Compliance and Audit Tests

Verify regulatory requirements:

| Regulation | Test Focus |
|------------|-----------|
| GDPR | Right to deletion, data portability, consent tracking |
| SOC2 | Access controls, audit trails, change management |
| PCI-DSS | Card data never stored unencrypted, tokenization |
| HIPAA | PHI access logging, encryption at rest/transit |

Verify every sensitive action creates an audit log with: actor, target, timestamp, IP address, and reason.

## CI/CD Testing Pipeline

### Stage 1: Pre-commit

- Linting
- Type checking
- Unit tests for changed files only
- Secrets scan
- Max time: 60 seconds
- Gate: any failure blocks commit

### Stage 2: Pull Request

- Full unit test suite
- Integration tests
- SAST scan
- Dependency vulnerability scan
- Coverage diff (new code 90%+ covered)
- Max time: 10 minutes
- Gate: any failure blocks merge

### Stage 3: Pre-deploy

- E2E smoke tests
- Performance baseline check
- DAST scan
- Contract tests against real sandbox APIs
- Max time: 15 minutes
- Gate: any failure blocks deploy

### Stage 4: Post-deploy

- 5-minute smoke test
- Error rate monitoring (<0.1%)
- Latency monitoring (P95 within budget)
- Business metric verification
- Max time: 5 minutes
- Gate: any failure triggers automatic rollback

## Regression Prevention

For every production bug:

1. Reproduce with a failing test.
2. Fix the code.
3. Verify the test passes and no regressions exist.
4. Add the regression test to the permanent suite.
5. Write a post-mortem: why did tests miss this?
6. Update the test strategy to catch similar bugs.

## Flaky Test Policy

- Quarantine any flaky test immediately (move to `quarantine/`).
- Fix it within 4 hours of quarantine.
- If not fixed in 24 hours, delete it and write a replacement.
- Document the root cause: race condition, time dependency, external state, or test pollution.

## AI-Generated Code Testing

- Human writes tests for AI-generated implementation.
- Run mandatory mutation testing on AI-generated code in critical paths.
- Human reviews AI-generated tests before trusting them.

### Auto-Reject These Tests

- Tests asserting implementation details.
- Tests mocking the system under test.
- Tests with no edge cases.
- Tests that pass when logic is intentionally broken.
- Tests without assertions.

## Test Data Strategy

- Never use production data.
- Clone schema only.
- Generate fake data with factories.
- Use deterministic seeds for reproducibility.
- PII in test data must be obviously fake.

### Factory Rules

- One factory per entity type.
- Compose factories for complex scenarios.
- Reset database state between suites.
- Never share mutable state between tests.

## Coverage Enforcement

Default coverage gates:

| Layer | Minimum Line | Minimum Branch | Gate |
|-------|-------------:|---------------:|------|
| Unit — Business Logic | 90% | 85% | Block merge |
| Unit — Auth/Payment/PII | 100% | 100% | Block merge |
| Integration — API | 80% | 75% | Block merge |
| Integration — External Services | 100% | 90% | Block merge |
| E2E — Critical Paths | 100% | N/A | Block deploy |
| Security — All Categories | 100% | 100% | Block merge |

New code must maintain or improve overall coverage. Reject any PR that drops coverage by more than 1% unless explicitly approved.

**Override in `skills/feature-factory/PROJECT_RULES.md`.**

## Emergency Procedures

| Symptom | Action |
|---------|--------|
| Local suite >15 minutes | Split into fast (<2 min) and slow (<15 min) suites |
| Tests pass locally, fail in CI | Check timezone, database state, env vars, file paths, container networking. Use Testcontainers. |
| Cannot reproduce production bug locally | Add structured logging, reproduce in staging with production-like data. Never debug in production. |
| Test database corrupted | Reset containers and investigate root cause: schema drift, migration conflict, or test pollution. |

## Pre-Ship Checklist

Before claiming ready for merge or deploy:

- [ ] Unit tests: coverage met, all passing
- [ ] Integration tests: all API contracts verified
- [ ] Security tests: SAST/DAST clean, no secrets leaked
- [ ] Performance tests: P95 within budget
- [ ] E2E tests: 100% critical paths passing
- [ ] Compliance tests: audit trails verified
- [ ] Coverage diff: new code covered, overall not dropped
- [ ] Flaky tests: zero in main suite
- [ ] AI-generated code: human-reviewed and mutation-tested for critical paths
- [ ] Test names: describe behavior, not implementation
- [ ] Edge cases covered: empty, null, max, malformed, concurrent, timeout

## Red Flags

- Skipping a test layer because it is "too slow".
- Merging with failing or flaky tests.
- Using production data in tests.
- Mocking your own code.
- Writing tests after implementation without watching them fail.
- Trusting green tests without reading them.
- Debugging in production.

## Integration

**Required skills:**

- `superskills:test-driven-development` — write tests first
- `superskills:verification-before-completion` — verify before claiming tests pass
- `superskills:writing-plans` — define test strategy in the project plan
- `superskills:finishing-a-development-branch` — complete work only when all gates pass
