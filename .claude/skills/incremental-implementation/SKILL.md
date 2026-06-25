---
name: incremental-implementation
description: Delivers changes incrementally. Use when implementing any feature or change that touches more than one file. Use when you're about to write a large amount of code at once, or when a task feels too big to land in one step.
---

# Incremental Implementation

## Overview

Build in thin vertical slices: implement one piece, test it, verify it, then expand. Each increment leaves the system building and testable. Never implement an entire feature in one pass.

## When to Use

- Any multi-file change, new feature, or refactor
- Any time you're tempted to write >100 lines before testing

**Skip when:** single-file, single-function change with minimal scope.

## The Increment Cycle

For each slice: **Implement** the smallest complete piece → **Test** (run/write tests) → **Verify** (build + tests pass) → move to the **next slice** (carry forward, don't restart).

**Comment the purpose of every test.** Whether unit or integration, each test method carries a brief comment (or Javadoc) saying *what behavior it verifies and why* — the intent should be clear without reading the assertions.

**Slicing:** Prefer **vertical** slices — one complete path through the stack (e.g. Create task → List → Edit → Delete), each working end-to-end. If one piece is risky or uncertain, do it **first** so failure surfaces early.

## Rules

1. **Simplest thing that works** — naive and obviously-correct before any abstraction. No speculative features. Three similar lines beat a premature abstraction.
2. **Stay in scope** — touch only what the task needs. Don't refactor adjacent code; note out-of-scope improvements instead of fixing them.
3. **One thing per increment** — don't mix concerns (e.g. a new component + a refactor + a config change).
4. **Always buildable** — after each increment the project compiles and existing tests pass. Never leave a broken state between slices.

## Increment Checklist

After each increment:
- [ ] Does one thing, completely
- [ ] Build succeeds; existing tests pass; new behavior works
- [ ] Stayed in scope

Run a verification command only after a change that could affect it.

## When Done

After all increments: each was individually verified, the full suite passes, the build is clean, and the feature works end-to-end as specified.
