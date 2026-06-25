# Feature Factory Integration Design

## Goal

Integrate the content from `new/software-factory-prompts-expanded/` and `new/mdfiles/` into Superskills as a set of composable skills. The factory model becomes an opt-in implementation strategy that is offered after `writing-plans` produces an approved implementation plan.

## New skills

### Entry point

- `skills/feature-factory-driven-development/SKILL.md`
  - Trigger: after `superskills:writing-plans` produces an approved plan.
  - Behavior: ask the user to choose between standard execution (`superskills:subagent-driven-development`) and the feature factory pipeline.
  - If subagents are unavailable, fall back to `superskills:executing-plans`.

### Factory pipeline

- `skills/feature-factory-orchestrator/SKILL.md`
  - Trigger: dispatched by `feature-factory-driven-development` when the user chooses the factory model.
  - Behavior: rewrite the plan into backend/frontend/shared task groups with dependency mapping, dispatch builders, run test-verifier, run validator, then hand off to `superskills:finishing-a-development-branch`.

- `skills/codebase-researcher/SKILL.md`
  - Trigger: dispatched by the orchestrator when the plan requires understanding an existing codebase.
  - Behavior: read-only codebase exploration, produce a research brief.

- `skills/backend-builder/SKILL.md`
  - Trigger: dispatched by the orchestrator when the plan contains backend tasks.
  - Behavior: implement backend tasks, compile/typecheck, write tests, produce a Build Summary.

- `skills/frontend-builder/SKILL.md`
  - Trigger: dispatched by the orchestrator when the plan contains frontend tasks.
  - Behavior: implement frontend tasks, compile/typecheck, write tests, produce a Build Summary.

- `skills/test-verifier/SKILL.md`
  - Trigger: dispatched by the orchestrator after all builders complete successfully.
  - Behavior: write and run acceptance tests; on failure, route back to the relevant builder.

- `skills/validator/SKILL.md`
  - Trigger: dispatched by the orchestrator after `test-verifier` passes.
  - Behavior: audit files, content, functionality, vulnerabilities, and security issues. On security findings, ask the user for clarification. On non-security findings, route back to the relevant builder.

### Protocol skills

- `skills/production-shipping-protocol/SKILL.md` — hard-gate production shipping rules from `new/mdfiles/CLAUDE.md`.
- `skills/testing-protocol/SKILL.md` — testing rules and gates from `new/mdfiles/TESTING.md`.
- `skills/deployment-protocol/SKILL.md` — deployment and rollback protocol from `new/mdfiles/DEPLOY.md`.

## Modified skills

- `skills/writing-plans/SKILL.md`
  - Update the plan header sub-skill reference from `superskills:subagent-driven-development` to `superskills:feature-factory-driven-development`.

## Reference files

- `skills/feature-factory/PROJECT_RULES.md` — project rules template.
- `skills/feature-factory/addons/` — 10 optional add-on agent prompts as reference files.

## Execution flow

```
user request
  ↓
superskills:brainstorming
  ↓
design approved
  ↓
superskills:writing-plans
  ↓
plan approved
  ↓
superskills:feature-factory-driven-development
  ↓
  ├─ Standard → superskills:subagent-driven-development
  └─ Factory → superskills:feature-factory-orchestrator
                  ↓
        rewrite plan into task groups
                  ↓
        dispatch codebase-researcher (if needed)
                  ↓
        dispatch backend-builder (if backend tasks exist)
        dispatch frontend-builder (if frontend tasks exist)
        (parallel where independent, sequential where dependent)
                  ↓
        on builder failure → builder corrects and reruns
                  ↓
        all builders pass
                  ↓
        dispatch test-verifier
                  ↓
        on test failure → route to relevant builder
                  ↓
        tests pass
                  ↓
        dispatch validator
                  ↓
        on security findings → ask user for clarification
        on other findings → route to relevant builder
                  ↓
        validation passes
                  ↓
        superskills:finishing-a-development-branch
```

## Builder requirements

- Implement only the scoped tasks.
- Run compile/typecheck/lint.
- Check for known vulnerabilities in package versions using the appropriate tool for the stack (e.g., `npm audit`, `pip-audit`, `cargo audit`, `bundle audit`).
  - If vulnerabilities are found, report severity, affected packages, and recommended fixes.
  - Do not silently auto-update major versions without user approval.
  - Patch/minor updates that do not break the API may be applied; major upgrades require user confirmation.
- Write unit and integration tests for added code.
- Produce a Build Summary with files changed, tests added, vulnerability scan results, and test results.
- On compilation, test, or critical vulnerability failure, correct and rerun before reporting completion.

## Test verifier requirements

- Wait for all builders to complete.
- Write acceptance tests mapped to the plan's acceptance criteria.
- Run the full test suite.
- On failure, return a report with failing criteria and route back to the relevant builder.

## Validator requirements

- Audit changed files against the approved plan.
- Check functionality, content, file structure, vulnerabilities, and security issues.
- On security findings, stop and ask the user for clarification.
- On non-security findings, route back to the relevant builder with specifics.

## Finishing handoff

After validation passes, the orchestrator invokes `superskills:finishing-a-development-branch`. No automatic commit or push happens in the factory flow.

## Verification

1. Every new skill has YAML frontmatter with `name` and `description`.
2. `description` fields follow the Superskills convention: third person, start with "Use when…".
3. All cross-skill references use `superskills:<skill-name>`.
4. `writing-plans` references `feature-factory-driven-development`.
5. No placeholders remain unresolved in skill frontmatter or core instructions.
