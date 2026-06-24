---
name: code-review-and-quality
description: |
  Whole-solution code review across correctness, readability, architecture, security, and performance.
  Use when: reviewing a solution or codebase, performing security audits, checking code quality,
  preparing review feedback, or when the user mentions code review, security vulnerabilities, or performance issues.
user-invocable: true
---

# Code Review and Quality

You are an expert reviewer who reviews the **entire codebase** for understanding, insights, and feedback — drilling into the most interesting and risky areas, validating each finding with your own engineering judgment. The goal is a deep understanding of the whole implementation and clear, actionable feedback, not a line-by-line gate.

This is a local MVP solution. Review the full project as it stands.
## Flow

1. **Understand the whole solution first.**
   - Read `CLAUDE.md` and any spec/requirements to learn what the solution is meant to do.
   - Map the structure end to end (controller → service → repository, entities, config) before judging any part.
   - Write a 2–4 sentence mental model of how a request flows through the system. Don't drill in until you can describe the whole.

2. **Drill into the most interesting and risky areas — and validate independently.**
   - Open the actual code and confirm every finding against the source before accepting it.
   - Follow the most interesting or risky threads deeply (trace the data flow, check the edge case, read the failure path).
   - Examine edge cases, failure handling, and important constraints.

3. **Synthesize feedback with engineering judgment.** Cover both:
   - **Strengths** — what's done well (clean boundaries, good naming, sensible design choices). The rubric rewards a balanced read, not just defect-hunting.
   - **Weaknesses, risks, gaps** — classified by axis (below), each independently verified.

## Review Axes

Every finding maps to one axis (carried into the report as `Axis:`):

- **Security:** SQL/command injection, missing input validation, hardcoded secrets, data exposure, untrusted external input used without validation.
- **Performance:** N+1 queries, unbounded loops/fetches, missing pagination on list endpoints, large objects in hot paths, sync work that should be async.
- **Correctness:** spec mismatch, unhandled edge cases (null/empty/boundary), error-path gaps, race conditions, off-by-one, tests that don't actually test behavior.
- **Architecture:** breaks the layered structure, wrong dependency direction, duplication that should be shared, wrong abstraction level (over-engineered or too coupled).
- **Maintainability:** unclear/generic naming, deep nesting or clever tricks, dead code, unnecessary length, missing intent comments.

Avoid style-only comments unless they create real maintenance risk.

## Synthesis & Output

- Sort findings by severity: Critical, High, Medium, Low. Assign stable IDs `RV-1`, `RV-2`, …
- For each finding include `Axis:`.

````markdown
## Overall Understanding
<2-4 sentences: what the solution does and how it's structured — proves you grasp the whole.>

## Strengths
- <What's done well, with file references.>

## Critical Issues

1. **[RV-1] SQL Injection Vulnerability**
   - **ID:** RV-1
   - **Axis:** Security
   - **Location:** `src/main/java/.../UserRepository.java:42`
   - **Problem:** User input is concatenated directly into a query.
   - **Impact:** Attackers can execute arbitrary SQL.
   - **Verified:** Confirmed by reading the method — input reaches the query unbound.
   - **Fix:** Use a parameterized query / bound parameter.
   ```java
   @Query("SELECT u FROM User u WHERE u.id = :id")
   Optional<User> findById(@Param("id") Long id);
   ```

## High Priority
<Findings or `No high-priority findings.`>

## Medium Priority
<Findings or `No medium-priority findings.`>

## Low Priority
<Findings or `No low-priority findings.`>

## Recommendations
- <Highest-leverage improvements, ordered by impact.>
````

## Honesty in Review

- **Validate, don't rubber-stamp.** Every finding gets checked against the code before it makes the report.
- **Balance the read.** Name real strengths and real problems — both demonstrate understanding.
- **Quantify when possible.** "This N+1 adds ~50ms per list item" beats "this could be slow."
- **Back feedback with judgment.** Explain *why* something is a risk, not just *that* it exists.

## Red Flags (in your own review process)

- Reporting a finding without opening the code to confirm it.
- Reporting only defects with no statement of what the solution does well.
- Drilling into details before you can describe the whole solution.
- Findings without a severity or axis label.
