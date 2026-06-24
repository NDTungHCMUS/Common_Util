---
name: feature-gap-analysis
description: Maps the gap between the current codebase and a requested new feature, then produces an ordered implementation plan with unit tests per task. Use when a user describes a new feature and you need to know what already exists, what is missing, and how to build it. Use before writing code for any feature that touches more than one file.
---

# Feature Gap Analysis & Plan

Given a description of a new feature, figure out **what the codebase needs**,
**what it already has**, name the **gap** between them, then turn that gap into an
ordered build plan where every task carries a unit test. The output is a document,
not code.

## When to Use

Use when a user asks for a new feature and the delta against the current code is
not obvious — you need to inventory what exists before planning. Pairs with
`/spec` (clarify *what* to build) and `/planning` (order the *how*); this skill
bridges them by grounding the plan in the actual codebase.

**Skip** for a single-file change with obvious scope, or when a PLAN.md already
covers the feature.

## The Process

1. **Lock the feature.** Restate the feature in one sentence. If the request has
   open questions or multiple readings, surface them and stop — do not guess
   (matches CLAUDE.md rule 1). A locked feature is the contract the gap is measured
   against.

2. **Derive what's needed.** From the locked feature, list the components and
   layers it requires — schema / domain / repository / service / API / config /
   test. This is the **defined** set: what a working feature must contain.

3. **Inspect what exists.** Read the codebase (codegraph explore/search first if
   available, else `grep`/`rg`/`Read`). For each needed component, find whether it
   already exists, partially exists, or is absent. Stay read-only.

4. **Name the GAP.** Produce the gap table: every needed component × its current
   state (EXISTS / PARTIAL / MISSING) × the concrete work to close it. The set of
   PARTIAL + MISSING rows *is* the build backlog.

5. **Order into tasks.** Turn the backlog into dependency-ordered tasks, slicing
   vertically (one complete path per task). Tasks 1…N-1 are code **+ a focused
   unit test** (mocked collaborators). Task N is the single integration test
   depending on all prior tasks. Each task states one machine-checkable AC.

6. **Present for review.** Save the document and confirm with the user before any
   code is written.

## Gap Table Format

```markdown
| Component (needed) | Layer | Current state | Work to close the gap |
|--------------------|-------|---------------|-----------------------|
| TaskController     | API   | MISSING       | none → POST/GET endpoints |
| Priority enum      | domain| PARTIAL       | add weight() accessor |
| application.yml    | config| EXISTS        | — (no change) |
```

`Current state` is one of **EXISTS** (use as-is), **PARTIAL** (modify), or
**MISSING** (create). EXISTS rows are dropped from the backlog; PARTIAL + MISSING
rows become tasks.

## Task Format

Each backlog task uses the four-field form. The **Unit test** field names the
focused test that proves the task — this is what makes the plan testable:

```markdown
## Task [N]: [Short descriptive title]

- **Domain:** [the one layer this task touches]
- **Depends on:** [task numbers, or "None"]
- **Change details:** [what concretely changes — files/symbols, before → after]
- **Unit test:** [the test class/method + the one condition it asserts]
- **AC:** [that named unit test passes]
```

If you cannot name a single unit test that proves the task, it is too big — split
it. **Task N** is the lone integration test: Domain `test`, depends on every prior
task, AC is the end-to-end flow passing against the assembled stack.

## Output Document Template

```markdown
# Gap Analysis & Plan: [Feature Name]

## Feature (locked)
[One sentence. Open questions, if any, listed and resolved.]

## What the feature needs
[Bulleted list of required components by layer — the "defined" set.]

## GAP
[The gap table above — needed × current state × work to close.]

## Task List
(Backlog = PARTIAL + MISSING rows, dependency-ordered. Tasks 1…N-1 are
code + unit test; Task N is the integration test depending on all above.)

- [ ] Task 1: ... (code + unit test)
- [ ] Task N: Integration test — full flow end-to-end (depends on all above)

## Tasks
[Each task in the four-field-plus-unit-test form above.]

## Checkpoints
[After which tasks the suite should be green — e.g. `./gradlew test`.]
```
