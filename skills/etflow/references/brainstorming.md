# Brainstorming — Collaborative Exploration Process

This document describes the process to follow in **Phase 3B** of etflow to turn a complex
requirement into a validated design through dialogue with the user.

---

## 1. Understand the project context

Before asking any questions, explore the project:

- Read the main files (README, directory structure, entry points)
- Check any existing documentation in `docs/`
- If available, look at recent commits to understand the current direction
- Identify technologies, architectural patterns, and conventions already in use

This allows you to ask relevant questions and propose approaches consistent with the real context.

---

## 2. Refine the idea — one question at a time

Ask questions to clarify purpose, constraints, and success criteria. Hard rules:

- **One question per message** — if you have more things to ask, start with the most important
- **Prefer multiple choice** — easier to answer than open-ended questions
- **Stop when you have enough** — don't interrogate beyond what's needed

Areas to explore (not all necessarily):
- Why this feature? What is the real underlying problem?
- Who are the users/actors involved?
- Are there technical or business constraints to respect?
- What defines success for this implementation?
- Are there dependencies on other systems or teams?

---

## 3. Propose 2-3 approaches

Before presenting the design, propose alternatives:

- Describe **2-3 different approaches** conversationally
- For each, briefly outline pros and cons
- **Recommend the preferred approach** with clear reasoning
- Let the user choose, but guide the decision

Apply **YAGNI** to every approach: remove everything that is not strictly necessary
to satisfy the current requirement. No over-engineering.

---

## 4. Present the design in sections

Once an approach is chosen, present the design incrementally:

- **200-300 words per section** — no more
- **Ask for confirmation after each section**: *"Does this make sense so far? Anything you'd change before continuing?"*
- **Be ready to go back** if something doesn't fit
- Cover in whatever order makes sense for the context: architecture, components, data flow, error handling, testing

Never present the entire design as one block. Incremental validation is essential
to avoid building something that doesn't meet the actual need.

---

## 5. Completion signal

The brainstorming is complete when:

- The user has confirmed each section of the design
- There are no open questions or significant ambiguities
- You have enough information to populate all sections of the design document

At that point, return to **Phase 3B Step 2** of etflow and write the document.
