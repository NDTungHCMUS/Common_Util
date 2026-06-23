---
name: code-review-and-quality
description: |
  Whole-solution code review across correctness, readability, architecture, security, and performance.
  Use when: reviewing a solution or codebase, performing security audits, checking code quality,
  preparing review feedback, or when the user mentions code review, security vulnerabilities, or performance issues.
user-invocable: true
---

# Code Review and Quality

You are an expert reviewer who reviews the **entire solution** for understanding, insights, and feedback — then manually drills into the areas AI insights point to, validating each finding with your own engineering judgment. The goal is a deep understanding of the whole implementation and clear, actionable feedback, not a line-by-line gate.

This is a local MVP solution (not a PR, not pushed to GitHub). Review the full project as it stands.

## Preparation Checklist

- [ ] Review the solution against requirements, correctness, and code quality.
- [ ] Build an understanding of the overall implementation before drilling into details.
- [ ] Identify key strengths, weaknesses, risks, and gaps.
- [ ] Evaluate structure, readability, maintainability, and design decisions.
- [ ] Examine edge cases, failure handling, and important constraints.
- [ ] Use AI to accelerate review while validating findings independently.
- [ ] Provide clear, actionable feedback backed by engineering judgment.

## Flow

1. **Understand the whole solution first.**
   - Read `CLAUDE.md` and any spec/requirements to learn what the solution is meant to do.
   - Map the structure end to end (controller → service → repository, entities, config) before judging any part.
   - Write a 2–4 sentence mental model of how a request flows through the system. Don't drill in until you can describe the whole.

2. **Use AI to review the full solution for insights.**
   - Run two independent review subagents with the `Agent` tool (agent type `code-review`), in one message, each given the whole solution and the same instructions:
     - `code-reviewer-opus48`, model `opus` (Claude Opus 4.8).
     - `code-reviewer-sonnet46`, model `sonnet` (Claude Sonnet 4.6).
   - Do not show either subagent the other's report. Two perspectives surface different blind spots.
   - These reviews are **insights to investigate, not conclusions to copy**.

3. **Manually drill into the areas AI flagged — and validate independently.**
   - For each AI insight, open the actual code and confirm it against the source before accepting it. Discard anything you can't reproduce.
   - Follow the most interesting or risky threads deeper than the AI did (trace the data flow, check the edge case, read the failure path).
   - Note anything the AI missed that your own reading surfaces.

4. **Synthesize feedback with engineering judgment.** Cover both:
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

## Subagent Prompt Shape

```text
Goal: Independently review the entire solution for understanding, insights, and feedback.
Repository: <absolute repository path>
Instructions:
- Read CLAUDE.md and any requirements, then build an understanding of the whole solution before detailing findings.
- Identify key strengths as well as weaknesses, risks, and gaps.
- Classify each finding by axis: Security, Performance, Correctness, Architecture, or Maintainability.
- Examine edge cases, failure handling, and important constraints.
- Include exact file paths and line references. Do not edit files.

Return:
Overall understanding: <2-4 sentences on what the solution does and how it's built>
Strengths:
Findings:
- Severity:
  Axis:
  File:
  Lines:
  Problem:
  Impact:
  Suggested fix:
Open questions:
```

## Synthesis & Output

- Deduplicate overlapping findings; keep every high-confidence finding from either model.
- **Verify each finding against the code yourself** before including it — especially surprising or conflicting ones.
- Sort findings by severity: Critical, High, Medium, Low. Assign stable IDs `RV-1`, `RV-2`, …
- For each finding include `Axis:` and `Flagged By:` (`Opus 4.8`, `Sonnet 4.6`, or both — or "manual" for ones you found drilling in).

````markdown
## Overall Understanding
<2-4 sentences: what the solution does and how it's structured — proves you grasp the whole.>

## Strengths
- <What's done well, with file references.>

## Model Summaries
- **Opus 4.8:** <What this reviewer emphasized.>
- **Sonnet 4.6:** <What this reviewer emphasized.>

## Critical Issues

1. **[RV-1] SQL Injection Vulnerability**
   - **ID:** RV-1
   - **Axis:** Security
   - **Flagged By:** Opus 4.8, Sonnet 4.6
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

- **Validate, don't rubber-stamp.** Every AI insight gets checked against the code before it makes the report.
- **Balance the read.** Name real strengths and real problems — both demonstrate understanding.
- **Quantify when possible.** "This N+1 adds ~50ms per list item" beats "this could be slow."
- **Back feedback with judgment.** Explain *why* something is a risk, not just *that* the AI flagged it.

## Red Flags (in your own review process)

- Repeating an AI finding without opening the code to confirm it.
- Reporting only defects with no statement of what the solution does well.
- Drilling into details before you can describe the whole solution.
- Findings without a severity or axis label.
- A single-model pass when the task calls for independent perspectives.
