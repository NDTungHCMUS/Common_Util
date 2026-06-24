---
name: planning-and-task-breakdown
description: Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into implementable tasks. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible.
---

# Planning and Task Breakdown

Decompose work into small, verifiable tasks with explicit acceptance criteria. Each task should be small enough to implement, test, and verify in one focused session.

## When to Use

Use when you have a spec to break down, a task too large or vague to start, work to parallelize, or an unclear implementation order.

**Skip** for single-file changes with obvious scope, or when the spec already lists well-defined tasks.

## The Planning Process

1. **Stay read-only.** Read the spec and relevant code, note existing patterns, dependencies, and risks. The output is a plan, not code.
2. **Order by dependency.** Build foundations first: schema → models/types → endpoints/validation → client → UI. A task ships only after the tasks it depends on.
3. **Slice vertically.** Build one complete feature path at a time (e.g. "User can register: schema + API + UI"), not one whole layer at a time. Each slice is independently testable.
4. **Front-load risk.** Put high-risk and unknown tasks early so they fail fast.
5. **Checkpoint every 2-3 tasks** so the system is always in a working state.

## Task Format

Keep each task to four fields — no separate "verification" or "files" blocks (that repetition is what bloats plans). The **AC** line *is* the acceptance criterion and the verification step:

```markdown
## Task [N]: [Short descriptive title]

- **Domain:** [the one layer this task touches — schema, API, service, repository, UI, infra, test]
- **Depends on:** [task numbers, or "None"]
- **Change details:** [what concretely changes — files/symbols touched, before → after]
- **AC:** [one machine-checkable condition — a named **unit test** passes, an endpoint returns X, a field persists]
```

If you cannot state **AC** as a single checkable condition, the task is too big — split it.

**Tasks 1 … N-1 are code + unit test only.** Each implements one slice and is verified by a focused **unit test** (mocked collaborators) — no integration test in these tasks. Don't stand up the full stack, a real DB, or end-to-end wiring inside them.

**Task N is the single integration-test task.** Its Domain is `test`, it **depends on every prior task**, and its **AC** is the end-to-end test passing against the assembled stack (the full feature flow, not one slice). This is the only task that exercises the wired-together system.

## Task Sizing

Aim for tasks of 1-5 files / one feature slice. Break a task down when it spans more than one focused session, needs more than 3 AC bullets, touches two independent subsystems, or has "and" in its title.

## Plan Document Template

```markdown
# Implementation Plan: [Feature/Project Name]

## The plan in one line
[What will exist or behave differently after this that doesn't now.]

## Change Manifest
One row per file touched — the rows are the decisions, and they double as the
post-run check against `git diff`. Layer: schema / API / service / repository /
domain / UI / config / test. Change: concrete before→after; new files start at "none".

| File | Layer | Change (before → after) |
|------|-------|-------------------------|
| `src/.../FooController.java` | API | none → `POST /foo` |
| `src/.../FooService.java` | service | none → ordering logic |

## What changes, by layer
One sentence per layer touched — the human-readable summary of the manifest.

## Task List
(Ordered by dependency; each in the four-field form. Tasks 1 … N-1 are code +
unit test only. The last task is always the integration test depending on all
prior tasks.)

- [ ] Task 1: ... (code + unit test)
- [ ] Task 2: ... (code + unit test)
- [ ] Task N: Integration test — full flow end-to-end (depends on all above)
```
