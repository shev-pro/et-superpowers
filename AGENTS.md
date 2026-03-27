# et-superpowers

A skills library and plugin for AI coding agents (Claude Code, Cursor, Codex, OpenCode, Gemini CLI). Provides a complete software development workflow through composable "skills" — markdown reference documents that guide agent behavior.

## Tech Stack

- **Language:** Bash (hooks), JavaScript (OpenCode plugin entry point)
- **Plugin formats:** Claude Code plugin (`.claude-plugin/`), Cursor plugin (`.cursor-plugin/`), Gemini CLI extension (`gemini-extension.json`), Codex/OpenCode via symlink/install
- **Package manager:** npm (metadata only — `package.json`)

## Project Structure

```
skills/                    # Core skills library — one directory per skill
  <skill-name>/
    SKILL.md               # Required: YAML frontmatter + skill body
    supporting-file.*      # Heavy reference files or reusable tools (100+ lines)
agents/                    # Cursor agent definitions
  code-reviewer.md
commands/                  # Deprecated slash-commands (redirect to skills)
hooks/
  session-start            # Bash script injected at every session start
  hooks.json               # Claude Code hook config (SessionStart trigger)
  hooks-cursor.json        # Cursor hook config
  run-hook.cmd             # Windows hook runner
.claude-plugin/
  plugin.json              # Claude Code plugin metadata
.cursor-plugin/
  plugin.json              # Cursor plugin metadata
.codex/
  INSTALL.md               # Codex installation instructions
.opencode/
  INSTALL.md               # OpenCode installation instructions
  plugins/superpowers.js   # OpenCode plugin entry point
docs/
  superpowers/
    specs/                 # Design documents (YYYY-MM-DD-<topic>-design.md)
    plans/                 # Implementation plans (YYYY-MM-DD-<topic>-plan.md)
  README.codex.md          # Codex-specific docs
  README.opencode.md       # OpenCode-specific docs
  testing.md               # Testing notes
tests/claude-code/         # Integration test scripts and token analysis tool
```

## Key Entry Points

- **Session bootstrap:** `hooks/session-start` — injects `skills/using-superpowers/SKILL.md` into every agent session
- **Skill discovery:** `skills/using-superpowers/SKILL.md` — teaches agents how to find and invoke all other skills
- **Cursor agent:** `agents/code-reviewer.md`
- **Claude Code plugin:** `.claude-plugin/plugin.json`
- **Cursor plugin:** `.cursor-plugin/plugin.json`
- **Gemini extension:** `gemini-extension.json`
- **OpenCode plugin:** `.opencode/plugins/superpowers.js`

## Skills

All skills live in `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`).

**Development Workflow**
- `etflow` — Pre-implementation gate: complexity triage → brainstorming → design doc → plan → hard stop
- `brainstorming` — Socratic design refinement
- `writing-plans` — Detailed implementation plans saved to `docs/plans/`
- `executing-plans` — Batch execution with human checkpoints
- `subagent-driven-development` — Per-task fresh subagents with two-stage review
- `dispatching-parallel-agents` — Concurrent subagent workflows
- `using-git-worktrees` — Isolated development branches
- `finishing-a-development-branch` — Merge/PR decision and plan archival

**Testing**
- `test-driven-development` — RED-GREEN-REFACTOR cycle
- `verification-before-completion` — Evidence-based completion gate

**Debugging**
- `systematic-debugging` — 4-phase root cause process

**Code Review**
- `code-review` — Structured review with language-specific references
- `requesting-code-review` — Pre-review checklist
- `receiving-code-review` — Responding to feedback

**Documentation**
- `codebase-documentation` — Multi-session doc generation: AGENTS.md, HLD, component docs, API, data model, rationale

**Meta**
- `using-superpowers` — Introduction to the skills system (auto-loaded each session)
- `writing-skills` — Create and test new skills

## How Skills Load

1. `hooks/session-start` runs on every new session.
2. It injects the full text of `skills/using-superpowers/SKILL.md` into the agent's context.
3. The agent then invokes the `Skill` tool (or platform equivalent) on demand to load any other skill.

Platform tool name mapping: `skills/using-superpowers/references/codex-tools.md` (Codex), `GEMINI.md` (Gemini CLI).

## Skill File Format

```yaml
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions — NOT a workflow summary]
---
```

Constraints: `name` uses letters/numbers/hyphens only; `description` starts with "Use when..."; max 1024 chars total in frontmatter.

## Development

```bash
# Run integration tests (requires `claude` CLI, 10–30 min)
cd tests/claude-code
./test-subagent-driven-development-integration.sh

# Analyze token usage
python3 tests/claude-code/analyze-token-usage.py ~/.claude/projects/<dir>/<session>.jsonl

# Check skill word count (target <150–200 words for frequently-loaded skills)
wc -w skills/<skill-name>/SKILL.md

# Install plugin locally for Claude Code testing
/plugin install superpowers@superpowers-dev

# Install for Codex (symlink)
ln -s $(pwd)/skills ~/.agents/skills/superpowers
```

## Configuration

No runtime environment variables. Platform-specific config lives in the respective plugin JSON files and hook scripts.

## Documentation Index

- [Codex Setup](docs/README.codex.md) — Codex-specific installation and usage
- [OpenCode Setup](docs/README.opencode.md) — OpenCode-specific installation and usage
- [Testing Notes](docs/testing.md) — Integration test guidance
- [Design Specs](docs/superpowers/specs/) — Design documents for past and future work
- [Implementation Plans](docs/superpowers/plans/) — Active and completed plans
