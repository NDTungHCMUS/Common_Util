---
description: Implement tasks incrementally — code, build, add test, run test. Add "auto" to run the whole plan in one approved pass.
---

Invoke the agent-skills:incremental-implementation skill.

No TDD here. Each task produces code **and** a test, but the test is written **after** the code (it need not fail first). Keep tests focused on the task; this is an MVP demo.

## Modes

`$ARGUMENTS` selects the mode:

- **`/build`** (default) — implement the *next* pending task, then stop.
- **`/build checkpoint`** (or `phase`) — run autonomously to the next checkpoint boundary, then stop for review.
- **`/build auto`** (or `all`) — one approval, then implement *every* task without stopping between them.

These modes don't make each task faster — they only remove the human step *between* tasks. Same build-verify loop runs every time.

## Default: one task

Pick the next pending task from the plan, then:

1. Read its acceptance criteria; load relevant context.
2. Implement the minimum code that satisfies them.
3. `./gradlew build` to verify compilation.
4. Write a test for the new behavior; run it (`./gradlew test --tests <ClassName>`).
5. Mark complete and stop.

## Autonomous: the whole plan (`/build auto`)

Removes the manual stepping between tasks, not the verification — every task still builds clean and earns a passing test.

1. **Require a spec.** Look only at `SPEC.md`, `docs/SPEC.md`, or a file under `spec/`. If none, stop and tell the user to run `/spec` — don't invent requirements.
2. **Plan if needed.** No `tasks/plan.md` → invoke agent-skills:planning-and-task-breakdown to generate one (with checkpoints).
3. **Single approval gate.** Present the full plan; wait for an unambiguous "yes" (hedges don't count). This is the only human gate. Don't commit — leave that to the user.
4. **Execute every task in dependency order**, running the default loop above for each.
5. **Stop and ask** when a build/test fails without an obvious fix or the spec is ambiguous. Re-invoking `/build auto` resumes from the next pending task.
6. **Summarize at the end:** tasks completed, tests added, build/test status, anything skipped or flagged.

## Checkpoint: through the next checkpoint (`/build checkpoint`)

Like `/build auto`, but stops at the next checkpoint boundary for the review the plan asks for.

1. **Require a plan with checkpoints** in `tasks/plan.md` (e.g. a `## Checkpoints` section). No plan → generate one (auto step 2). Plan but no checkpoints → tell the user and offer `/build auto`.
2. **Find the target checkpoint** — the nearest one at or after the first pending task whose tasks aren't all done. Scope = every pending task through that checkpoint.
3. **Single approval gate.** Present just this phase's tasks and the checkpoint's exit criteria; wait for an unambiguous "yes".
4. **Execute the phase in dependency order** with the default loop.
5. **Stop and ask** under the same conditions as `/build auto` step 5.
6. **At the checkpoint, halt and verify the gate** — run the exit criteria, report each PASS/FAIL, and stop. Don't roll into the next phase. The user re-invokes `/build checkpoint` to continue.

If any step fails, follow agent-skills:debugging-and-error-recovery.
