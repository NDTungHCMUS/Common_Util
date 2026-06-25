---
name: clarify-and-research
description: Turn a large-scope requirement into a clarify.md — open questions plus researched solution/data-structure options with trade-offs and a recommendation, for the user to answer inline before writing the spec. Use when a requirement is broad or ambiguous and you need to nail down decisions and compare approaches before /spec.
---

# Clarify & Research

Turn a raw, large-scope requirement into a single `clarify.md` that the user
answers inline before feeding it into `/spec`. Do **not** write a spec, plan, or
any code — this skill stops at `clarify.md`.

## Produce two things

1. **Open questions** — every ambiguity worth settling before implementation,
   numbered so they can be answered one at a time.
2. **Solution options** — the data-structure / algorithm / architecture choices
   that could satisfy the requirement, each with honest Pros/Cons, ending in a
   **Recommendation**.

## How to work

1. **Think first.** Restate the requirement. State assumptions explicitly. Surface
   multiple interpretations — never pick silently. Keep to the project's MVP /
   simplicity ethos (CLAUDE.md).
2. **Research before asking.** Use `Grep`/`Glob`/`Read` on this repo, and the
   `research` agent or `WebSearch`/`WebFetch` for external standards, library
   behavior, and data-structure trade-offs. Ground every option in something real.
3. **Be exhaustive on questions, ruthless on relevance.** Cover core-operation
   semantics, durability/persistence, concurrency scope, failure/edge behavior,
   ordering/tie-breaks, data shape & cardinality, boundary states, exactly-once vs
   at-least-once, scaling. Drop anything the requirement already settles.
4. **Each option** gets a one-line description, **Pros**, **Cons**, and fit. Always
   end with a **Recommendation** + one-sentence why.

## Output — write to `clarify.md` in the project root

```markdown
# <Feature> — Clarifications & Design Options

A design conversation, not yet code. Below is what should be nailed down before
implementing, plus the data-structure/algorithm options with their trade-offs.

## Problem statement
<restated requirement + explicit requirements as bullets>

---

## Things to clarify first

### 1. <short title>
<the ambiguity, 2–3 plausible interpretations laid out>
<**Recommendation:** line where there's a sensible default>

<!-- user answers here -->

### 2. <short title>
...

---

## Data structure / algorithm options

Assuming the core need: <one-line restatement of the technical crux>.

### Option A — <name>
<one-line description>
- **Pros**: ...
- **Cons**: ...

### Option B — <name>
...

**Recommendation:** Option <X> — <one sentence why, tied to the constraints>.

<!-- user picks here -->
```

After writing the file, tell the user it's ready to answer and that `/spec` is the
next step once filled in.
