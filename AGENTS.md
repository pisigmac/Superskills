# Superskills — Agent Guide

This file is for AI agents working in the `superskills` repository. Read it before you read any other file or take any action.

## If you are an AI agent, read this first

This repository has a 94% PR rejection rate. Almost every rejected PR was submitted by an agent that did not read or did not follow the contributor guidelines. Your job is to protect your human partner from that outcome.

Before you edit, create, or submit anything, understand the project and follow the rules below. When in doubt, stop and ask your human partner.

## Project overview

Superskills is a software-development methodology distributed as a plugin for coding agents. It provides a bootstrap that loads at session start and a library of composable skills that trigger automatically during development work.

- **Name:** `superskills`
- **Version:** `5.1.0` (check `package.json`)
- **License:** MIT
- **Target harnesses:** Claude Code, Codex CLI, Codex App, Cursor, OpenCode, Gemini CLI, GitHub Copilot CLI, Factory Droid
- **Core idea:** Mandatory, auto-triggering skills that enforce brainstorming → planning → TDD → review → finish.

## Directory structure

```
assets/           # App icon and logo
  app-icon.png
  app-icon.svg
  superskills-small.svg

docs/             # Human and agent documentation
  plans/          # Historical design and implementation plans
  superskills/    # Default output directory for generated specs and plans
  windows/        # Cross-platform hook documentation
  README.opencode.md
  testing.md

hooks/            # Session-start hooks that inject the using-superskills bootstrap
  hooks.json
  hooks-cursor.json
  run-hook.cmd
  session-start

scripts/          # Maintenance scripts
  bump-version.sh
  sync-to-codex-plugin.sh

skills/           # Skill library; one directory per skill
  <skill-name>/
    SKILL.md
    ...supporting files...

tests/            # Test suites organized by harness or feature
  brainstorm-server/
  claude-code/
  codex-plugin-sync/
  explicit-skill-requests/
  opencode/
  skill-triggering/
  subagent-driven-dev/
```

Some files and directories referenced by scripts and tests are not present in this working copy because they are harness-specific or git metadata: `.git/`, `.github/`, `.cursor-plugin/`, `.opencode/`, `.codex-plugin/`, `.version-bump.json`, `.gitignore`, `.gitattributes`. Do not recreate them unless your task explicitly requires it.

## Skill conventions

Skills live in `skills/<kebab-name>/SKILL.md`.

- Each `SKILL.md` must have YAML frontmatter with `name` and `description`.
- `description` must start with "Use when…", be written in the third person, and describe the triggering condition only. Never summarize the workflow in the description.
- Use `superskills:<skill-name>` when cross-referencing skills.
- Keep the core plugin zero-dependency. External tools belong in separate plugins.
- Skill content shapes agent behavior; treat it as carefully as code. Do not reword Red Flags tables, rationalization lists, or "human partner" language without eval evidence.

## New skill areas

### Feature Factory

The `skills/feature-factory*/` directories contain an opt-in implementation pipeline:

- `skills/feature-factory-driven-development/` — entry point offered after `writing-plans`.
- `skills/feature-factory-orchestrator/` — coordinates builders, verifiers, and validators.
- `skills/codebase-researcher/`, `skills/backend-builder/`, `skills/frontend-builder/`, `skills/test-verifier/`, `skills/validator/` — factory roles.
- `skills/production-shipping-protocol/`, `skills/testing-protocol/`, `skills/deployment-protocol/` — hard-gate protocols.
- `skills/feature-factory/PROJECT_RULES.md` — project rules template.
- `skills/feature-factory/addons/` — optional add-on agent prompts (not standalone skills).

`skills/writing-plans/SKILL.md` now dispatches `superskills:feature-factory-driven-development` after a plan is approved, which lets the user choose between standard execution and the factory pipeline.

## Development workflows

Run the right test suite for the area you changed. There is no top-level `npm test`.

- Brainstorm server tests:
  ```bash
  cd tests/brainstorm-server && npm test
  ```
- Claude Code tests:
  ```bash
  cd tests/claude-code && ./run-skill-tests.sh
  ```
- OpenCode tests:
  ```bash
  cd tests/opencode && ./run-tests.sh
  ```
- Bump version:
  ```bash
  ./scripts/bump-version.sh <X.Y.Z>
  ```
- Sync to Codex plugin:
  ```bash
  ./scripts/sync-to-codex-plugin.sh [-n|--yes|--bootstrap|--local PATH]
  ```
- Render skill flowcharts (requires Graphviz):
  ```bash
  cd skills/writing-skills && ./render-graphs.js ../<skill> [--combine]
  ```

## Pull request requirements

Every PR must:

1. Solve a real problem that someone actually experienced. Speculative or theoretical fixes are not acceptable.
2. Fully complete the PR template at `.github/PULL_REQUEST_TEMPLATE.md`. No blank sections or placeholder text.
3. Search existing open **and** closed PRs for the same problem. Reference them and explain why your approach differs if a prior attempt failed.
4. Be reviewed and approved by your human partner before submission. Show them the complete diff.
5. Stay focused on one problem. Do not bundle unrelated changes.
6. Include test results from at least one harness in the environment table.

## What we will not accept

- **Third-party dependencies.** Superskills is zero-dependency by design. The only exception is adding support for a new harness.
- **"Compliance" rewrites of skills.** Do not restructure or reword skills to match Anthropic's published skill guidance without extensive eval evidence.
- **Project-specific or personal configuration.** If it only helps one project, team, or workflow, publish it as a separate plugin.
- **Bulk or spray-and-pray PRs.** Pick one issue, understand it deeply, and submit quality work.
- **Speculative or theoretical fixes.** Every change must address a specific, observed problem.
- **Domain-specific skills.** Core skills must be useful across all project types.
- **Fork-specific changes.** Do not upstream fork customizations or rebranding.
- **Fabricated content.** Invented claims or hallucinated functionality are closed immediately.
- **Bundled unrelated changes.** Split them into separate PRs.

## New harness support

A PR that adds support for a new harness must include a complete session transcript proving the integration works end-to-end.

The acceptance test is a clean session with exactly this user message:

> Let's make a react todo list

A working integration auto-triggers the `brainstorming` skill before writing any code.

These are **not** valid integrations:

- Manually copying skill files into the harness
- Wrapping with `npx skills` or similar runtime shims
- Anything that requires the user to opt in per session
- Anything where `brainstorming` does not auto-trigger

## Skill changes require evaluation

Skills are behavior-shaping code, not prose. If you modify a skill:

1. Use `superskills:writing-skills` to develop and test the change.
2. Run adversarial pressure tests across multiple sessions.
3. Show before/after eval results in your PR.
4. Do not modify carefully-tuned content without evidence the change is an improvement.

## Deployment and branch rules

- All code changes must be pushed to the `dev` branch first.
- If the `dev` branch does not exist, create it from `main` before pushing.
- Do not push directly to `main`.
- Before merging `dev` into `main`, stop and ask the user for explicit confirmation.
- Show the user the diff and wait for their approval before any `main` promotion.

## General rules

- Read `CLAUDE.md` for the full contributor guidelines.
- Use the relevant Superskills skills before any action.
- Follow existing patterns and voice. "Your human partner" is deliberate terminology.
- Keep changes minimal and focused.
- Verify before claiming success. Run the appropriate tests and include the output.
- If you are unsure whether something belongs in core, it probably does not.
