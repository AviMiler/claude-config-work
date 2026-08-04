# Stage 2 (Build) — Detail

Read this only while actually in Stage 2. Not needed for the other stages.

## Write for agent legibility

On top of the general quality rules from `anthropic-skills:agentic-code` and
`code-standards`, code should be structured so a future agent (with no
memory of this task) can understand it without reading the whole codebase:

- **One-line purpose header** at the top of every file you create or
  substantially rewrite, stating what it's responsible for. Must match the
  wording of that file's `PROJECT_MAP.md` entry exactly — if one changes,
  the other must too (see doc-sync below).
- **Explicit imports, no wildcards.** A future PROJECT_MAP import/dependency
  listing has to be derivable mechanically, not guessed.
- **Full type signatures on inputs/outputs** of functions and public
  interfaces — a caller (agent or human) should know a function's contract
  from its signature, without reading the body.
- **No indirection beyond 2 files to understand one behavior.** If tracing
  what a piece of code does requires jumping through 3+ files (generic
  abstraction layers, dynamic dispatch, config-driven wiring), that's a
  smell — flatten it unless there's a real reason for the layer.
- **Predictable file placement** — follow the project's existing folder
  convention so file location is guessable, not something to search for.

## Keep documentation honest, not just additive

When a change **modifies or removes** existing behavior — not just adds
new code — find and update every place that documented the *old* behavior:
the file's purpose header, its `PROJECT_MAP.md` entry, `ARCHITECTURE.md` if
the change touches what it describes, and `spec.md` if a requirement
changed. A rename, a removed function, an inverted default, a deleted
module — each of these leaves a stale doc somewhere if you only ever add
and never correct. This is a Stage 2 responsibility (you know what you
removed/changed); Stage 4 then formalizes it, but Stage 4 only sees what
you flag, so surface it explicitly in the Stage 3/4 brief.
