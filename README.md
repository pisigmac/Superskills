# Superskills

Superskills is a complete software-development methodology for your coding agents, built on top of a set of composable skills and a small bootstrap that makes sure your agent uses them.

It works across Claude Code, Codex CLI, Codex App, Cursor, OpenCode, Gemini CLI, GitHub Copilot CLI, and Factory Droid.

## Quickstart

Install Superskills in the harness you use:

- [Claude Code](#claude-code)
- [Codex CLI](#codex-cli)
- [Codex App](#codex-app)
- [Cursor](#cursor)
- [OpenCode](#opencode)
- [Gemini CLI](#gemini-cli)
- [GitHub Copilot CLI](#github-copilot-cli)
- [Factory Droid](#factory-droid)

## How it works

From the moment you start a session, Superskills shapes how your agent approaches work. When you say you want to build something, the agent does not jump straight into code. Instead, it steps back and refines what you are actually trying to do.

Once the design is clear, the agent shows it to you in short, reviewable chunks. After you approve, it builds a detailed implementation plan and then works through it task by task, reviewing its own work as it goes.

The core workflow is:

1. **brainstorming** — Refine the idea, explore alternatives, and produce an approved design document.
2. **using-git-worktrees** — Create an isolated workspace and verify a clean test baseline.
3. **writing-plans** — Break the work into small, concrete tasks with exact file paths, code, and verification steps.
4. **subagent-driven-development** or **executing-plans** — Implement the plan, with per-task review.
5. **test-driven-development** — Write tests first, make them pass, then refactor.
6. **requesting-code-review** — Review completed work against the plan.
7. **finishing-a-development-branch** — Verify tests and decide whether to merge, open a PR, keep, or discard the branch.

Because the skills trigger automatically, you do not have to remember them. Your agent just has Superskills.

## What's inside

### Skills library

**Testing**
- **test-driven-development** — RED-GREEN-REFACTOR cycle, including common testing anti-patterns.

**Debugging**
- **systematic-debugging** — Four-phase root-cause process.
- **verification-before-completion** — Evidence before claims.

**Collaboration**
- **brainstorming** — Socratic design refinement.
- **writing-plans** — Detailed implementation plans.
- **executing-plans** — Batch execution with human checkpoints.
- **dispatching-parallel-agents** — Concurrent subagent workflows.
- **requesting-code-review** — Pre-review checklist.
- **receiving-code-review** — Responding to feedback.
- **using-git-worktrees** — Parallel development branches.
- **finishing-a-development-branch** — Merge/PR decision workflow.
- **subagent-driven-development** — Fast iteration with two-stage review.

**Meta**
- **writing-skills** — Create and test new skills.
- **using-superskills** — Introduction to the skills system.

### Feature Factory

An opt-in implementation pipeline offered after planning, for complex full-stack features:

- **feature-factory-driven-development** — Choose standard execution or the factory pipeline.
- **feature-factory-orchestrator** — Coordinate builders, verifiers, and validators.
- **codebase-researcher** — Map the existing codebase before building.
- **backend-builder** — Implement backend tasks with tests and vulnerability checks.
- **frontend-builder** — Implement frontend tasks with tests and vulnerability checks.
- **test-verifier** — Run acceptance tests after builders complete.
- **validator** — Final audit of files, functionality, and security.
- **production-shipping-protocol** — Hard-gate checks before shipping.
- **testing-protocol** — Testing standards and gates.
- **deployment-protocol** — Deployment and rollback protocol.

## Philosophy

- **Test-Driven Development** — Write tests first, always.
- **Systematic over ad-hoc** — Process over guessing.
- **Complexity reduction** — Simplicity as the primary goal.
- **Evidence over claims** — Verify before declaring success.

Read [the original release announcement]([RELEASE_POST_URL]).

## Installation

Install Superskills separately for each harness you use.

### Claude Code

Superskills is available on the [official Claude plugin marketplace]([CLAUDE_MARKETPLACE_URL]) *(in progress)*.

**Official marketplace:**

```bash
/plugin install superskills@claude-plugins-official  # (in progress)
```

**Superskills marketplace:**

```bash
/plugin marketplace add pisigmac/Superskills-marketplace  # (in progress)
/plugin install superskills@superskills-marketplace      # (in progress)
```

### Codex CLI

Superskills is available on the [official Codex plugin marketplace](https://github.com/openai/plugins) *(in progress)*.

```bash
/plugins
```

Search for `superskills` and select `Install Plugin` *(in progress)*.

### Codex App

Superskills is available on the [official Codex plugin marketplace](https://github.com/openai/plugins) *(in progress)*.

1. In the Codex app, click **Plugins** in the sidebar.
2. Find `Superskills` in the Coding section.
3. Click the `+` next to Superskills and follow the prompts *(in progress)*.

### Cursor

In Cursor Agent chat *(in progress)*:

```text
/add-plugin superskills  # (in progress)
```

Or search for "superskills" in the plugin marketplace *(in progress)*.

### OpenCode

OpenCode uses its own plugin install; install Superskills separately even if you already use it in another harness *(in progress)*.

1. Tell OpenCode:

   ```
   Fetch and follow instructions from https://raw.githubusercontent.com/pisigmac/Superskills/refs/heads/main/.opencode/INSTALL.md
   ```

2. Detailed docs: [`docs/README.opencode.md`](docs/README.opencode.md)

### Gemini CLI

```bash
gemini extensions install https://github.com/pisigmac/Superskills  # (in progress)
```

Update later:

```bash
gemini extensions update superskills  # (in progress)
```

### GitHub Copilot CLI

```bash
copilot plugin marketplace add pisigmac/Superskills-marketplace  # (in progress)
copilot plugin install superskills@superskills-marketplace       # (in progress)
```

### Factory Droid

```bash
droid plugin marketplace add https://github.com/pisigmac/Superskills  # (in progress)
droid plugin install superskills@superskills                          # (in progress)
```

## Contributing

If you are contributing code, documentation, or skills, read [`AGENTS.md`](AGENTS.md) first. It contains the project structure, conventions, and contribution requirements for this repository.

Pull requests must follow the PR template, solve a real problem, and be reviewed by a human before submission. See `CLAUDE.md` for the full contributor guidelines.

## Updating

Updates are harness-dependent and often automatic.

## License

MIT License — see the [`LICENSE`](LICENSE) file for details.

## Community

Superskills is built by [pisigmac](https://github.com/pisigmac/Superskills).

- **Discord**: [Join us]([DISCORD_INVITE_URL]) for community support, questions, and sharing what you are building with Superskills.
- **Issues**: https://github.com/pisigmac/Superskills/issues
- **Release announcements**: [Sign up](https://github.com/pisigmac/Superskills/releases) to get notified about new versions.

## Sponsorship

If Superskills has helped you do something that makes money and you are so inclined, please consider [sponsoring my open-source work](https://github.com/sponsors/pisigmac).

Thanks!

- pisigmac

---

*This is an extended version of the Superpowers skill.*
