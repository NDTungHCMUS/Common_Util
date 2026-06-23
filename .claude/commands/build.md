---
description: Implement tasks incrementally — code, build, add test, run test. Add "auto" to run the whole plan in one approved pass.
---

Invoke the agent-skills:incremental-implementation skill.

This project does **not** use test-first (TDD). Each task still produces both code and a test, but the test is written **after** the implementation (test-after) — it does not need to fail first. Keep tests focused on the task's behavior; this is an MVP demo, not production.

## Modes

- **`/build`** — implement the *next* pending task, then stop (careful, one slice at a time).
- **`/build checkpoint`** — advance one *phase* at a time: run autonomously through the next checkpoint boundary, then stop for the human review the plan asks for.
- **`/build auto`** — generate the plan if needed, get a single approval, then implement *every* task without stopping between them.

`$ARGUMENTS` selects the mode. Treat `auto` (canonical) or `all` as autonomous mode; `checkpoint` (or `phase`) as checkpoint mode; anything else (or empty) is the default single-task mode. Note: autonomous and checkpoint modes are not faster *per task* — they run the same build-verify loop — they only remove the human stepping *between* tasks.

## Default: one task

Pick the next pending task from the plan. Then:

1. Read the task's acceptance criteria
2. Load relevant context (existing code, patterns, types)
3. Implement the minimum code that satisfies the acceptance criteria
4. Run `./gradlew build` to verify compilation
5. Write a test covering the new behavior (test-after — no need to fail first)
6. Run the test (`./gradlew test --tests <ClassName>`)
7. Mark the task complete and stop

## Autonomous: the whole plan (`/build auto`)

Use this once a spec exists and you want to collapse plan + build into one run. It removes the manual stepping between tasks — **not** the verification. Every task still builds clean and earns a passing test.

1. **Require a spec.** Look only for a spec at a known path: `SPEC.md` at the repo root, `docs/SPEC.md`, or a file under `spec/`. A README or arbitrary doc does **not** count. If none exists, stop and tell the user to run `/spec` first — do not invent requirements.
2. **Establish a clean baseline.** Run `git status --porcelain` and note what was already dirty before you start, so the changes you make this run stay distinguishable from pre-existing local work. (This run does not commit per task — see step 4.)
3. **Plan if needed.** If there is no `tasks/plan.md`, invoke agent-skills:planning-and-task-breakdown to generate one.
4. **Single checkpoint.** Present the full plan and wait for an unambiguous affirmative (e.g. "approve", "go", "yes"). Treat hedged responses ("looks reasonable", "I guess") as **not** approved. This is the only human gate — after approval, run autonomously. This run does not commit; leave committing to the user once the work looks right.
5. **Execute every task in dependency order.** Use each task's declared dependencies; if they aren't explicit, execute in the order the plan lists them. For each task, run the full default loop above (implement → `./gradlew build` → write test → run test → mark complete).
6. **Stop and ask the user** (do not push through) when:
    - a test can't be made to pass or the build breaks without an obvious fix → follow agent-skills:debugging-and-error-recovery
    - the spec is ambiguous, or a task needs a decision the spec doesn't cover
    - a task is high-risk or irreversible — auth/permission changes, destructive data migrations, payments, deletions, deploys, anything touching secrets, **or anything you can't undo with `git revert`** → follow agent-skills:doubt-driven-development and get explicit sign-off before continuing

   After the user resolves a blocker, they re-invoke `/build auto` — it resumes from the next pending task.
7. **Summarize at the end:** tasks completed, tests added, build/test status, and anything skipped, flagged, or left for the user.

## Checkpoint: through the next checkpoint (`/build checkpoint`)

Advance one *phase* at a time: runs autonomously like `/build auto`, but stops at the next checkpoint boundary so the human review the plan asks for actually happens. Same verification per task.

1. **Require a plan with checkpoints.** Read `tasks/plan.md` (and `tasks/todo.md`). It must define checkpoints — e.g. a `## Checkpoints` section with "Checkpoint A — after Task 3", "Checkpoint B — after Task 6", etc. If there is no plan, fall back to `/build auto` step 2 to generate one (which includes checkpoints); if a plan exists but defines no checkpoints, tell the user and offer `/build auto` instead — do not invent boundaries.
2. **Find the target checkpoint.** Locate the first pending task, then find the *nearest* checkpoint at or after it whose tasks aren't all complete. The target scope is every pending task from the first one through that checkpoint's last task (inclusive).
3. **Single checkpoint approval.** Present just the tasks in this phase (not the whole plan) and the checkpoint's exit criteria, and wait for an unambiguous affirmative. Hedged responses are **not** approval. This is the only human gate until the checkpoint.
4. **Execute the phase in dependency order.** For each task in scope, run the full default loop (implement → `./gradlew build` → write test → run test → mark complete).
5. **Stop and ask** under the same conditions as `/build auto` step 5 (unfixable failure, ambiguity, high-risk/irreversible work).
6. **At the checkpoint, halt and verify the gate.** Run the checkpoint's exit criteria from the plan (tests green, build clean, the named flow works). Report each criterion PASS/FAIL and **stop** — do not roll into the next phase. Summarize tasks completed, tests added, and the checkpoint verdict. The user re-invokes `/build checkpoint` to advance to the next one, or `/build` for a single task.

If any step fails, follow the agent-skills:debugging-and-error-recovery skill.
