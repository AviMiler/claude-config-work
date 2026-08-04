---
name: code-standards
description: >
  Personal code-writing standards — additions on top of the official
  anthropic-skills:agentic-code skill, not a replacement for it. A growing
  list of rules the user adds to over time. Always load alongside
  anthropic-skills:agentic-code during Stage 2 (Build) of
  enforcing-coding-workflow, and for any other code-writing task.
---

# Code Standards (Personal Additions)

This skill does **not** replace `anthropic-skills:agentic-code` — it sits
on top of it. Load both. Where they overlap, this file's rules are more
specific to how this user wants code written; where they don't overlap,
both apply.

This file grows one rule at a time. Each rule is self-contained — read the
whole file, but a rule shouldn't depend on context from another rule to
make sense.

## Rules

### 1. File and folder organization

Split code into files and folders by topic — never one large file mixing
unrelated responsibilities.

- **One file, one responsibility.** If a file handles two distinct concerns
  (e.g. data access + business logic, or UI + state management), split it.
- **One folder per topic/domain**, not per technical layer alone. Group by
  what the code is *about* (e.g. `auth/`, `billing/`, `users/`), and only
  subdivide by technical role (`handlers`, `models`, `utils`) *inside* that
  topic folder when the topic grows large enough to need it — not before.
- **Follow the target language's own best-practice convention for this**,
  don't invent a generic structure. What "idiomatic" means differs by
  language/ecosystem:
  - Python: packages (`__init__.py`) per domain, not a flat `utils.py`
    dumping ground.
  - JS/TS: feature-based folders (`src/features/auth/`) over type-based
    top-level folders (`src/components/`, `src/hooks/` mixing every
    feature together) once the project has more than a couple of features.
  - Go: `internal/` for private packages, `pkg/` for anything meant to be
    imported externally, one package per domain concept.
  - Java/Kotlin: package-per-domain under the group/artifact root, not
    package-per-layer (`controllers/`, `services/`, `models/` spanning
    every feature).
  - If unsure which convention applies, check what the ecosystem's own
    standard tooling/framework scaffolds by default (e.g. what `django-admin
    startapp`, `create-react-app`, or the framework's official generator
    produces) rather than guessing.
- This is a Stage 2 (Build) rule — apply it while writing, not as cleanup
  after. If an existing file in the project already violates this and the
  current task touches it, split it as part of the change (this counts as
  in-scope, not a drive-by refactor, since it's the same file the task is
  already modifying).

<!-- Next rule goes here. One `### N. Title` section per addition. -->
