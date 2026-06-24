---
name: spec-driven-development
description: Creates specs before coding. Use when starting a new project, feature, or significant change and no specification exists yet. Use when requirements are unclear, ambiguous, or only exist as a vague idea.
---

# Spec-Driven Development

## Overview

Write a structured spec before writing code. The spec is the shared source of truth — it defines what we're building, why, and how we'll know it's done. Code without a spec is guessing.

## When to Use

Use for new features, ambiguous requirements, multi-file changes, or anything over ~30 minutes.
**Skip** for single-line fixes, typos, or self-contained unambiguous changes.

## Workflow

This skill owns one gate: **write the spec and get it approved before any code.** Planning and implementation happen after, in their own skills.

```
SPECIFY ──→ human approves ──→ plan + build
```

1. Start from the high-level vision. Ask clarifying questions until requirements are concrete.
2. **List your assumptions first** — don't silently fill in ambiguity. Surfacing misunderstandings before code is the whole point.
3. Write the spec (template below) and get human approval.

## Spec Template

```markdown
# Spec: [Feature Name]

## Objective
What we're building and why. Who's the user? What does success look like?

## User Journey & Business Rules
- **Journey** — numbered steps from entry point to goal (actor → action → system response).
- **Business rules** — validation, eligibility, state transitions, edge cases, what happens on violation.
- **Scenarios** — critical paths as Given/When/Then (these become the acceptance tests).

## Tech Stack
Framework, language, key dependencies with versions.

## Commands
Full build / test / lint / dev commands with flags.

## Project Structure
Directory layout — where source, tests, and docs live. Unit and integration
tests share one `src/test/java` source set but live in **separate packages**:
unit tests sit beside the layer they cover (`…/service`, `…/controller`,
`…/repository`) as `*Test.java`; integration tests go under a dedicated
`…/integration` package as `*IT.java`. Example:

```
src/test/java/com/example/<app>/
├── service/        ← unit tests (*Test.java, mocked collaborators)
├── controller/     ← unit tests (*Test.java)
├── repository/     ← unit tests (*Test.java)
└── integration/    ← integration tests (*IT.java, full assembled stack)
```

## Code Style
One short real code snippet.

## Testing Strategy
Framework, coverage expectations, and which test level for which concern.
State the split explicitly: **Tasks 1 … N-1 are covered by unit tests**
(`*Test.java`, mocked collaborators, beside their layer); **Task N is the one
integration test** (`*IT.java` under `…/integration`, exercising the
wired-together stack end-to-end).
```

## After the Spec

Once approved, hand off:
- **Plan & tasks** → `planning-and-task-breakdown` skill or `/plan`.
- **Implement** → `incremental-implementation` skill or `/build`.

Keep the spec alive: when decisions or scope change, update the spec first, then implement.

## Verification

- [ ] Spec covers all seven sections above
- [ ] Journey is concrete steps with business rules and Given/When/Then scenarios
- [ ] Human has reviewed and approved
- [ ] Spec is saved to a file in the repo
