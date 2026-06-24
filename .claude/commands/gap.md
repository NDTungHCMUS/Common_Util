---
description: Analyze the gap between the current codebase and a new feature, then produce an implementation plan with a unit test per task
---

Invoke the feature-gap-analysis skill for the feature the user describes: $ARGUMENTS

Work read-only (plan mode — no code changes). Then:

1. Lock the feature in one sentence. If anything is ambiguous, surface the
   options and stop for an answer — do not guess (CLAUDE.md rule 1).
2. Derive the components the feature needs, by layer (the "defined" set).
3. Inspect the codebase for each — use codegraph explore/search first if
   available, otherwise `grep`/`rg`/`Read`. Mark each EXISTS / PARTIAL / MISSING.
4. Produce the GAP table (needed × current state × work to close).
5. Turn the PARTIAL + MISSING rows into a dependency-ordered task list. Tasks
   1…N-1 are code + a focused unit test; Task N is the single integration test
   depending on all prior tasks. Every task names the unit test that proves it.
6. Present the document for human review before any code is written.

Save the result to GAP.md in the project root.
