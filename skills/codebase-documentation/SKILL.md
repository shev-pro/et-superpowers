---
name: codebase-documentation
description: Generate structured documentation for existing codebases. Creates an AGENTS.md entry point linking to deeper docs/ documents (HLD, component docs, API docs). Use when the user asks to document a codebase, create project documentation, write architecture docs, or generate AGENTS.md. Handles monorepos by asking about documentation placement strategy.
---

# Codebase Documentation

Generate clear, concise documentation for existing codebases. Docs serve **both humans and AI agents** as navigation aids — not prose novels.

## Principles

- **Code is the source of truth.** Docs point to it, never duplicate it.
- **Concise over comprehensive.** Each doc earns its existence.
- **Pointers over explanations.** Link to files/functions, don't re-describe them.
- **Mermaid diagrams** for architecture, data flow, and sequences where they add clarity.
- **One level of depth per document.** AGENTS.md → topic docs → (code).

## Document Hierarchy

```
AGENTS.md                              # Entry point — project overview + doc index
docs/
├── high-level-design.md               # Architecture, key decisions, component map
├── <component-name>/                  # Per-component/module folder
│   ├── README.md                      # Component deep dive
│   ├── api.md                         # Component-specific API (if applicable)
│   └── data-model.md                  # Component-specific data model (if applicable)
├── api.md                             # Global API surface (if applicable)
├── data-model.md                      # Global data structures, DB schema (if applicable)
├── codestyle.md                       # Naming, comments, idioms (if applicable)
├── rationale.md                       # Non-obvious decisions with reasoning (if applicable)
└── deployment.md                      # Build, deploy, infra (if applicable)
```

Not every project needs all docs. Pick what matters. Propose a doc list to the user.

## Workflow

**This is a multi-session process.** Each phase is completed in a single conversation. Between sessions, `docs/PLAN.md` tracks progress. Never try to do everything in one go.

### Session Entry Point (MANDATORY — every invocation starts here)

1. **Look for `docs/PLAN.md`.**
2. **If it exists:** read it, find the next `[pending]` phase, show the user where we left off, and **ask for permission to start that phase**.
3. **If it does not exist:** this is a fresh project — begin with Phase 1 below.

---

### Phase 1: Assess & Explore the Codebase

Before writing anything, understand the project scope:

1. Read top-level files: `go.mod`, `package.json`, `Cargo.toml`, `Makefile`, `Dockerfile`, etc.
2. Map the directory structure (depth 4-5).
3. Identify entry points, main packages, and key abstractions.
4. Check for existing docs, README, or AGENTS.md.
5. Detect code style tooling: linter configs (`.eslintrc`, `golangci-lint.yml`, `.rubocop.yml`), formatters (`.prettierrc`, `rustfmt.toml`, `.editorconfig`), and style guides.
6. If a monorepo/workspace is detected → go to **Monorepo Decision** below.

**Size assessment.** After initial exploration, estimate the project size:
- **Small:** <10 packages/modules, single service.
- **Medium:** 10-30 packages, a few distinct components.
- **Large:** 30+ packages, multiple services/domains, or monorepo.

If the size is ambiguous, **ask the user** how big they consider the codebase and whether there are areas they want prioritized or excluded.

**Phase 1 closes** by presenting the exploration summary to the user and asking: *"Does this look accurate? Anything I missed or got wrong?"* Only proceed after user confirms.

### Phase 2: Propose Document Structure

Present 2-3 options to the user for how to organize docs. Example:

**Option A — Minimal** (small projects):
- `AGENTS.md` + `docs/high-level-design.md` only.
- Pros: low maintenance. Cons: less navigable for newcomers.

**Option B — Component-split** (medium projects):
- `AGENTS.md` + HLD + one folder per major component/module.
- Pros: good balance, each component is self-contained. Cons: more files to maintain.

**Option C — Full suite** (large/complex projects):
- `AGENTS.md` + HLD + component folders (with component-specific API/data model) + global API + data model + deployment.
- Pros: thorough. Cons: higher maintenance cost.

State your recommendation with a short reason, then let user decide.

**Phase 2 closes** after the user picks a structure.

### Phase 3: Create the Documentation Plan

Once the user picks a structure, **generate a documentation plan** and save it to `docs/PLAN.md`. Each step in the plan becomes a separate phase to execute.

The plan must contain:

1. **Ordered list of phases** — each phase is one document to create, with a concrete file path.
2. **For each phase:** which code areas to read, what the doc should cover, estimated complexity (simple/medium/complex).
3. **Dependencies between phases** — e.g. HLD should be written before component docs since component docs reference it.
4. **Open questions** — things to ask the user before or during writing.

Example plan format:

```markdown
# Documentation Plan

> Generated by codebase-documentation skill. Track progress here.
> Each phase = one conversation session. Resume by re-invoking the skill.

## Scope
- Structure: Option B (Component-split)
- Components identified: auth, billing, notifications

## Phases

### Phase 4. `docs/high-level-design.md` — [pending]
- **Read:** top-level structure, entry points, config
- **Covers:** architecture overview, component map, key decisions
- **Complexity:** medium
- **Depends on:** nothing

### Phase 5. `docs/auth/README.md` — [pending]
- **Read:** `src/auth/`, `src/middleware/auth.go`
- **Covers:** auth flow, token handling, middleware chain
- **Complexity:** complex
- **Depends on:** phase 4

### Phase 6. `docs/billing/README.md` — [pending]
- **Read:** `src/billing/`, `src/models/invoice.go`
- **Covers:** payment processing, invoice lifecycle
- **Complexity:** medium
- **Depends on:** phase 4

...

### Phase N. `AGENTS.md` — [pending]
- **Covers:** entry-point index referencing all other docs
- **Depends on:** all previous phases
- **Note:** always the last phase

## Open Questions
- [ ] What payment provider is used? (not obvious from code)
- [ ] Is the notification service being replaced?
```

**Phase 3 closes** by showing the plan to the user and asking: *"Plan looks good? Should I start with Phase 4 next time, or adjust anything?"*

### Phase 4–N: Execute One Phase Per Session

**Each session completes exactly one phase from `docs/PLAN.md`:**

1. **Read `docs/PLAN.md`** and pick the next `[pending]` phase respecting dependency order.
2. **Tell the user** which phase you're about to work on and ask for confirmation to proceed.
3. **Read the relevant code** thoroughly for that phase.
4. **Draft the document** using the templates in [templates.md](templates.md).
5. **Include Mermaid diagrams** for architecture and flows.
6. **Add file/function pointers** — use relative paths from repo root.
7. **Keep each doc under 300 lines.** Split if longer.
8. **Ask the user** if something is ambiguous — don't guess about intent or design rationale.
9. **Flag non-obvious patterns** encountered during this phase (see Rationale Discovery below).

**Phase close (MANDATORY):**

After completing the document, **always** do the following before ending:

1. **Update `docs/PLAN.md`** — mark the phase as `[done]`.
2. **Present the result** to the user with a brief summary of what was created.
3. **Ask explicitly:** *"Are you happy with this phase? Anything to change before we close it?"*
4. **Only after user confirms**, the phase is closed. If the user wants changes, apply them and ask again.
5. **Show what's next:** *"Next session: Phase X — `<file>`". Ready when you are."*

**Do NOT start the next phase in the same session.** Each phase is one iteration.

`AGENTS.md` is always the final phase — it references all other docs and serves as the entry-point index.

### Writing Guidelines

- Start each doc with a 1-2 sentence summary of what it covers.
- Use tables for structured info (config, env vars, endpoints).
- Use code references like `src/handler/auth.go:HandleLogin()` to point to implementation.
- Mermaid diagrams: use `graph TD` for architecture, `sequenceDiagram` for flows.
- No filler. No "This document describes...". Jump to content.
- If a section would just restate the code, link to the file instead.

### Flagging Non-Obvious Patterns (Rationale Discovery)

While exploring and documenting the codebase, **actively watch for code that looks unusual, non-standard, or contrary to best practices**. When you spot something, don't assume it's a mistake — ask the user by proposing 2-3 hypotheses for why it exists:

> I noticed `<what you found>`. This is unusual because `<why it stands out>`. Possible reasons:
>
> **A)** <hypothesis 1, e.g. "Legacy constraint from an earlier architecture">
> **B)** <hypothesis 2, e.g. "Performance optimization for a specific workload">
> **C)** <hypothesis 3, e.g. "Third-party API limitation forcing this approach">
>
> Which is closest, or is there a different reason?

Capture each confirmed rationale in `docs/rationale.md` using the template from [templates.md](templates.md). Build this document incrementally as you work through the documentation plan — don't treat it as a separate step.

Examples of things worth flagging:
- Unconventional project structure or module boundaries.
- Manual implementations where a standard library/framework solution exists.
- Unusual error handling, retry, or caching strategies.
- Dependencies that seem redundant or outdated.
- Patterns that contradict the project's own codestyle elsewhere.

### Proactive Questions

When writing, **ask the user** about:

- Deployment topology if not evident from config.
- External dependencies or integrations not visible in code.
- Planned changes that should be noted.

Propose 2-3 options with short descriptions when asking. State your suggestion.

## Monorepo Decision

When the project is a monorepo/workspace, ask the user:

**Option 1 — Per-service docs:**
Each service gets its own `AGENTS.md` + `docs/` folder.
- Pros: self-contained, easy to navigate per service, scales well.
- Cons: cross-service relationships harder to document, some duplication.

**Option 2 — Single root docs:**
One `AGENTS.md` at root, single `docs/` folder covering everything.
- Pros: unified view, easier to show cross-service flows.
- Cons: can become unwieldy, harder to maintain per-team.

**Option 3 — Hybrid:**
Root `AGENTS.md` with high-level overview + cross-cutting docs. Each service has its own `docs/` for service-specific details.
- Pros: best of both worlds. Cons: needs clear ownership rules.

**Recommendation:** Option 3 (Hybrid) for most monorepos. State this and let user decide.

## Updating Existing Docs

When docs already exist:

1. Read existing docs first.
2. Identify gaps, stale sections, or missing components.
3. Propose updates as a checklist to the user.
4. Preserve existing structure unless user agrees to restructure.

## Templates

Ready-to-fill templates for each document type are in [templates.md](templates.md). Read that file when generating documents.
