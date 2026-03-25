# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Superpowers is a skills library and plugin for AI coding agents (Claude Code, Cursor, Codex, OpenCode, Gemini CLI). It provides a complete software development workflow through composable "skills" — markdown reference documents that guide agent behavior.

## Commands

### Integration Tests

Tests run real Claude Code sessions in headless mode and take 10–30 minutes:

```bash
# Run the main integration test
cd tests/claude-code
./test-subagent-driven-development-integration.sh
```

**Requirements:**
- Claude Code installed and available as `claude`
- Running FROM the superpowers directory (skills only load from here)
- Local dev marketplace enabled: `"superpowers@superpowers-dev": true` in `~/.claude/settings.json`

### Token Analysis

```bash
python3 tests/claude-code/analyze-token-usage.py ~/.claude/projects/<project-dir>/<session-id>.jsonl
```

### Skill Word Count

```bash
wc -w skills/<skill-name>/SKILL.md
# getting-started/frequently-loaded skills: target <150-200 words
```

### Plugin Installation (for local dev testing)

```bash
# Claude Code marketplace
/plugin install superpowers@superpowers-dev

# Codex: symlink skills directory
ln -s $(pwd)/skills ~/.agents/skills/superpowers
```

## Architecture

### Directory Structure

```
skills/                    # Core skills library — each skill in its own directory
  skill-name/
    SKILL.md               # Required: YAML frontmatter + skill content
    supporting-file.*      # Only for heavy reference (100+ lines) or reusable tools
hooks/
  session-start            # Bash script: injects using-superpowers into every session
  run-hook.cmd             # Cross-platform hook runner
  hooks.json               # Hook config for Claude Code (SessionStart trigger)
  hooks-cursor.json        # Hook config for Cursor
.claude-plugin/
  plugin.json              # Claude Code plugin metadata
.cursor-plugin/
  plugin.json              # Cursor plugin metadata
gemini-extension.json      # Gemini CLI extension config
docs/superpowers/
  specs/                   # Design documents (YYYY-MM-DD-<topic>-design.md)
  plans/                   # Implementation plans (YYYY-MM-DD-<topic>-plan.md)
tests/claude-code/         # Integration test scripts + token analysis tool
```

### How Skills Load

The `hooks/session-start` script runs on every new session (via `hooks.json`) and injects the full content of `skills/using-superpowers/SKILL.md` into context. This bootstraps the agent into knowing how to find and invoke all other skills via the `Skill` tool.

All other skills load on-demand when the agent calls the `Skill` tool.

### Skill File Format

Every `SKILL.md` requires YAML frontmatter with exactly two fields:

```yaml
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions and symptoms — NOT a workflow summary]
---
```

**Critical constraints:**
- `name`: letters, numbers, hyphens only (no special characters)
- `description`: must start with "Use when...", must NOT summarize the skill's workflow (doing so causes agents to follow the description instead of reading the skill body)
- Max 1024 characters total in frontmatter
- Description written in third person

### The Development Workflow Skills Encode

Skills implement this mandatory workflow:
1. **brainstorming** → design doc in `docs/superpowers/specs/`
2. **using-git-worktrees** → isolated branch
3. **writing-plans** → implementation plan in `docs/superpowers/plans/`
4. **subagent-driven-development** → fresh subagent per task, two-stage review (spec compliance then code quality)
5. **test-driven-development** → RED-GREEN-REFACTOR enforced inside each subagent
6. **finishing-a-development-branch** → merge/PR decision

### Platform Support

The same skills directory serves multiple platforms. Tool name differences are handled via:
- `skills/using-superpowers/references/codex-tools.md` — Codex tool equivalents
- `GEMINI.md` — Gemini CLI tool mapping (loaded automatically)

## Writing and Testing Skills

New skills follow TDD: run a pressure scenario WITHOUT the skill first (RED), then write the minimal skill to address observed failures (GREEN), then close loopholes (REFACTOR). See `skills/writing-skills/SKILL.md` for the full guide.

**Personal skills** go in `~/.claude/skills/` (Claude Code) or `~/.agents/skills/` (Codex), not in this repo.
