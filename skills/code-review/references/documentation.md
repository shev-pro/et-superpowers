# Documentation — Review Checks

Load this file when the review involves documentation accuracy, stale comments,
JSDoc/docstring drift, or API documentation gaps.

> **Read-only audit**: findings here are analytical only — do not modify any files.

---

## What to Look For

### Code Comments in Changed Files
- JSDoc / docstrings that don't match current function signatures (wrong param names, types, return type)
- Comments describing behavior that no longer exists
- TODO/FIXME comments now resolved or stale
- Inline comments explaining code that has since changed
- Example code in comments that would fail to run
- Type annotations in comments contradicting actual types

### Documentation Files
Check for presence and accuracy of:
- `README.md` at root and in subdirectories
- `docs/` directories
- `SPEC.md`, `CHANGELOG.md`, `CONTRIBUTING.md`
- `AGENTS.md` at project root
- Plugin/skill-specific: `plugin.json`, `SKILL.md`

### For Each Code Change, Verify Docs Reflect:
- New / changed / removed API signatures or behavior
- New / changed / removed configuration options
- New / changed / removed commands, agents, hooks, or skills
- Changed installation or setup steps
- Changed file paths or locations
- Changed usage examples or patterns

---

## Actionability Filter

**Before reporting any doc issue, it must pass ALL of these.** Drop findings that fail any criterion.

1. **In scope**
   - Diff-based review: only report doc issues *caused by* the code changes. Pre-existing doc problems are out of scope.
   - Explicit path review: audit everything in scope — pre-existing inaccuracies are valid.

2. **Actually incorrect or missing** — "could add more detail" is not a finding. "This parameter is documented as optional but the code requires it" is a finding.

3. **User would be blocked or confused** — would someone following this doc fail, get an error, or waste significant time? If yes, report it. If they'd figure it out quickly, it's Low at best.

4. **Not cosmetic** — formatting, wording preferences, and "could be clearer" are Low. Focus on factual accuracy.

5. **Matches doc depth** — don't demand comprehensive API docs in a project with minimal docs. Match existing documentation style and depth.

6. **High confidence** — "this doc says X but the code does Y", not "this could be improved".

---

## Severity Guide

> Documentation issues are **capped at Medium** — docs don't cause data loss or security breaches.

| Level | Documentation patterns |
|---|---|
| 🟠 Medium | Examples that error; incorrect API signatures / param names / file paths; new features with zero docs; major behavior changes not reflected; removed features still documented; incorrect install/setup steps; JSDoc with wrong param names or types |
| 🟢 Low | Minor param/option changes not reflected; outdated examples that still work; missing edge cases or caveats; stale TODO/FIXME comments; minor wording; formatting inconsistencies |

If you're tempted to go above Medium, reconsider — even actively misleading docs are Medium because users can recover by reading the code.

---

## Output Format

```
## Documentation Audit — [scope]

### Code Changes Analyzed
- `path/to/file.ts`: [brief description of change]

### Documentation Issues

**[SEVERITY] Issue Title**
**Location**: `path/to/doc.md` (line X-Y)
**Related Code**: `path/to/code.ts:line`
**Problem**: Clear description of the discrepancy
**Current doc says**: [quote or summary]
**Code actually does**: [what code does]
**Suggested update**: Specific text or change needed

### Missing Documentation
[New features/changes with no documentation at all]

### Code Comment Issues

**[SEVERITY] Issue Title**
**Location**: `path/to/code.ts:line`
**Problem**: Description of stale/incorrect comment
**Current comment**: [quote]
**Actual behavior**: [what code does]
**Suggested update**: Specific replacement or "Remove comment"

### Summary
- Medium: N
- Low: N
```

---

## Writing Standard for Suggestions

When proposing documentation updates:
- **Mirror the doc's format** — if it uses tables, suggest table updates; bullets → bullets
- **Match heading hierarchy** — follow existing H1/H2/H3 structure
- **Preserve voice** — technical docs stay technical
- **Keep consistent conventions** — if the doc uses `` `code` `` for commands, do the same
- **Commands, flags, paths must match code exactly** — verify before suggesting

---

## False Positives to Avoid

- Docs that are intentionally high-level (don't demand implementation detail)
- Examples simplified for clarity that still demonstrate the correct concept
- Version numbers in changelogs (historical, not stale)
- Comments explaining *why*, not *what* — these rarely go stale

---

## Out of Scope (handled elsewhere)

| Topic | Where it goes |
|---|---|
| Code bugs | `$review-bugs` |
| Code organization (DRY, coupling) | `$review-maintainability` |
| Type safety | `$review-type-safety` |
| Test coverage gaps | `$review-coverage` |
| AGENTS.md compliance | `$review-agents-md-adherence` |
