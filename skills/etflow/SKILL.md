---
name: etflow
description: >
  Governs the development process in Claude Code and Cursor by enforcing a mandatory
  analysis phase before writing any code. Use this skill every time the user asks to
  develop, implement, add, create, modify, or refactor something — even if the request
  seems trivial. Never skip the evaluation phase.
  Typical triggers: "implement X", "add Y", "do Z", "can you...", "I need...",
  "create a...", "modify the...", "refactor...", "I want...".
---

# etflow — Engineering & Task Flow

This skill defines the mandatory process Claude must follow **every time** a user requests
the development of a feature, modification, non-trivial fix, or new component.

---

## Phase 1 — Requirement intake

Receive the user's request and make sure you understand it well.

- If the requirement is vague, ask **at most 2 questions** to clarify the scope.
- Don't ask unnecessary questions: if the context is sufficient, move directly to evaluation.
- Briefly restate the requirement in one sentence to confirm understanding.

---

## Phase 2 — Complexity assessment

Subjectively assess whether the request is **simple**, **complex**, or **very complex**:

### Indicators of a SIMPLE task (proceed directly to implementation):
- Change is localized to 1-2 existing files with no significant side effects
- No new entities, DB tables, external APIs, or modules introduced
- The solution is obvious and unambiguous, requires no architectural trade-offs
- Estimable in a few minutes of work
- Examples: typo fix, adding a field to a form, color change, small utility function

### Indicators of a COMPLEX task (requires one design document):
- Touches multiple modules, services, or layers of the application
- Introduces new entities, endpoints, tables, dependencies, or integrations
- Multiple possible approaches exist with non-trivial trade-offs
- Has impacts on security, performance, scalability, or UX
- Requires coordination with other components or teams
- Estimable in hours or days of work

### Indicators of a VERY COMPLEX task (may require HLD + multiple LLDs):
- Spans multiple independent subsystems or services
- Could realistically be broken into separate, committable chunks of work
- Different parts could be designed or implemented by different people
- Estimated in days or weeks of work across multiple implementation sessions

**When in doubt, treat the task as COMPLEX.** An extra document is always better
than a rushed implementation without proper thinking.

---

## Phase 3A — SIMPLE task: direct implementation

If the task is simple:

1. If `docs/RATIONALE.md` exists, scan it for constraints that touch the files or
   patterns you're about to change. If something relevant is found, flag it to the
   user before writing any code. If it does not exist, skip this check.
2. State explicitly: *"This is a simple task, proceeding directly."*
3. Implement the solution.
4. Briefly explain what you did and why.

---

## Phase 3B — COMPLEX task: brainstorming → design document

If the task is complex, **never fill in the template directly**. Instead, use the
collaborative brainstorming process described in `references/brainstorming.md` to
arrive at the design through dialogue. The final document is the outcome of this
process, not the starting point.

### Step 0 — Read the project rationale (optional)

Check if `docs/RATIONALE.md` exists in the repository.

- If it exists: read it in full. Internalize every entry — these are intentional
  deviations from best practices that must not be "fixed". Do not propose approaches
  that contradict an existing rationale entry without explicitly flagging the conflict.
- If it does not exist: skip this step entirely.

### Step 1 — Guided brainstorming

Follow the process in `references/brainstorming.md`:

- Explore the project context (files, docs, recent commits)
- Ask **one question at a time** to refine the idea (prefer multiple choice)
- **Ask the user about known constraints**: *"Are there any anti-patterns or non-obvious
  decisions already in place that I should be aware of before proposing an approach?
  For example: deliberate deviations from best practices, API limitations, team agreements,
  or legacy constraints that must be respected."* — collect these before proposing approaches
- Propose **2-3 approaches** with trade-offs, recommend the preferred one with reasoning,
  ensuring all proposals respect known constraints
- Present the design **in sections of 200-300 words**, asking for confirmation after each
- Apply YAGNI: cut everything that is not strictly necessary

### Step 1b — Propose document structure (VERY COMPLEX only)

If during brainstorming the task shows signs of being very complex, **before writing anything**,
propose splitting the design into a High-Level Design (HLD) + one or more Low-Level Design (LLD)
documents:

1. Present the proposed split to the user:
   - **HLD** (`YYYY-MM-DD_<feature>-hld.md`): architecture, key decisions, component overview,
     data flow, phasing — the "what and why"
   - **LLD-1** (`YYYY-MM-DD_<feature>-lld-<subsystem>.md`): detailed design for subsystem A
   - **LLD-2** (`YYYY-MM-DD_<feature>-lld-<subsystem>.md`): detailed design for subsystem B

2. Ask: *"Does this breakdown make sense? Should I produce all documents now, or would
   you prefer to review and commit the HLD first, then tackle each LLD separately?"*

3. Wait for explicit approval before proceeding. The user decides both the split **and** the
   session strategy (all at once vs. one at a time with commits in between).

4. If the user chooses **one at a time**: produce only the HLD now, stop, and wait.
   Each subsequent LLD becomes its own etflow session starting from Phase 3B Step 2.

### Step 2 — Writing the document(s)

Once the design (or the current document in scope) has been validated section by section:

1. Declare: *"The design is finalized. Writing the document now in `docs/design/`."*
2. Read `references/design-doc-template.md` and generate the document filling every relevant
   section. Pay special attention to:
   - **Section 3 — Known constraints & rationale**: populate with any constraints collected
     during brainstorming that are specific to this feature, plus references to relevant
     entries in `docs/RATIONALE.md`
   - **References** — external docs, ICDs, specs, API docs consulted
   - **Design decisions** — key choices with rationale (not every detail, only real trade-offs)
   - **Proposed solution** — components, changes, main flow
   - **Architecture diagram** — ASCII/Mermaid if the solution involves multiple moving parts
   - **File summary** — complete list of files touched (New/Edit)
   - **Alternatives considered** — approaches evaluated and discarded
   - **Impacts and risks** — what could go wrong and mitigations
   - **Implementation phases** — phase 1 now, future phases deferred (or single phase if simple)
   - **Definition of done** — explicit checklist
3. Omit sections that genuinely don't apply. Never leave placeholder text or empty sections.
4. Save to `docs/design/YYYY-MM-DD_<feature-name-kebab-case>.md`.
   For HLD/LLD splits, use the naming convention agreed in Step 1b.

### Step 3 — Update docs/RATIONALE.md (optional)

If `docs/RATIONALE.md` exists and new constraints emerged from this session, propose
adding entries to it. Wait for confirmation before writing. If RATIONALE.md does not
exist, skip this step entirely — do not create it unless the user asks.

### Step 4 — Write the implementation plan

Declare: *"Design is finalized. Moving to implementation planning."*

Invoke the `writing-plans` skill. Instruct it to save the plan to:
`docs/plans/YYYY-MM-DD_<feature-name>-plan.md`

### Step 5 — Review the plan

Dispatch a plan reviewer subagent using `references/plan-document-reviewer-prompt.md`
(from the `writing-plans` skill). Provide the plan path and design doc path.

Save the reviewer output to: `docs/plans/YYYY-MM-DD_<feature-name>-plan-review.md`

If issues found: fix and re-dispatch. Max 2 iterations, then surface to human.

### Step 6 — Human confirmation gate

Present to the user:

> *"Design doc: `docs/design/YYYY-MM-DD_<feature-name>.md`*
> *Plan: `docs/plans/YYYY-MM-DD_<feature-name>-plan.md`*
> *Review: `docs/plans/YYYY-MM-DD_<feature-name>-plan-review.md`*
>
> Review both documents. Reply 'go' when ready."*

Do not proceed until explicit user confirmation.

### Step 7 — Stop with handoff

After confirmation, tell the user:

> *"Open a new chat. Your first message:*
> *'Implement the plan in `docs/plans/YYYY-MM-DD_<feature-name>-plan.md`'"*

**Stop. Do not write any code in this session.**

---

## Behavioral rules

- **Never skip Phase 2**, even if the user says "it's quick" or "just do...".
  Always assess the real complexity yourself.
- **Never start writing code** for complex tasks before the design document has been
  completed and approved by the user.
- **If RATIONALE.md exists, read it before proposing approaches** — never suggest
  something that contradicts a documented constraint without explicitly flagging the conflict.
  If it does not exist, skip it and move on.
- **Always propose the HLD/LLD split before writing** — never decide unilaterally to
  produce multiple documents without user approval.
- If the user insists on skipping the design document, briefly explain the value of
  the process and ask for explicit confirmation before making an exception.
- The design document must not be a bureaucratic exercise: it should be useful,
  concise, and decision-oriented. Avoid empty sections or boilerplate.

---

## References

- Brainstorming process → `references/brainstorming.md` *(read when starting Phase 3B)*
- Design document template → `references/design-doc-template.md` *(read when writing the final document)*
- Project rationale template → `references/rationale-template.md` *(use when RATIONALE.md does not exist yet)*
