---
name: backend-builder
description: Use when implementing backend tasks in a feature factory pipeline.
---

# Backend Builder

Implement the backend half of a feature. Scope is limited to backend folders and contracts.

## Input

Read before starting:

- Approved technical brief
- Codebase research summary
- Project rules (e.g., `PROJECT_RULES.md`)

## Scope

Operate only within backend folders:

- API routes / controllers
- Services / business logic
- Database schema and migrations
- Background jobs and queue configuration
- Shared backend utilities

Do not touch:

- Frontend components, templates, HTML, CSS
- Frontend hooks or state management
- Any file outside the agreed scope in the brief

## Process

Follow in order:

1. **Re-read the brief.** Confirm every data model change, API contract, and business rule.
2. **Implement the data layer first.** Migrations, schema changes, then repositories / models.
3. **Implement services.** Keep business logic out of routes.
4. **Implement API routes.** Validate input, call services, return structured errors.
5. **Implement background jobs** if the brief requires them.
6. **Write unit tests** for every service function.
7. **Write integration tests** for every API route covering success and at least one error path.
8. **Run compile / typecheck / lint.** Fix every failure.
9. **Run the test suite.** Fix every failure.
10. **Run a vulnerability audit** with the appropriate tool for the project:
    - Node.js: `npm audit`
    - Python: `pip-audit`
    - Rust: `cargo audit`
    - Ruby: `bundle audit`
    - Other ecosystems: use their standard audit tool
11. **Report vulnerabilities** with severity and recommended fix.
12. **Apply patch and minor security updates** without asking. Ask for confirmation before major upgrades.
13. **If compile, test, lint, or a critical vulnerability fails, correct and rerun** before reporting completion.
14. **Produce a Build Summary.**

## Build Summary

Produce a Build Summary with this structure:

```markdown
# Build Summary: {{FEATURE_NAME}}

## Files Added
| File | Purpose |
|------|---------|
| ... | ... |

## Files Modified
| File | Change |
|------|--------|
| ... | ... |

## Patterns Reused
- ...

## Project Rules Applied
- ...

## Tests Added
| Test | Type | Result |
|------|------|--------|
| ... | unit / integration | passing / failing |

## Vulnerability Scan Results
- Tool: ...
- Summary: ...
- Action taken: ...

## Test Results
- ...

## Notes for Frontend Builder
- Endpoint: ...
- Request / response shapes
- UI constraints or disabled states
```

## Red Flags

**Never:**

- Touch frontend code. Document needed UI changes in "Notes for Frontend Builder."
- Invent new dependencies without explicit approval.
- Expose raw errors or stack traces to the client.
- Skip tenant isolation on DB queries where applicable.
- Skip compile / typecheck / lint.
- Skip the vulnerability audit.
- Ignore critical vulnerabilities.
- Apply major dependency upgrades without asking.
- Report completion while tests, compile, lint, or critical vulnerabilities are failing.

**Always:**

- Write unit tests for every service function.
- Write integration tests for every API route covering success and at least one error path.
- Return structured errors to clients.
- Apply patch/minor security updates without asking.
- Ask before major upgrades.

## Integration

**Required workflow skills:**

- **superskills:writing-plans** — Provides the implementation plan
- **superskills:test-driven-development** — Drives tests before implementation
- **superskills:requesting-code-review** — Review before handoff
- **superskills:finishing-a-development-branch** — Complete the branch after backend work
