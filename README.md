# et-superpowers

A fork of [Superpowers](https://github.com/obra/superpowers) — a complete software development workflow for coding agents, built on composable "skills". This fork extends the original with additional skills focused on guiding developers who are new to AI-assisted workflows.

## How it works

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it *doesn't* just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do.

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest.

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY.

Next up, once you say "go", it launches a *subagent-driven-development* process in a fresh chat, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers.


## Installation

**Note:** Installation differs by platform. Claude Code and Cursor use the plugin system. Codex and OpenCode require manual setup.

### Claude Code (via Plugin Marketplace)

In Claude Code, register the marketplace first:

```bash
/plugin marketplace add shev-pro/et-superpowers
```

Then install:

```bash
/plugin install superpowers@superpowers
```

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/shev-pro/et-superpowers/refs/heads/main/.codex/INSTALL.md
```

**Detailed docs:** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/shev-pro/et-superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

### Gemini CLI

```bash
gemini extensions install https://github.com/shev-pro/et-superpowers
```

To update:

```bash
gemini extensions update superpowers
```

### Verify Installation

Start a new session in your chosen platform and ask for something that should trigger a skill (for example, "help me plan this feature" or "let's debug this issue"). The agent should automatically invoke the relevant skill.

## The Basic Workflow

1. **etflow** - Activates whenever you ask to build something. Assesses complexity, runs brainstorming for complex tasks, produces a design doc, then an implementation plan saved to `docs/plans/`. Hard stops before any code — implementation always starts in a new chat.

2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activated by etflow. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps. Saves to `docs/plans/`.

4. **subagent-driven-development** or **executing-plans** - Activates in the new implementation chat. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, archives plan to `docs/plans/completed/`, presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Development Workflow**
- **etflow** - Mandatory pre-implementation gate: complexity triage, brainstorming, design doc, implementation plan, reviewer, hard stop before code
- **brainstorming** - Socratic design refinement with visual companion
- **writing-plans** - Detailed implementation plans saved to `docs/plans/`
- **executing-plans** - Batch execution with checkpoints
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow, archives completed plans

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Code Review**
- **code-review** - Structured review with language-specific references (Go, Python, TypeScript, Kotlin, Java)
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback

**Documentation**
- **codebase-documentation** - Multi-session structured documentation generation: AGENTS.md, HLD, component docs, API, data model, rationale

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success
- **Design before code** - No implementation without an approved plan

## Contributing

Skills live directly in this repository. To contribute:

1. Fork the repository
2. Create a branch for your skill
3. Follow the `writing-skills` skill for creating and testing new skills
4. Submit a PR

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

Skills update automatically when you update the plugin:

```bash
/plugin update superpowers
```

## License

MIT License - see LICENSE file for details

## Issues

- **Issues**: https://github.com/shev-pro/et-superpowers/issues
