# TESTING.md — Product Developer Testing Protocol

Testing is not a phase. It is a continuous filter between "works on my machine" and "works in production."

---

## Philosophy

**Fast feedback > comprehensive coverage.**
A test that takes 30 seconds to run and catches 80% of bugs is worth more than a test suite that takes 10 minutes and catches 95%.

**Tests are executable specifications.**
If you can't write a test for it, you don't understand the requirement.

---

## Test Pyramid (Daily Shipping Edition)

| Layer | Ratio | Purpose | Speed |
|-------|-------|---------|-------|
| **Unit** | 70% | Logic, algorithms, pure functions | <100ms |
| **Integration** | 20% | API contracts, database queries, external services | <2s |
| **E2E / Smoke** | 10% | Critical user paths, deploy validation | <30s |

**Rule:** If your test takes >5 seconds, it belongs in CI, not your local loop.

---

## Unit Tests — The Daily Workhorse

### What to Test
- Business logic (calculations, state machines, transforms)
- Input validation and sanitization
- Edge cases (empty, null, max values, malformed data)
- Error handling paths

### What NOT to Test
- Framework internals (React rendering, Express routing)
- Third-party SDKs (test your wrapper, not their code)
- Simple getters/setters
- CSS/styling

### Structure: Arrange-Act-Assert
```python
def test_calculate_discount_applies_20_percent():
    # Arrange
    cart = Cart(items=[Item(price=100, qty=2)])

    # Act
    result = calculate_discount(cart, code="SAVE20")

    # Assert
    assert result == 40  # 20% of 200
```

### Naming Convention
`test_<function>_<condition>_<expected_result>`

Examples:
- `test_auth_login_valid_credentials_returns_token`
- `test_payment_process_insufficient_funds_raises_error`
- `test_export_csv_empty_list_returns_empty_file`

---

## Integration Tests — The Contract Police

### What to Test
- API endpoint contracts (request/response shape)
- Database queries return correct data
- External service integrations (with mocks for CI)
- Authentication/authorization flows

### Mocking Rules
1. **Mock external APIs, not your own code.**
2. **Record real responses once, replay forever.** (Use VCR, nock, or equivalent)
3. **Fail the test if the real API changes its contract.**

### Example: API Contract Test
```python
def test_create_user_returns_201_with_valid_payload(client):
    payload = {"email": "test@example.com", "password": "Secure123!"}

    response = client.post("/api/users", json=payload)

    assert response.status_code == 201
    assert "id" in response.json()
    assert "password" not in response.json()  # Never leak passwords
```

---

## E2E / Smoke Tests — The Safety Net

### What to Test
- User can sign up → log in → perform core action
- Payment flow completes end-to-end (use Stripe test keys)
- Critical admin functions work
- Deploy didn't break the homepage

### Tools by Stack
| Stack | Tool |
|-------|------|
| Web (React/Vue) | Playwright, Cypress |
| Mobile | Maestro, Detox |
| API-only | Postman/Newman, k6 |
| Full-stack | Playwright + API checks |

### Rule: The "3-Minute Smoke"
After every deploy, run a 3-minute smoke test. If it fails, roll back immediately.

---

## Test Data Strategy

### Never Use Production Data
- Clone schema, generate fake data
- Use factories (factory_boy, faker, @faker-js/faker)
- Seed data should be deterministic (same seed = same data)

### Fixture Rules
- One fixture per entity type
- Compose fixtures for complex scenarios
- Reset database state between tests (transaction rollback)

```python
# Good
@pytest.fixture
def active_user(db):
    return UserFactory(status="active")

@pytest.fixture
def premium_user(db):
    user = UserFactory(status="active")
    SubscriptionFactory(user=user, tier="premium")
    return user
```

---

## CI/CD Testing Pipeline

### Stage 1: Pre-commit (Local)
- Linting (eslint, ruff, black)
- Type checking (tsc, mypy)
- Unit tests for changed files only
- **Max time: 30 seconds**

### Stage 2: Pull Request (GitHub Actions/GitLab CI)
- Full unit test suite
- Integration tests
- Security scan (Snyk, Trivy)
- **Max time: 5 minutes**

### Stage 3: Pre-deploy (Staging)
- E2E smoke tests
- Performance baseline check
- Dependency vulnerability scan
- **Max time: 10 minutes**

### Stage 4: Post-deploy (Production)
- 3-minute smoke test
- Health check endpoints
- Error rate monitoring (Sentry, Datadog)
- **Max time: 3 minutes**

---

## Regression Prevention

### The Bug → Test Rule
Every production bug gets a test before the fix.

```
1. Reproduce the bug with a failing test
2. Fix the code
3. Verify the test passes
4. Commit test + fix together
```

### Flaky Test Policy
- A flaky test (fails randomly) is worse than no test
- If a test flakes twice, disable it and fix within 24 hours
- Common causes: time dependencies, race conditions, external state

---

## AI-Generated Code Testing

### Rule: Generated Code Gets Human Tests
- AI writes the implementation
- You write the test (or verify AI-generated tests)
- Never trust AI-generated tests without running them

### Red Flags in AI Tests
- Tests that assert on implementation details, not behavior
- Mocking everything (testing mocks, not code)
- Missing edge cases AI "forgot"
- Tests that pass even when you break the logic

---

## Coverage Targets (Pragmatic)

| Code Type | Target |
|-----------|--------|
| Business logic | 90%+ |
| API endpoints | 80%+ |
| Utility functions | 70%+ |
| UI components | 50%+ (test critical interactions) |
| Configuration | 0% (don't test config files) |

**Rule:** 100% coverage with weak tests is worse than 70% with strong tests.

---

## Emergency Procedures

**Test suite takes >10 minutes locally:**
→ Split into "fast" (local) and "slow" (CI) suites

**Tests pass locally but fail in CI:**
→ Check: timezone, database state, environment variables, file paths

**Can't reproduce a production bug locally:**
→ Add logging, reproduce in staging, use production clone (anonymized)

**Test database is corrupted:**
→ `docker-compose down -v && docker-compose up` (reset everything)

---

## Checklist: Before Marking "Done"

- [ ] Unit tests written for new logic
- [ ] Integration test for new API endpoint
- [ ] E2E test if this is a critical user path
- [ ] All tests pass locally
- [ ] CI pipeline passes
- [ ] No new warnings or lint errors
- [ ] Test names describe behavior, not implementation
- [ ] Edge cases covered (empty, null, max, malformed)

---

*Companion to CLAUDE.md v2.0 | Last updated: 2026-05-10*
