# TESTING.md — Production-Grade Testing Protocol v3.0

Testing is not validation. It is **proof that the system behaves correctly under all known conditions.**
Untested code is undeployable code.

---

## Philosophy

**Coverage without confidence is vanity.**
A test that passes but doesn't catch regressions is technical debt wearing a green badge.

**Every line of production code must justify its existence with a test.**
If you can't write a test for it, the code is too complex or the requirement is too vague.

---

## The Production Test Pyramid

| Layer | Target Coverage | Purpose | Gate |
|-------|-----------------|---------|------|
| **Unit** | 90%+ business logic, 80%+ utilities | Pure functions, algorithms, state machines | Must pass before PR |
| **Integration** | 80%+ API contracts, 100% external service wrappers | Database queries, API contracts, message queues | Must pass before merge |
| **Contract** | 100% public API surface | Request/response schemas, backward compatibility | Must pass before merge |
| **Security** | 100% auth, payment, PII paths | AuthZ, injection, XSS, secrets leakage | Must pass before merge |
| **Performance** | 100% critical paths | Latency budgets, throughput, resource saturation | Must pass before deploy |
| **E2E / Smoke** | 100% business-critical user journeys | Sign-up → payment → core action → logout | Must pass before deploy |
| **Chaos / Resilience** | 100% failure modes | Circuit breakers, retries, graceful degradation | Monthly gate |
| **Compliance** | 100% regulated flows | Audit trails, data retention, PII handling | Must pass before deploy |

**Rule:** If a layer fails, the pipeline stops. No exceptions.

---

## Unit Tests — The Foundation

### Coverage Requirements
| Code Type | Minimum Coverage | Branch Coverage |
|-----------|-----------------|-----------------|
| Business logic (calculations, transforms, state machines) | 90% | 85% |
| API controllers / handlers | 80% | 75% |
| Utility functions | 70% | 65% |
| Auth / payment / PII handling | 100% | 100% |
| Configuration / constants | 0% | 0% |

### What to Test
- All public methods and functions
- Every conditional branch (if/else, switch, ternary)
- Error handling paths (exceptions, error returns)
- Boundary values (min, max, empty, null, overflow)
- Race conditions in concurrent code
- Idempotency in retry logic

### What NOT to Test
- Framework internals (React rendering, Express routing)
- Third-party SDKs (test your adapter/wrapper, not their code)
- Generated code (test the generator, not the output)
- Private methods (if they need testing, make them package-internal or test via public API)

### Structure: Given-When-Then (BDD-style)
```python
def test_when_user_submits_valid_payment_then_order_is_created():
    # Given
    user = UserFactory(tier="premium")
    payment = PaymentMethodFactory(valid=True)

    # When
    result = OrderService.create(user, payment, amount=100)

    # Then
    assert result.status == "confirmed"
    assert result.transaction_id is not None
    assert AuditLog.exists(action="payment_processed", user_id=user.id)
```

### Naming Convention
`test_<unit>_<condition>_<expected_outcome>`

Examples:
- `test_auth_service_when_token_expired_then_raises_unauthorized`
- `test_payment_gateway_when_network_timeout_then_retries_3_times`
- `test_user_repository_when_email_duplicate_then_raises_conflict`

### Mutation Testing (Critical Paths)
For auth, payment, and PII code: run mutation testing (e.g., Stryker, MutPy).
If mutants survive, the tests are too weak.

---

## Integration Tests — The Contract Police

### Coverage Requirements
- 100% of API endpoints (every route, every method)
- 100% of database query paths (including pagination, filtering, sorting)
- 100% of external service integrations (with contract stubs)
- 100% of message queue producers and consumers

### Mocking Rules
1. **Mock external APIs only.** Never mock your own code.
2. **Contract tests verify real API behavior.** Use Pact, WireMock, or recorded VCR cassettes.
3. **Fail the build if the real API contract changes.** Integration tests must break when the provider changes.
4. **Stub databases with Testcontainers.** Never use in-memory databases for integration tests (they lie about behavior).

### API Contract Test Template
```python
def test_post_users_returns_201_with_valid_payload(client, db):
    payload = {
        "email": "test@example.com",
        "password": "SecurePass123!",
        "tos_accepted": True
    }

    response = client.post("/api/v1/users", json=payload)

    assert response.status_code == 201
    body = response.json()
    assert "id" in body and UUID(body["id"])
    assert "email" in body
    assert "password" not in body  # Never leak credentials
    assert "created_at" in body

    # Verify side effects
    assert db.query(User).filter_by(email=payload["email"]).count() == 1
    assert AuditLog.exists(action="user_created", user_id=body["id"])
```

### Database Integration Rules
- Use Testcontainers (PostgreSQL, MySQL, Redis) — not SQLite or H2
- Each test runs in a transaction, rolled back after
- Schema migrations applied once per test suite, not per test
- Seed data via factories, never raw SQL

---

## Security Tests — The Red Line

### Mandatory Security Test Categories

| Category | Tool Example | Frequency | Gate |
|----------|-------------|-----------|------|
| **Static Analysis (SAST)** | Semgrep, SonarQube, Bandit | Every PR | Block merge |
| **Dependency Scanning** | Snyk, OWASP Dependency-Check | Every PR | Block merge |
| **Secrets Detection** | GitLeaks, TruffleHog | Every commit | Block merge |
| **Dynamic Analysis (DAST)** | OWASP ZAP, Burp Suite | Weekly | Block deploy if critical |
| **Container Scanning** | Trivy, Clair | Every build | Block deploy if critical |
| **Fuzzing** | AFL, libFuzzer | Monthly | Track findings |

### Auth & AuthZ Tests (100% Coverage)
```python
def test_admin_endpoint_rejects_non_admin_user():
    user = UserFactory(role="user")
    token = generate_token(user)

    response = client.get("/api/v1/admin/users", headers={"Authorization": f"Bearer {token}"})

    assert response.status_code == 403

def test_payment_endpoint_validates_csrf_token():
    client = authenticated_client()

    response = client.post("/api/v1/payments", json={"amount": 100}, headers={"X-CSRF-Token": "invalid"})

    assert response.status_code == 403
```

### Injection Tests
- SQL injection attempts on every query parameter
- NoSQL injection on MongoDB filters
- Command injection on file upload paths
- XPath/LDAP injection if applicable
- Template injection on rendered views

### PII Handling Tests
- Verify PII is encrypted at rest
- Verify PII is masked in logs
- Verify PII is not returned in API responses unless explicitly requested
- Verify audit logs capture access to PII

---

## Performance Tests — The Latency Budget

### Latency Budgets (Per Endpoint)
| Endpoint Type | P50 Target | P95 Target | P99 Target |
|---------------|-----------|-----------|-----------|
| Read (GET) | <50ms | <100ms | <200ms |
| Write (POST/PUT) | <100ms | <200ms | <500ms |
| Report / Export | <1s | <3s | <10s |
| Auth / Payment | <100ms | <200ms | <300ms |

### Performance Test Types

| Type | Purpose | Frequency | Gate |
|------|---------|-----------|------|
| **Load Test** | Verify throughput under expected load | Before major release | Block deploy if P95 > budget |
| **Soak Test** | Verify stability over time (memory leaks, connection pool exhaustion) | Monthly | Track degradation |
| **Stress Test** | Find breaking point | Quarterly | Document limits |
| **Spike Test** | Handle sudden traffic bursts | Before marketing campaigns | Block deploy if failures > 0.1% |

### Performance Test Example (k6)
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up
    { duration: '5m', target: 100 },   // Steady state
    { duration: '2m', target: 200 },   // Spike
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'],   // P95 must be <200ms
    http_req_failed: ['rate<0.01'],     // Error rate <1%
  },
};

export default function () {
  const res = http.get('https://api.example.com/v1/orders');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });
  sleep(1);
}
```

---

## E2E / Smoke Tests — The Safety Net

### Coverage Requirements
- 100% of business-critical user journeys
- 100% of revenue-critical flows (signup → payment → invoice)
- 100% of admin-critical flows (user management, reporting)

### Test Isolation
- Each E2E test creates its own user, data, and state
- No shared state between tests
- Database reset before each test suite (not per test — too slow)

### E2E Example (Playwright)
```javascript
test('user can complete purchase end-to-end', async ({ page }) => {
  // Given
  const user = await createTestUser({ email: 'e2e@test.com', password: 'TestPass123!' });

  // When
  await page.goto('/login');
  await page.fill('[name="email"]', user.email);
  await page.fill('[name="password"]', user.password);
  await page.click('button[type="submit"]');
  await page.waitForURL('/dashboard');

  await page.goto('/products/123');
  await page.click('button[data-testid="add-to-cart"]');
  await page.goto('/checkout');
  await page.fill('[name="card-number"]', '4242 4242 4242 4242');
  await page.click('button[data-testid="place-order"]');

  // Then
  await page.waitForSelector('[data-testid="order-confirmation"]');
  const orderId = await page.getAttribute('[data-testid="order-id"]', 'data-value');

  // Verify backend state
  const order = await api.getOrder(orderId);
  expect(order.status).toBe('confirmed');
  expect(order.payment.status).toBe('succeeded');
});
```

---

## Chaos & Resilience Tests

### Monthly Chaos Engineering
- Kill database connections mid-transaction → verify rollback
- Drop 50% of external API responses → verify circuit breaker
- Fill disk to 95% → verify graceful degradation
- Kill random pods/containers → verify auto-restart and zero-downtime

### Resilience Test Example
```python
def test_payment_gateway_timeout_triggers_circuit_breaker():
    with mock.patch('requests.post', side_effect=TimeoutError):
        # First failure
        with pytest.raises(PaymentError):
            PaymentService.charge(amount=100)

        # Circuit should open
        with pytest.raises(CircuitBreakerOpen):
            PaymentService.charge(amount=100)

        # Verify fallback: order queued for retry
        assert RetryQueue.count() == 1
```

---

## Compliance & Audit Tests

### Regulatory Requirements
| Regulation | Test Focus |
|------------|-----------|
| **GDPR** | Right to deletion, data portability, consent tracking |
| **SOC2** | Access controls, audit trails, change management |
| **PCI-DSS** | Card data never stored unencrypted, tokenization verified |
| **HIPAA** | PHI access logging, encryption at rest/transit |

### Audit Trail Verification
```python
def test_sensitive_action_creates_audit_log():
    admin = UserFactory(role="admin")
    target_user = UserFactory()

    UserService.delete(target_user.id, performed_by=admin)

    log = AuditLog.find(action="user_deleted", target_id=target_user.id)
    assert log.performed_by == admin.id
    assert log.timestamp is not None
    assert log.ip_address is not None
    assert log.reason is not None  # Deletion must have justification
```

---

## CI/CD Testing Pipeline (Hard Gates)

### Stage 1: Pre-commit (Local)
- Linting (eslint, ruff, black, prettier)
- Type checking (tsc, mypy, pyright)
- Unit tests for changed files only
- Secrets scan (GitLeaks)
- **Max time: 60 seconds**
- **Gate:** Any failure blocks commit

### Stage 2: Pull Request
- Full unit test suite (90%+ coverage gate)
- Integration tests (Testcontainers)
- SAST scan (Semgrep)
- Dependency vulnerability scan (Snyk)
- Coverage diff (new code must be 90%+ covered)
- **Max time: 10 minutes**
- **Gate:** Any failure blocks merge

### Stage 3: Pre-deploy (Staging)
- E2E smoke tests (100% critical paths)
- Performance baseline check (P95 < budget)
- Security DAST scan (OWASP ZAP)
- Contract tests against real sandbox APIs
- **Max time: 15 minutes**
- **Gate:** Any failure blocks deploy

### Stage 4: Post-deploy (Production)
- 5-minute smoke test
- Error rate monitoring (must be <0.1%)
- Latency monitoring (P95 < budget)
- Business metric verification (orders, signups, payments flowing)
- **Max time: 5 minutes**
- **Gate:** Any failure triggers automatic rollback

---

## Regression Prevention

### The Bug → Test → Process Rule
Every production bug follows this sequence:
```
1. Reproduce with a failing test
2. Fix the code
3. Verify test passes + no regressions
4. Add regression test to permanent suite
5. Write post-mortem: why did tests miss this?
6. Update test strategy to catch similar bugs
```

### Flaky Test Policy (Zero Tolerance)
- A flaky test is immediately quarantined (moved to `quarantine/` directory)
- Must be fixed within 4 hours of quarantine
- If not fixed in 24 hours, delete it and write a replacement
- Root cause must be documented: race condition, time dependency, external state, or test pollution

---

## AI-Generated Code Testing

### Policy
- AI writes implementation → Human writes tests
- AI-generated code in critical paths gets **mandatory mutation testing**
- AI-generated tests get **mandatory human review** — never trust green tests without reading them

### Red Flags (Auto-Reject)
- Tests that assert on implementation details (e.g., `assert_called_with` on internal method)
- Tests that mock the system under test (testing mocks, not code)
- Tests with no edge cases (only happy path)
- Tests that pass when you intentionally break the logic (false positives)
- Tests without assertions (they "run" but prove nothing)

---

## Test Data Strategy

### Never Use Production Data
- Clone schema only
- Generate fake data with factories (factory_boy, faker, @faker-js/faker)
- Deterministic seeds (same seed = same data for reproducibility)
- PII in test data must be obviously fake (`test@example.com`, `4242-4242-4242-4242`)

### Factory Rules
- One factory per entity type
- Compose factories for complex scenarios
- Reset database state between test suites (transaction rollback per test)
- Never share mutable state between tests

```python
class UserFactory(factory.Factory):
    class Meta:
        model = User

    email = factory.Faker('email')
    password_hash = factory.LazyAttribute(lambda _: hash_password('TestPass123!'))
    status = 'active'
    created_at = factory.Faker('date_time')

class PremiumUserFactory(UserFactory):
    @factory.post_generation
    def subscription(obj, create, extracted, **kwargs):
        SubscriptionFactory(user=obj, tier='premium', status='active')
```

---

## Coverage Enforcement

### Coverage Gates
| Layer | Minimum | Branch | Gate |
|-------|---------|--------|------|
| Unit — Business Logic | 90% | 85% | Block merge |
| Unit — Auth/Payment/PII | 100% | 100% | Block merge |
| Integration — API | 80% | 75% | Block merge |
| Integration — External Services | 100% | 90% | Block merge |
| E2E — Critical Paths | 100% | N/A | Block deploy |
| Security — All Categories | 100% | 100% | Block merge |

### Coverage Diff Rule
New code must maintain or improve overall coverage.
A PR that drops coverage by >1% is rejected unless explicitly approved.

---

## Emergency Procedures

**Test suite takes >15 minutes locally:**
→ Split into `fast` (local, <2 min) and `slow` (CI, <15 min) suites

**Tests pass locally but fail in CI:**
→ Check: timezone, database state, env vars, file paths, container networking
→ Use Testcontainers to eliminate "works on my machine"

**Can't reproduce production bug locally:**
→ Add structured logging, reproduce in staging with production-like data
→ Use production clone (anonymized) for debugging
→ Never debug in production

**Test database corrupted:**
→ `docker-compose down -v && docker-compose up` (nuclear reset)
→ Investigate root cause: schema drift, migration conflict, or test pollution

---

## Pre-Ship Checklist

- [ ] Unit tests: 90%+ coverage, all passing
- [ ] Integration tests: all API contracts verified
- [ ] Security tests: SAST/DAST clean, no secrets leaked
- [ ] Performance tests: P95 within budget
- [ ] E2E tests: 100% critical paths passing
- [ ] Compliance tests: audit trails verified
- [ ] Coverage diff: new code 90%+, overall not dropped
- [ ] Flaky tests: zero in main suite
- [ ] AI-generated code: human-reviewed and mutation-tested (critical paths)
- [ ] Test names: describe behavior, not implementation
- [ ] Edge cases: empty, null, max, malformed, concurrent, timeout

---

## The Testing Creed

> We do not test to find bugs. We test to prove there are none.
> We do not ship because the build is green. We ship because we have verified every path a user can take, every failure mode they might hit, and every edge case they might trigger.
> A green build without confidence is a lie. A red build with clarity is progress.

---

*Companion to CLAUDE.md v3.0 — Production-Grade Shipping | Last updated: 2026-05-10*
