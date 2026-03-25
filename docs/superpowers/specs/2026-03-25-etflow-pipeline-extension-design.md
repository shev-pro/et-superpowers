# Design: etflow Pipeline Extension

**Date:** 2026-03-25
**Status:** Approved

## Summary

Extend the `etflow` skill to cover the full design-to-ready-to-implement pipeline for complex tasks. After the design doc is finalized, `etflow` invokes `writing-plans`, runs an automated plan review, waits for human confirmation, then hard-stops with a handoff message directing the user to open a new chat for implementation.

Additionally, standardize the plan file location across the repo to `docs/plans/`, and archive completed plans to `docs/plans/completed/` at the end of implementation.

## Target User

A developer who knows how to code but is unfamiliar with AI-assisted workflows. They understand concepts like design docs, branches, and PRs but need the AI to enforce the right sequence so they don't skip steps under pressure.

## Goals

- One skill covers the full journey from "I have an idea" to "plan is ready, go implement"
- Implementation always starts in a clean-context new chat (context hygiene, not a human gate)
- Plans live in one canonical location: `docs/plans/`
- Completed plans are archived, not deleted

## Non-Goals

- Changing the simple-task path (Phase 3A) — direct implementation stays as-is
- Making `codebase-documentation` part of the feature flow — remains independent
- Requiring `RATIONALE.md` — optional throughout, never blocks progress

## File Layout

```
docs/
├── design/
│   └── YYYY-MM-DD_<feature>.md               # design doc (etflow Phase 3B)
├── plans/
│   ├── YYYY-MM-DD_<feature>-plan.md          # implementation plan (writing-plans)
│   ├── YYYY-MM-DD_<feature>-plan-review.md   # plan reviewer output
│   └── completed/
│       ├── YYYY-MM-DD_<feature>-plan.md
│       └── YYYY-MM-DD_<feature>-plan-review.md
└── RATIONALE.md                               # optional, project-level constraint registry
```

## Changes

### 1. `skills/etflow/SKILL.md`

Replace the current Phase 3B Step 4 (hard stop) with four new steps:

**Step 4 — Invoke `writing-plans`**
Declare: *"Design is finalized. Moving to implementation planning."* Invoke the `writing-plans` skill, instructing it to save the plan to `docs/plans/YYYY-MM-DD_<feature>-plan.md`.

**Step 5 — Dispatch plan reviewer subagent**
Using `skills/writing-plans/plan-document-reviewer-prompt.md`, dispatch a reviewer subagent that checks:
- Task granularity (each task 2–5 minutes of work)
- Complete file paths and verification steps
- No task depends on context not provided in the plan
- TDD structure present

Save reviewer output to `docs/plans/YYYY-MM-DD_<feature>-plan-review.md`.
If issues found: fix and re-review. Max 2 iterations, then surface to human.

**Step 6 — Human confirmation gate**
Present:
> *"Design doc: `docs/design/YYYY-MM-DD_<feature>.md`*
> *Plan: `docs/plans/YYYY-MM-DD_<feature>-plan.md`*
> *Review: `docs/plans/YYYY-MM-DD_<feature>-plan-review.md`*
> Review both documents. Reply 'go' when ready to proceed."*

Do not proceed until explicit user confirmation.

**Step 7 — Hard stop with handoff**
After confirmation:
> *"Open a new chat. Your first message: 'Implement the plan in `docs/plans/YYYY-MM-DD_<feature>-plan.md`'."*

Session ends. No code written in this session.

**RATIONALE.md references:** Keep existing references but mark them optional — check it if it exists, propose updates if warranted, never block on it.

---

### 2. `skills/writing-plans/SKILL.md`

Change the default save path from `docs/superpowers/plans/` to `docs/plans/`.

---

### 3. `skills/finishing-a-development-branch/SKILL.md`

Add a step before the merge/PR decision:

> Move plan and review to completed:
> - `docs/plans/<plan>.md` → `docs/plans/completed/<plan>.md`
> - `docs/plans/<plan>-review.md` → `docs/plans/completed/<plan>-review.md`

The plan file path is already known at this point (read at the start of `subagent-driven-development`).

## Definition of Done

- [ ] `etflow` Phase 3B extended with Steps 4–7
- [ ] `etflow` RATIONALE.md references marked optional
- [ ] `writing-plans` saves to `docs/plans/` by default
- [ ] `finishing-a-development-branch` moves plan + review to `docs/plans/completed/` before merge decision
- [ ] Manual walkthrough: create a feature request, run through full etflow flow, verify files land in correct locations
