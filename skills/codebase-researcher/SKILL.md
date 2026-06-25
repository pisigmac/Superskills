---
name: codebase-researcher
description: Use when you need to understand an existing codebase before building a new feature.
---

# Codebase Researcher

Inspect the codebase and explain how things work before writing any code. This skill is read-only. Never edit files or run state-changing commands.

**Typical invoker:** `superskills:feature-factory-orchestrator` dispatches this skill after a feature request arrives and before planning or implementation begins.

**Core principle:** Understand first, build second. Every claim in the brief must point to evidence in the repository.

## When to Use

Use when you need to understand an existing codebase before building a new feature.

## Input

Receive from the invoker:

1. A rough feature description or request.
2. Any project rules file (for example, `PROJECT_RULES.md` or equivalent), if it exists.
3. The scope of the research: files, modules, or subsystems to focus on.

## The Process

Follow these steps in order. Use only read-only tools: `Read`, `Grep`, `Glob`. Never use `Edit`, `Write`, or state-changing `Bash` commands.

1. **Map the domain.** Find files related to the feature area: routes, services, models, jobs, utilities, tests, and configuration.
2. **Document patterns.** List conventions the project already uses: naming, folder structure, error handling, validation, authentication, authorization, logging, and dependency injection.
3. **Find prior art.** Identify similar features already built. Prefer copying their structure over inventing a new one.
4. **Flag risks and constraints.** Surface hidden complexity with evidence:
   - Timezone handling
   - Multi-tenant isolation
   - Retry / idempotency logic
   - Race conditions
   - Missing authentication or authorization
   - External API rate limits
   - Database migrations or schema changes
   - Required environment variables or feature flags
5. **Trace architecture.** Identify entry points, request flow, service boundaries, and data flow.
6. **Assess test impact.** List existing tests that will need updates and where new tests should live.
7. **Ask, do not assume.** If something is unclear, write it as an open question. Never guess.

## Output Format

Produce a concise research brief with exactly this structure:

```markdown
# Research Brief: {{FEATURE_NAME}}

## 1. Relevant Files
| File | Role | Why It Matters |
|------|------|----------------|
| `path/to/file.ts` | Role in system | Why it matters for this feature |

## 2. Patterns & Conventions
- **Pattern name:** Description with evidence (`file.ts`, line N).

## 3. Similar Features / Prior Art
- `path/to/similar-feature/` — What to reuse.

## 4. Architecture & Data Flow
- Entry point:
- Request flow:
- Data flow:
- Service boundaries:

## 5. Risks, Constraints & Open Questions
| Risk / Constraint | Severity | Evidence |
|-------------------|----------|----------|
| Description | High / Medium / Low | What you found |

### Open Questions
1. Question that must be answered before planning.

## 6. Tests to Update or Add
| Test File | Reason |
|-----------|--------|
| `path/to/test.ts` | Why it needs changes |

Research complete. Ready for planning.
```

## Rules

- Read-only only. Never edit files or run state-changing commands during research.
- Cite evidence for every claim. Prefer: "`file.ts` at line 42 does X." Never write "it probably works like X."
- If `PROJECT_RULES.md` or an equivalent rules file is missing, flag it as a project risk.
- If a file or behavior is ambiguous, list it and explain why you are unsure.
- Keep the brief concise. The goal is actionable understanding, not exhaustive documentation.
- End every brief with: "Research complete. Ready for planning."

## Example

**Bad:** "The invoice system probably uses a cron job for reminders."

**Good:** "`src/jobs/invoice.ts` uses `processInvoiceQueue` for background work. No reminder job exists. The queue is initialized in `src/lib/queue.ts` with a default retry policy of 3 attempts."

## Integration

**Typical invoker:**
- `superskills:feature-factory-orchestrator` — dispatches this skill before planning and implementation.

**Related skills:**
- `superskills:brainstorming` — Use first if the feature request is vague or the scope is unclear.
- `superskills:writing-plans` — Consumes this brief to create an implementation plan.
- `superskills:test-driven-development` — Uses the test impact section to guide new tests.
