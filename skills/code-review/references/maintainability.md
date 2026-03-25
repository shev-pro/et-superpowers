# Maintainability — Review Checks

Load this file when performing a maintainability-focused review, or when the user
asks about code quality, technical debt, design principles, or architecture concerns.

> **Read-only audit**: findings here are analytical only — do not modify code.

---

## Issue Categories

### DRY (Don't Repeat Yourself)
Flag:
- Duplicate functions or copy-pasted logic blocks across files
- Redundant type definitions serving the same concept
- Repeated validation patterns that could be extracted
- Third (or more) copy of a pattern already duplicated elsewhere

Not DRY violations:
- Test setup repetition (intentional for isolation)
- Type definitions mirroring API contracts (documentation, not duplication)
- Similar-but-different code serving distinct business rules

### YAGNI (You Aren't Gonna Need It)
Flag:
- Over-engineered abstractions with no current consumer
- Unused flexibility points / configuration options nobody uses
- Premature generalizations ("this might be useful someday")
- Speculative features not tied to a concrete requirement

### KISS (Keep It Simple, Stupid)
Flag:
- Unnecessary indirection layers (A calls B calls C calls D with no added value)
- Mixed concerns in a single unit
- Deep nesting (> 3 levels without clear reason)
- Convoluted control flow where a simpler approach exists
- Abstractions that obscure rather than clarify

### Dead Code
Flag:
- Unused functions / unreferenced imports / orphaned exports
- Commented-out code blocks
- Unreachable branches
- Vestigial parameters never read by the function body

### Concept & Contract Drift
Flag:
- The same domain concept represented in multiple incompatible ways across layers
  (e.g., `User` in domain vs `UserDTO` in persistence with different field names/types
  and no clear mapping layer)
- Glue code proliferating to bridge the incompatible representations
- Brittle invariants maintained manually in multiple places

### Boundary Leakage
Flag:
- Domain logic referencing persistence/ORM types directly
- Presentation/formatting logic inside core business logic
- Internal implementation details exported and used by unrelated modules
- Changes in one layer requiring edits in a logically unrelated layer

### Migration Debt
Flag:
- Dual fields / deprecated formats kept "temporarily" without a removal date
- Transitional wrappers or compatibility bridges with no concrete exit plan
- `@deprecated` annotations without follow-up tickets or timeline

### Coupling Issues
Flag:
- Circular dependencies between modules
- God objects that know too much (> 5-7 unrelated responsibilities)
- Feature envy: method using more of *another* class's data than its own
- Tight coupling that makes isolated unit testing impossible

### Cohesion Problems
Flag:
- Single file/module handling 3+ concerns from different layers
- Shotgun surgery: one logical change requiring scattered edits across many files
- Divergent change: one module changed for multiple unrelated reasons

### Testability Blockers
Flag:
- Hard-coded dependencies (no injection point for test doubles)
- Global or static mutable state accessed from 2+ modules
- Hidden side effects in constructors or module initialization
- Missing seams (no interface, abstract class, or callback to substitute)
- Constructors doing real work (network calls, file I/O, DB queries)

### Temporal Coupling
Flag:
- Hidden dependencies on execution order not enforced by types or constructor args
- Initialization sequences that silently break if called out of order
- "Must call `init()` before `use()`" patterns without enforcement

### Common Anti-Patterns
Flag:
- **Data clumps**: same group of 3+ parameters always appearing together → extract to object
- **Long parameter lists**: 5+ parameters → introduce parameter object
- Linter/type suppression abuse (see below)

---

## Linter / Type Suppression Audit

For every suppression found, ask: *Is this genuinely necessary, or hiding a fixable issue?*

| Suppression | Language | Severity if unexplained |
|---|---|---|
| `eslint-disable` | JS/TS | 🟡 Medium |
| `@ts-ignore` | TypeScript | 🟡 High |
| `@ts-expect-error` | TypeScript | 🟡 High |
| `# type: ignore` | Python | 🟡 Medium |
| `// nolint` | Go | 🟡 Medium |
| `@SuppressWarnings` | Java/Kotlin | 🟢 Low–Medium |

A suppression is acceptable if it has an explanation comment and there's no better fix.
A suppression is a finding if it's unexplained or hiding a real structural issue.

---

## Actionability Filter

**Before reporting any issue, it must pass ALL of these.** Drop findings that fail any criterion.

1. **In scope**
   - Diff-based review: only report issues *introduced or meaningfully worsened* by the change. Pre-existing tech debt is out of scope.
   - Explicit path review: audit everything in scope — pre-existing issues are valid.

2. **Worth the churn** — fix value must exceed refactor cost.
   Rule of thumb: worth it if `(lines eliminated) >= 50% of (lines added for abstraction + lines changed at call sites)`.

3. **Matches codebase patterns** — don't demand abstractions absent elsewhere. If the codebase doesn't use DI, don't flag its absence.

4. **Not an intentional tradeoff** — some duplication is intentional (test isolation, avoiding coupling). If the same pattern exists in 2+ other places unchallenged, assume it's a convention.

5. **Concrete impact** — "could be cleaner" is not a finding. Required: "Will cause X problem because Y."

6. **Author would prioritize** — would a reasonable author fix this before shipping? If no → Low severity at best.

7. **High confidence** — "this WILL cause X because Y", not "this might cause issues".

---

## Severity Guide

| Level | Maintainability patterns |
|---|---|
| 🔴 Critical | Exact code duplication across files; dead code misleading developers; mixed concerns preventing testing; 2+ incompatible representations of same concept; circular dependencies; global mutable state across modules |
| 🟡 High | Near-duplicate logic with minor variations; unused abstractions adding cognitive load; complex indirection with no benefit; migration debt without removal plan; single file handling 3+ concerns; long param lists (5+); hard-coded deps blocking unit tests; unexplained suppressions |
| 🟠 Medium | Minor extractable duplication; slightly over-engineered solutions; moderate simplifiable complexity; small consistency deviations; suppression without explanation |
| 🟢 Low | Stylistic inconsistencies; minor naming improvements; unused imports; well-documented suppressions that could be removed |

**Calibration**: maintainability reviews rarely have more than 1-2 Critical issues. If you're marking more than two as Critical, recheck each against the explicit Critical patterns above.

---

## Output Format

```
## Maintainability Review — [scope]

### Executive Assessment
[3-5 sentences on overall state, most significant concerns]

### [SEVERITY] Issue Title
**Category**: DRY | YAGNI | KISS | Dead Code | Consistency | Coupling | Cohesion |
               Testability | Concept Drift | Boundary Leakage | Migration Debt |
               Anti-pattern | Suppression
**Location**: `file.ts:line`, `other.ts:line`
**Description**: What the issue is
**Evidence**: [code snippet showing the issue]
**Impact**: Specific consequence — "Will cause shotgun surgery when X changes"
**Effort**: Quick win (<30 min) | Moderate refactor (1-4h) | Significant restructuring
**Suggested Fix**: Concrete recommendation
```

---

## False Positives to Avoid

- Test file duplication → intentional for isolation, skip
- Type definitions mirroring API contracts → documentation, not duplication
- Similar-but-different code serving distinct business rules → verify before flagging
- Intentional denormalization for performance → check for comments/context first

---

## Out of Scope (handled elsewhere)

| Topic | Where it goes |
|---|---|
| Type safety issues (primitive obsession, stringly-typed APIs) | `$review-type-safety` |
| Documentation accuracy (stale comments, doc drift) | `$review-docs` |
| Functional bugs (runtime errors, crashes) | `$review-bugs` |
| Test coverage gaps | `$review-coverage` |
| AGENTS.md compliance | `$review-agents-md-adherence` |
