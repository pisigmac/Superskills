---
name: frontend-builder
description: Use when implementing frontend tasks in a feature factory pipeline.
---

# Frontend Builder

Implement the frontend half of a feature. Scope is limited to frontend folders, the approved brief, and the API contract.

## Input

Read before starting:

- Approved technical brief
- Codebase research summary
- Backend builder's summary (API contract) — read this first

## Scope

Operate only within frontend folders:

- Components, pages, and templates
- Client hooks and state management
- Shared UI utilities and design system components
- Frontend-specific tests and test helpers
- Styling and assets consumed by the UI

Do not touch:

- API routes, controllers, or backend services
- Database schema, migrations, or background jobs
- Any file outside the agreed scope in the brief

## Process

Follow in order:

1. **Re-read the brief and the API contract.** Confirm every endpoint, request/response shape, and UI behavior.
2. **If the API contract is wrong for the UI, report it as feedback.** Do not patch the backend. Document the mismatch and stop.
3. **Build components from the bottom up.** Start with small presentational components, then compose pages.
4. **Implement hooks and state.** Keep client state close to where it is used; derive UI state from props and fetched data.
5. **Add loading, success, and error states** for every async operation.
6. **Apply accessibility requirements.** Use semantic HTML, correct ARIA attributes, keyboard navigation, focus management, and sufficient color contrast.
7. **Follow project styling conventions.** Reuse design system components and CSS patterns; do not invent new styling without approval.
8. **Write unit tests** for every utility, hook, and presentational component.
9. **Write integration tests** for every page or feature flow covering success and at least one error path.
10. **Run compile / typecheck / lint.** Fix every failure.
11. **Run the test suite.** Fix every failure.
12. **Run a vulnerability audit** with the appropriate tool for the project:
    - Node.js: `npm audit`
    - Python: `pip-audit`
    - Rust: `cargo audit`
    - Ruby: `bundle audit`
    - Other ecosystems: use their standard audit tool
13. **Report vulnerabilities** with severity and recommended fix.
14. **Apply patch and minor security updates** without asking. Ask for confirmation before major upgrades.
15. **If compile, test, lint, or a critical vulnerability fails, correct and rerun** before reporting completion.
16. **Produce a Build Summary.**

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
|------|---------|
| ... | ... |

## API Contract Consumed
- Endpoint: ...
- Request / response shapes
- Mismatch notes (if any)

## UI States Handled
| State | Behaviour |
|-------|-----------|
| ... | ... |

## Patterns Reused
- ...

## Accessibility Notes
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
```

## Red Flags

**Never:**

- Touch backend code. Document API contract mismatches instead.
- Invent endpoints or response shapes. Use exactly what the backend documented.
- Skip loading, success, and error states on async operations.
- Skip compile / typecheck / lint.
- Skip the vulnerability audit.
- Ignore critical vulnerabilities.
- Apply major dependency upgrades without asking.
- Report completion while tests, compile, lint, or critical vulnerabilities are failing.

**Always:**

- Read the API contract before writing code.
- Write unit tests for added utilities, hooks, and components.
- Write integration tests for pages and feature flows.
- Apply patch/minor security updates without asking.
- Ask before major upgrades.
- Rerun compile / typecheck / lint / tests after any fix.

## Integration

**Required workflow skills:**

- **superskills:writing-plans** — Provides the implementation plan
- **superskills:codebase-researcher** — Understand existing frontend patterns
- **superskills:test-driven-development** — Drives tests before implementation
- **superskills:backend-builder** — Provides the API contract
- **superskills:requesting-code-review** — Review before handoff
- **superskills:finishing-a-development-branch** — Complete the branch after frontend work
