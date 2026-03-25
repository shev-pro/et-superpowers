# Codebase Documentation Skill

A Claude Code skill that generates structured, navigable documentation for any codebase. Designed to serve both humans and AI agents.

## The Problem

Codebases grow faster than their documentation. When docs exist, they're often outdated, duplicative, or disconnected from the actual code. AI agents navigating a project need a reliable entry point, not a wall of prose.

## The Approach

This skill treats documentation as a **multi-session, iterative process** — not a one-shot dump. Each session completes one focused phase, with a persistent plan (`docs/PLAN.md`) tracking progress across conversations.

### Core Principles

- **Code is the source of truth.** Docs point to it, never duplicate it.
- **Concise over comprehensive.** Every doc earns its existence.
- **Pointers over explanations.** Link to files and functions instead of re-describing them.
- **One level of depth per document.** `AGENTS.md` → topic docs → code.

## Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    SESSION ENTRY POINT                          │
│                                                                 │
│   Does docs/PLAN.md exist?                                     │
│     ├── YES → Read it, find next [pending] phase, resume       │
│     └── NO  → Fresh project, start Phase 1                     │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Assess & Explore
  │  Read project files, map structure, identify entry points,
  │  estimate size (small / medium / large).
  │  Close: present summary → user confirms accuracy.
  ▼
Phase 2: Propose Document Structure
  │  Present 2-3 options (Minimal / Component-split / Full suite).
  │  State recommendation, let user decide.
  ▼
Phase 3: Create Documentation Plan
  │  Generate docs/PLAN.md with ordered phases, one doc per phase,
  │  dependencies, and open questions.
  │  Close: user approves the plan.
  ▼
Phase 4–N: Execute One Phase Per Session
  │  Each session:
  │    1. Read PLAN.md → pick next [pending] phase
  │    2. Confirm with user → read relevant code
  │    3. Write the document using templates
  │    4. Flag non-obvious patterns (rationale discovery)
  │    5. Update PLAN.md → mark phase [done]
  │    6. Show what's next
  ▼
Final Phase: AGENTS.md
    Always last — references all generated docs as the entry-point index.
```

### Key Design Choices

**One phase per session.** Documentation benefits from iteration. Each session focuses on one document, ensuring thorough code reading and user review before moving on.

**Rationale discovery.** While exploring code, the skill actively flags unusual patterns and proposes hypotheses to the user, building a `docs/rationale.md` incrementally.

**Monorepo-aware.** Detects workspaces and offers three placement strategies: per-service docs, single root docs, or a hybrid approach.

## Document Hierarchy

```
AGENTS.md                        # Entry point — project overview + doc index
docs/
├── PLAN.md                      # Tracks documentation progress across sessions
├── high-level-design.md         # Architecture, key decisions, component map
├── <component-name>/            # Per-component folder
│   ├── README.md                # Component deep dive
│   ├── api.md                   # Component-specific API
│   └── data-model.md            # Component-specific data model
├── api.md                       # Global API surface
├── data-model.md                # Data structures, DB schema
├── codestyle.md                 # Naming, idioms (only what tooling doesn't enforce)
├── rationale.md                 # Non-obvious decisions with reasoning
└── deployment.md                # Build, deploy, infrastructure
```

Not every project needs all docs — the skill proposes only what matters.

## Files

- **[SKILL.md](SKILL.md)** — Full skill definition with workflow, guidelines, and rules.
- **[templates.md](templates.md)** — Ready-to-fill templates for each document type (AGENTS.md, HLD, component docs, API, data model, code style, rationale, deployment).

## Usage

1. Copy this skill into your Claude Code skills directory.
2. Invoke it by asking Claude to document your codebase.
3. The skill will guide you through the phases, one session at a time.
