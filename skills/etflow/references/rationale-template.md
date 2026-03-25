# Project Rationale

> This document captures intentional deviations from standard best practices,
> historical decisions with non-obvious reasoning, and constraints that future
> developers (or AI assistants) must be aware of before proposing changes.
>
> **Before suggesting "improvements", check here first.**

Last updated: YYYY-MM-DD

---

## How to read this document

Each entry follows this structure:

- **What it looks like**: the pattern that appears wrong or suboptimal at first glance
- **Why it exists**: the real reason — constraint, trade-off, deliberate choice
- **Do not change unless**: conditions under which revisiting this decision is legitimate
- **Added**: date and context when this decision was recorded

---

## Entries

### [Short title of the decision]

**What it looks like**: [describe the pattern that seems like a bug or bad practice]

**Why it exists**: [the actual reason — legacy constraint, external API limitation,
team agreement, performance trade-off, security requirement, etc.]

**Do not change unless**: [specific condition that would make revisiting this legitimate —
e.g. "Ducont enables native callbacks", "we migrate off PostgreSQL", "SDK version >= X"]

**Added**: YYYY-MM-DD — [brief context, e.g. "discovered during DDA QA deployment"]

---

<!-- Add new entries above this line -->
