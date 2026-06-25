---
name: feature-factory-orchestrator
description: Use when the user has chosen the feature factory pipeline and you need to coordinate builders, verifiers, and validators.
---

# Feature Factory Orchestrator

Coordinate the feature factory pipeline from an approved implementation plan through builders, test verification, and validation.

**Core principle:** Plan → research → build → verify → validate → finish. Each stage gates the next.

## The Process

```dot
digraph process {
    rankdir=TB;

    "Receive approved plan" [shape=box];
    "Rewrite plan into task groups" [shape=box];
    "Plan requires understanding existing code?" [shape=diamond];
    "Dispatch superskills:codebase-researcher" [shape=box];
    "backend-tasks exist?" [shape=diamond];
    "Dispatch superskills:backend-builder" [shape=box];
    "frontend-tasks exist?" [shape=diamond];
    "Dispatch superskills:frontend-builder" [shape=box];
    "All builders succeeded?" [shape=diamond];
    "Dispatch superskills:test-verifier" [shape=box];
    "Tests pass?" [shape=diamond];
    "Route failures to relevant builder" [shape=box];
    "Dispatch superskills:validator" [shape=box];
    "Security issues?" [shape=diamond];
    "Stop and ask user for clarification" [shape=box];
    "Non-security issues?" [shape=diamond];
    "Route issues to relevant builder" [shape=box];
    "Invoke superskills:finishing-a-development-branch" [shape=box style=filled fillcolor=lightgreen];

    "Receive approved plan" -> "Rewrite plan into task groups";
    "Rewrite plan into task groups" -> "Plan requires understanding existing code?";
    "Plan requires understanding existing code?" -> "Dispatch superskills:codebase-researcher" [label="yes"];
    "Dispatch superskills:codebase-researcher" -> "backend-tasks exist?";
    "Plan requires understanding existing code?" -> "backend-tasks exist?" [label="no"];
    "backend-tasks exist?" -> "Dispatch superskills:backend-builder" [label="yes"];
    "backend-tasks exist?" -> "frontend-tasks exist?" [label="no"];
    "Dispatch superskills:backend-builder" -> "frontend-tasks exist?";
    "frontend-tasks exist?" -> "Dispatch superskills:frontend-builder" [label="yes"];
    "frontend-tasks exist?" -> "All builders succeeded?" [label="no"];
    "Dispatch superskills:frontend-builder" -> "All builders succeeded?";
    "All builders succeeded?" -> "Dispatch superskills:test-verifier" [label="yes"];
    "All builders succeeded?" -> "Route failures to relevant builder" [label="no"];
    "Route failures to relevant builder" -> "Dispatch superskills:backend-builder";
    "Dispatch superskills:test-verifier" -> "Tests pass?";
    "Tests pass?" -> "Dispatch superskills:validator" [label="yes"];
    "Tests pass?" -> "Route failures to relevant builder" [label="no"];
    "Dispatch superskills:validator" -> "Security issues?";
    "Security issues?" -> "Stop and ask user for clarification" [label="yes"];
    "Security issues?" -> "Non-security issues?" [label="no"];
    "Non-security issues?" -> "Route issues to relevant builder" [label="yes"];
    "Non-security issues?" -> "Invoke superskills:finishing-a-development-branch" [label="no"];
    "Route issues to relevant builder" -> "Dispatch superskills:backend-builder";
}
```

## 1. Receive and Rewrite the Plan

Read the approved implementation plan. Extract every task. Rewrite the plan into exactly these groups:

- **backend-tasks:** Tasks that modify server-side code, APIs, databases, or infrastructure.
- **frontend-tasks:** Tasks that modify client-side code, UI, or user-facing behavior.
- **shared-tasks:** Tasks that touch both backend and frontend, or tasks that neither builder should own alone.
- **dependency-order:** Explicit ordering constraints between tasks (e.g., "Task B requires Task A").

Assign every task to exactly one group. Do not leave tasks ungrouped. Do not add groups.

## 2. Research Existing Code

If any task requires understanding existing code, dispatch `superskills:codebase-researcher` before any builder.

- Provide the full plan and the rewritten task groups.
- Request a summary of relevant existing code, patterns, and constraints.
- Use the research output to refine builder instructions. Do not let builders rediscover this context on their own.

## 3. Dispatch Builders

Dispatch builders only when their task groups are non-empty.

- **Dispatch `superskills:backend-builder`** only if `backend-tasks` contains tasks.
- **Dispatch `superskills:frontend-builder`** only if `frontend-tasks` contains tasks.

**Parallelization rules:**

- Run backend and frontend builders in parallel when their tasks are independent.
- Respect `dependency-order`. Dependent tasks run sequentially, even within the same builder.
- Shared tasks may need to run before, after, or alongside builders. Use `dependency-order` to decide. When in doubt, run shared tasks before dependent builder tasks.

**Builder instructions must include:**

- The exact tasks assigned to that builder.
- The full dependency order that affects those tasks.
- The acceptance criteria from the plan.
- A reminder to report DONE, DONE_WITH_CONCERNS, NEEDS_CONTEXT, or BLOCKED.

## 4. Verify Tests

After all builders report success, dispatch `superskills:test-verifier`.

- Provide the full plan and the acceptance criteria.
- Wait for the verifier's report.

**If tests fail:**

- Identify which builder owns the failing code or criteria.
- Route the failing criteria back to that builder.
- Re-run the verifier only after the builder reports success.
- Do not proceed to validation with failing tests.

## 5. Validate

After tests pass, dispatch `superskills:validator`.

- Provide the full plan, the implementation, and the test results.
- Wait for the validator's report.

**If the validator finds security issues:**

- Stop immediately.
- Ask the user for clarification before continuing.
- Do not route security issues back to a builder without user input.

**If the validator finds non-security issues:**

- Route the issues back to the relevant builder.
- Re-run tests after the builder reports success.
- Re-run the validator after tests pass.

## 6. Finish

After validation passes, invoke `superskills:finishing-a-development-branch`.

- Do not commit or push automatically.
- Let the finishing skill present options and execute the user's choice.

## Hard Rules

**Never:**

- Skip the codebase-researcher step when the plan depends on existing code.
- Dispatch a builder when its task group is empty.
- Run dependent tasks in parallel.
- Proceed to validation while tests are failing.
- Route security issues from the validator to a builder without user approval.
- Commit or push automatically.

**Always:**

- Rewrite the plan into the four required groups.
- Preserve dependency order across all dispatches.
- Re-run the verifier after any builder fixes failures.
- Stop and ask the user when the validator reports security issues.
- Use `superskills:finishing-a-development-branch` to complete the work.

## Integration

**Required workflow skills:**

- **superskills:codebase-researcher** — Understand existing code before building.
- **superskills:backend-builder** — Implement backend tasks.
- **superskills:frontend-builder** — Implement frontend tasks.
- **superskills:test-verifier** — Verify all tests pass.
- **superskills:validator** — Validate implementation quality and security.
- **superskills:finishing-a-development-branch** — Complete development after validation.

**Related workflow skills:**

- **superskills:feature-factory** — Choose the feature factory pipeline.
- **superskills:writing-plans** — Produce the approved implementation plan this skill executes.
