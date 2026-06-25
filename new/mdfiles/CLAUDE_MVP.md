# CLAUDE.md — Product Developer Protocol v2.0

These rules apply to every task in this project unless explicitly overridden.
Bias: **caution on core logic, velocity on validation work.** Use judgment.

---

## P0 — Non-Negotiables (Break These = Broken Product)

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

### Rule 4 — Ship the Validation, Not the Perfection
If a feature proves the hypothesis (users want X), ship it with known rough edges. Mark TODOs for polish.
Perfection is for retention features; validation is for new features.
Never let "could be cleaner" block "proves the concept."

### Rule 5 — User Signal > Code Elegance
If analytics show a "messy" feature is heavily used, optimize it in place.
If a "beautiful" feature has zero usage, kill it regardless of code quality.
The codebase serves the user, not the other way around.

---

## P1 — Execution Discipline (Break These = Slow Death)

### Rule 6 — Token Budgets Are Guardrails, Not Shackles
Per-task: 8,000 tokens (complex features) / 2,000 tokens (tweaks).
Per-session: 50,000 tokens.
If approaching budget, checkpoint and start fresh — but don't fragment a coherent feature across sessions just to stay under budget.
Surface the breach. Do not silently overrun.

### Rule 7 — Surgical Changes (with Product Exception)
Touch only what you must. Clean up only your own mess.
Don't "improve" adjacent code, comments, or formatting.
Don't refactor what isn't broken.
**Exception:** If a feature is being killed based on product signal (user feedback, metrics, pivot), aggressive removal is allowed. Document the "why" in the commit message.

### Rule 8 — Use AI for Acceleration, Not as a Crutch
Use AI for: judgment calls, drafting, complex logic scaffolding, configuration generation, error analysis.
Use code for: hot paths, deterministic core algorithms, security-critical logic.
If code can answer *and* it's faster to write than verify AI output, code answers. Otherwise, generate + verify.

### Rule 9 — Read Before You Write
Before adding code, read exports, immediate callers, shared utilities.
"Looks orthogonal" is dangerous. If unsure why code is structured a way, ask.

### Rule 10 — Tests Verify Intent, Not Just Behavior
Tests must encode WHY behavior matters, not just WHAT it does.
A test that can't fail when business logic changes is wrong.

### Rule 11 — External Dependencies Are Suspicious Until Proven
Every API call, SDK, and third-party service gets: timeout handling, retry logic, circuit breaker, and graceful degradation.
Assume the API will fail during your demo. Build accordingly.

### Rule 12 — Every Significant Change Must Be Reversible
Feature flags, database migrations with rollbacks, or branch-based deploys.
If you can't turn it off in <5 minutes without a code deploy, it's not ready for production.

---

## P2 — Operational Hygiene (Break These = Technical Debt)

### Rule 13 — Surface Conflicts, Don't Average Them
If two patterns contradict, pick one (more recent / more tested).
Explain why. Flag the other for cleanup.
Don't blend conflicting patterns.

### Rule 14 — Checkpoint After Every Significant Step
Summarize what was done, what's verified, what's left.
Don't continue from a state you can't describe back.
If you lose track, stop and restate.

### Rule 15 — Match the Codebase's Conventions, Even If You Disagree
Conformance > taste inside the codebase.
If you genuinely think a convention is harmful, surface it. Don't fork silently.

### Rule 16 — Fail Loud
"Completed" is wrong if anything was skipped silently.
"Tests pass" is wrong if any were skipped.
Default to surfacing uncertainty, not hiding it.

---

## P3 — Context-Specific Overrides

### For Prototypes / MVPs
- Rules 4 and 5 override Rule 2. Ship fast, refactor later.
- Rule 12 can be simplified to "branch-based deploys" only.
- Rule 10 checkpoints can be verbal summaries, not written docs.

### For Production / Revenue-Critical Code
- Rule 2 overrides Rule 4. No shipping with known rough edges.
- Rule 11 is mandatory, not optional.
- Rule 12 requires feature flags + database rollback scripts.
- Rule 8: AI-generated code gets human review before merge.

### For AI/ML Features
- Rule 8: Model inference code is "hot path" — write by hand, don't generate.
- Rule 11: LLM APIs get exponential backoff, prompt caching, and fallback models.
- Rule 5: A/B test every model change. "Better" metrics > "cleaner" code.

---

## Emergency Escapes

**When stuck for >10 minutes:** Stop. State what's unclear. Ask for help.
**When scope creeps mid-task:** Push back. New feature = new task.
**When a "quick fix" touches >3 files:** It's not quick. Stop and plan.
**When deleting a feature:** Use `git revert` or feature flag. Never comment-out and leave.

---

*Last updated: 2026-05-10 | Context: AI product development, daily shipping, multi-project environment*
