---
name: planning-and-task-breakdown
description: Breaks work into ordered tasks. Use when you have a spec or clear requirements and need to break work into implementable tasks. Use when a task feels too large to start, when you need to estimate scope, or when parallel work is possible.
---

# Planning and Task Breakdown

## Overview

Decompose work into small, verifiable tasks with explicit acceptance criteria. Good task breakdown is the difference between an agent that completes work reliably and one that produces a tangled mess. Every task should be small enough to implement, test, and verify in a single focused session.

## When to Use

- You have a spec and need to break it into implementable units
- A task feels too large or vague to start
- Work needs to be parallelized across multiple agents or sessions
- You need to communicate scope to a human
- The implementation order isn't obvious

**When NOT to use:** Single-file changes with obvious scope, or when the spec already contains well-defined tasks.

## The Planning Process

### Step 1: Enter Plan Mode

Before writing any code, operate in read-only mode:

- Read the spec and relevant codebase sections
- Identify existing patterns and conventions
- Map dependencies between components
- Note risks and unknowns

**Do NOT write code during planning.** The output is a plan document, not implementation.

### Step 2: Identify the Dependency Graph

Map what depends on what:

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

Implementation order follows the dependency graph bottom-up: build foundations first.

### Step 3: Slice Vertically

Instead of building all the database, then all the API, then all the UI — build one complete feature path at a time:

**Bad (horizontal slicing):**
```
Task 1: Build entire database schema
Task 2: Build all API endpoints
Task 3: Build all UI components
Task 4: Connect everything
```

**Good (vertical slicing):**
```
Task 1: User can create an account (schema + API + UI for registration)
Task 2: User can log in (auth schema + API + UI for login)
Task 3: User can create a task (task schema + API + UI for creation)
Task 4: User can view task list (query + API + UI for list view)
```

Each vertical slice delivers working, testable functionality.

### Step 4: Write Tasks

Keep each task to four fields — no separate "verification" or "files" blocks beyond
these (that repetition is what makes plans bloat). The **AC** line *is* the acceptance
criterion and the verification step:

```markdown
## Task [N]: [Short descriptive title]

- **Domain:** [the one layer/area this task touches — e.g. schema, API, service, repository, UI, infra, test]
- **Depends on:** [task numbers, or "None"]
- **Change details:** [what concretely changes — the files/symbols touched and the before → after]
- **AC:** [one machine-checkable condition — a named test passes, an endpoint returns X, a field persists]
```

If you cannot state **AC** as a single checkable condition, the task is too
big — split it.

**Always end with an integration-test task.** After all implementation tasks, add a
final task whose Domain is `test`, which **depends on every prior task**, and which
verifies the full feature flow end-to-end (not just one slice). Its **AC** is
the end-to-end test passing against the assembled stack.

### Step 5: Order and Checkpoint

Arrange tasks so that:

1. Dependencies are satisfied (build foundation first)
2. Each task leaves the system in a working state
3. Verification checkpoints occur after every 2-3 tasks
4. High-risk tasks are early (fail fast)

Add explicit checkpoints:

```markdown
## Checkpoint: After Tasks 1-3
- [ ] All tests pass
- [ ] Application builds without errors
- [ ] Core user flow works end-to-end
- [ ] Review with human before proceeding
```

## Task Sizing Guidelines

| Size | Files | Scope | Example |
|------|-------|-------|---------|
| **XS** | 1 | Single function or config change | Add a validation rule |
| **S** | 1-2 | One component or endpoint | Add a new API endpoint |
| **M** | 3-5 | One feature slice | User registration flow |
| **L** | 5-8 | Multi-component feature | Search with filtering and pagination |
| **XL** | 8+ | **Too large — break it down further** | — |

If a task is L or larger, it should be broken into smaller tasks. An agent performs best on S and M tasks.

**When to break a task down further:**
- It would take more than one focused session (roughly 2+ hours of agent work)
- You cannot describe the acceptance criteria in 3 or fewer bullet points
- It touches two or more independent subsystems (e.g., auth and billing)
- You find yourself writing "and" in the task title (a sign it is two tasks)

## Plan Document Template

```markdown
# Implementation Plan: [Feature/Project Name]

## The plan in one line
[What will exist or behave differently after this that doesn't now.]

## Change Manifest
One row per file touched. This table replaces an "architecture decisions" essay —
the rows are the decisions, and they double as the post-run check against `git diff`.

| File | Layer | Change (before → after) |
|------|-------|-------------------------|
| `src/.../FooController.java` | API | none → `POST /foo` |
| `src/.../FooService.java` | service | none → ordering logic |

- **Layer:** schema / API / service / repository / domain / UI / config / test.
- **Change:** concrete before→after, not intent. New files: before-state is "none".

## What changes, by layer
One sentence per layer touched — the human-readable summary of the manifest above.
A few lines, no more.

## Task List
(Tasks ordered by dependency; each in the Step 4 four-field form. The last task is
always the integration test covering all prior tasks.)

- [ ] Task 1: ...
- [ ] Task 2: ...
- [ ] Task N: Integration test — full flow end-to-end (depends on all above)
```

## Parallelization Opportunities

When multiple agents or sessions are available:

- **Safe to parallelize:** Independent feature slices, tests for already-implemented features, documentation
- **Must be sequential:** Database migrations, shared state changes, dependency chains
- **Needs coordination:** Features that share an API contract (define the contract first, then parallelize)

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll figure it out as I go" | That's how you end up with a tangled mess and rework. 10 minutes of planning saves hours. |
| "The tasks are obvious" | Write them down anyway. Explicit tasks surface hidden dependencies and forgotten edge cases. |
| "Planning is overhead" | Planning is the task. Implementation without a plan is just typing. |
| "I can hold it all in my head" | Context windows are finite. Written plans survive session boundaries and compaction. |

## Red Flags

- Starting implementation without a written task list
- Tasks that say "implement the feature" without a checkable **AC**
- No Change Manifest, or files changed that aren't in it
- No final integration-test task
- All tasks are XL-sized
- Dependency order isn't considered

## Verification

Before starting implementation, confirm:

- [ ] Every task has a single checkable **AC**
- [ ] Every task names its **Domain**, **Depends on**, and **Change details**
- [ ] Dependencies are ordered correctly (foundations first)
- [ ] The Change Manifest lists every file the tasks will touch
- [ ] A final integration-test task covers the full flow and depends on all prior tasks
- [ ] The human has reviewed and approved the plan
