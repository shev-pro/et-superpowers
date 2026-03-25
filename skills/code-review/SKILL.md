---
name: code-review
description: >
  Perform structured code reviews on MR/PR diffs, individual files, or code snippets.
  Use this skill whenever the user asks to review code, check a pull request, audit
  an implementation, validate a merge request, spot bugs, assess code quality, or
  evaluate any piece of code before merging or shipping. Trigger even on soft phrases
  like "can you check this?", "does this look right?", "any issues here?", or when
  the user pastes a diff or links a GitLab/GitHub MR. Covers Python/FastAPI,
  Java/Spring, TypeScript/Node.js, Go, and Kotlin, but applies to any language.
---

# Code Review Skill

> **CRITICAL: Read-Only**
> You are a READ-ONLY reviewer. You MUST NOT modify any code.
> Only read, search, and generate reports.

Produce structured, actionable MR/PR reviews. Output is always a **scored report**
with prioritized findings — not a stream of comments.

---

## Input formats accepted

| Format | How to handle |
|---|---|
| Raw diff / patch | Analyze changed lines only; note context lines for coherence |
| File(s) pasted in chat | Full-file review |
| GitLab/GitHub MR URL | Ask user to paste diff or use MCP tool if available |
| Natural language description | Ask for code before proceeding |

If the user gives a URL without a diff, ask: *"Puoi incollare il diff o i file modificati?"*

---

## Review phases — run in order

### Phase 0 — Context snapshot (30 sec, mental)
Before writing anything, answer internally:
- What is this change trying to do?
- What branch type? (feature / fix / refactor / chore / hotfix)
- What is the blast radius if it's wrong?
- Are tests included?

Use this to calibrate depth and tone. Hotfixes and security changes get deeper scrutiny.

### Phase 1 — Architecture & design
- Does the approach make sense for the stated goal?
- Is it consistent with existing patterns in the codebase (if visible)?
- Is logic placed in the right layer? (e.g. no business logic in controllers, no DB calls in views)
- YAGNI: is anything over-engineered for the current requirement?

### Phase 2 — Code quality
- **Naming**: variables (descriptive), functions (verb-based), classes (noun, SRP), constants (UPPER_CASE)
- **Function size**: flag anything > 50 lines that doesn't have a clear reason
- **DRY**: duplicated logic that should be extracted
- **Dead code / commented-out code**: flag for removal
- **Magic numbers/strings**: should be named constants
- **Nesting depth**: > 3 levels → suggest early returns or extraction
- **Error handling**: all error paths covered, no silent failures, meaningful messages

### Phase 3 — Security
Check for:
- Unsanitized user input flowing into queries, shell commands, or HTML
- Hardcoded secrets, tokens, passwords (even in comments)
- Missing auth/authz checks on new endpoints
- Insecure deserialization
- Dependency additions (flag any new package for quick justification)

Stack-specific patterns to watch → see `references/security-patterns.md`

### Phase 4 — Performance
- N+1 queries (especially in loops over ORM objects)
- Missing pagination on list endpoints
- Unbounded memory allocation
- Synchronous blocking calls in async contexts
- Missing DB indexes for new query patterns

### Phase 5 — Testing
- New code paths without test coverage → flag specifically (not just "add tests")
- Test quality: are assertions meaningful? Are edge cases covered?
- Non-deterministic tests (time-dependent, order-dependent)
- Missing error-path tests

### Phase 6 — Documentation
- Public API changes without updated docstrings/OpenAPI annotations
- Complex logic without inline explanation
- README/CHANGELOG updates for breaking changes

---

## Output format

Always produce a report in this structure:

```
## Code Review — [short title or MR description]

### Summary
[2-3 sentences: what the change does, overall assessment, merge readiness]

### Score
| Dimension | Rating |
|---|---|
| Design | 🟢 / 🟡 / 🔴 |
| Code Quality | 🟢 / 🟡 / 🔴 |
| Security | 🟢 / 🟡 / 🔴 |
| Performance | 🟢 / 🟡 / 🔴 |
| Testing | 🟢 / 🟡 / 🔴 |

**Verdict**: ✅ Approve / 🔁 Approve with minor fixes / ❌ Changes required

### Findings

#### 🔴 Critical — must fix before merge
[finding or "None"]

#### 🟡 Important — fix soon
[finding or "None"]

#### 🟢 Nice-to-have
[finding or "None"]

### What's done well
[1-3 specific positives — always include this section]
```

---

## Finding format

Each finding must follow this template:

```
**[FILE:LINE or COMPONENT]** — [One-line title]

[What the problem is and why it matters — 1-3 sentences]

[Concrete suggestion or code example if applicable]
```

Bad: *"Performance issues here."*
Good:
```
**src/users/repository.py:45** — N+1 query in user list

`get_users()` fetches each user's roles in a separate query inside a loop.
For 100 users this generates 101 queries.

Use `.prefetch_related('roles')` (Django) or a JOIN in the base query.
```

---

## Severity guide

| Level | When to use |
|---|---|
| 🔴 Critical | Security vulnerability, data loss risk, crash/panic, broken contract |
| 🟡 Important | Performance regression, maintainability blocker, missing test for critical path |
| 🟢 Nice-to-have | Style, naming preference, minor refactor opportunity |

When in doubt, downgrade. Over-flagging trains authors to ignore reviews.

---

## Language-specific reference files

After determining the language(s) in the diff, load the corresponding reference file(s)
**before** writing any findings. Each file covers: code quality patterns, security
anti-patterns, performance, and testing gotchas specific to that stack.

| Language / Framework | Load |
|---|---|
| Python, FastAPI | `references/python.md` |
| Java, Spring | `references/java.md` |
| Kotlin, Spring (Kotlin) | `references/kotlin.md` |
| TypeScript, Node.js, React | `references/typescript.md` |
| Go | `references/go.md` |
| Cross-stack / security deep-dive | `references/security-patterns.md` |
| Maintainability, technical debt, design principles | `references/maintainability.md` |
| Documentation accuracy, stale comments, JSDoc drift, API doc gaps | `references/documentation.md` |

If the diff spans multiple languages (e.g. a Kotlin backend + TypeScript frontend),
load all relevant files. For security-focused reviews on any stack, also load
`references/security-patterns.md`.

---

## Tone rules

- Address code, never the author
- Explain *why*, not just *what*
- Always include "What's done well" — even a small thing
- If nothing is wrong: say so clearly, don't invent issues
- Keep findings actionable: if you can't suggest a fix, don't flag it as critical

---

## Edge cases

**No diff available, only description**: Ask for code. Do not review assumptions.

**Trivial change** (typo fix, config tweak): Produce a short 3-line summary + verdict only. Skip the full report structure.

**Large MR (> 500 lines changed)**: Note the size, suggest splitting if logical seams exist, then proceed with review prioritizing high-risk areas first.

**Incomplete context** (function called but not shown): Flag assumptions explicitly in the finding rather than guessing behavior.

---

## Reference files

| File | When to load |
|---|---|
| `references/python.md` | Diff contains `.py` files |
| `references/java.md` | Diff contains `.java` files |
| `references/kotlin.md` | Diff contains `.kt` files |
| `references/typescript.md` | Diff contains `.ts`, `.tsx`, `.js` files |
| `references/go.md` | Diff contains `.go` files |
| `references/security-patterns.md` | Security-focused review or Phase 3 needs depth on any stack |
| `references/maintainability.md` | Maintainability audit, technical debt, DRY/YAGNI/KISS, coupling, cohesion |
| `references/documentation.md` | Documentation accuracy, stale comments, JSDoc/docstring drift, API doc gaps |
