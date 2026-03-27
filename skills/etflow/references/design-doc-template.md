# Design Document — [Feature Name]

**Date:** YYYY-MM-DD
**Author:** [user's name — do not include the AI name]
**Status:** Draft | In Review | Approved
**Service:** [service/repo name]
**References:**
- [Doc name vX.X (Date)]
- [Link or filename]

---

## 1. Context and motivation

> Why are we doing this? What problem does it solve?
> Describe the current state and what is missing or broken.

---

## 2. Goal

> What do we want to achieve at the end of this implementation?
> Be measurable where possible (e.g. "the user can do X without having to do Y").

---

## 3. Known constraints & rationale

> Anti-patterns or deviations from best practices that are **intentional** in this project.
> Claude reads `docs/RATIONALE.md` before proposing approaches — this section captures
> any constraints specific to this feature, or confirms which global constraints apply here.
> If no constraints apply, write "None — no known deviations from best practices affect this feature."

| What it looks like | Why it exists | Do not change unless |
|--------------------|---------------|----------------------|
| [pattern that seems wrong] | [the real reason] | [precondition to revisit] |

---

## 4. Design decisions

> Key decisions made upfront. Use this table for choices that are not obvious or that
> were debated. Don't list every trivial implementation detail — only decisions with
> real trade-offs or that future readers might question.

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [What was decided] | [The chosen option] | [Why this and not something else] |

---

## 5. Proposed solution

> Describe the chosen technical approach. Be specific: which components are touched,
> which new entities are introduced, how data flows, etc.

### 5.1 Changes to existing components

- **[Component/File]**: [what changes and why]

### 5.2 New components / entities

- **[Name]**: [description and responsibilities]

### 5.3 Main flow

> Describe the end-to-end flow, in pseudocode or step list.

---

## 6. Architecture diagram

> ASCII or Mermaid diagram showing the main components and their interactions.
> Include queues, DBs, external APIs, CronJobs — anything that moves data.
> Delete this section if the solution is simple enough that the flow description above suffices.

```
[diagram here]
```

---

## 7. File summary

> Complete list of files touched by this implementation. Useful for scoping code review
> and estimating effort.

| File | Action | Description |
|------|--------|-------------|
| `path/to/file.py` | New / Edit | [what and why] |

---

## 8. Alternatives considered

> Did you evaluate other approaches? Why were they discarded?
> (If no alternatives were evaluated, write "N/A" and explain why the solution was obvious.)

| Alternative | Pros | Cons | Reason discarded |
|---|---|---|---|
| ... | ... | ... | ... |

---

## 9. Impacts and risks

> What could go wrong? Are there critical dependencies, regression risks,
> impacts on performance, security, or UX?

- **Risk 1**: [description] → [mitigation]
- **Risk 2**: ...

---

## 10. Implementation phases

> Break the work into sequential phases when the full implementation can't or shouldn't
> land in one shot. For simpler features, a single phase is fine — just remove the others.

### Phase 1 — [Name] (current)

**Scope**: [what is in, what is explicitly out]

**Files**:

| File | Action | Description |
|------|--------|-------------|
| `...` | New/Edit | ... |

**Verification**:
1. [How to verify this phase works]
2. ...

### Phase 2 — [Name] (future)

- [Bullet list of what's deferred]

---

## 11. Definition of done

> How do we know this feature is "done"? What must be true for us to consider it complete?

- [ ] Criterion 1
- [ ] Criterion 2

---

## Additional notes

> Any other relevant information: links to tickets, decisions made in calls,
> open questions not yet resolved, known limitations.
