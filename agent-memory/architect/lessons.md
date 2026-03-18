---
type: agent-lessons
agent: architect
description: Correction log for the architect agent — mistakes made and rules derived to prevent recurrence.
---

# architect Agent — Lessons

## Lesson Format
Each lesson: Rule / **Why:** reason / **How to apply:** when this triggers

---

## 2026-03-18 — Phase boundary exceeded without check-in

**Rule:** When a plan marks a session boundary ("Phase 1 this session"), stop at that
boundary, summarise what was built, and check in before continuing. Do not continue
through subsequent phases on momentum alone.

**Why:** Plan stated "Phase 1 this session" covering 5 agents + supporting files.
Implemented all 6 phases without stopping. Outcome was fine this time, but the value
of a phase gate is highest when things are going well — that is when course-correction
is cheapest. If the direction had been wrong, all 6 phases would need unwinding.

**How to apply:** After completing any phase that the plan designates as a session
boundary, write a phase summary and explicitly surface it. Do not silently continue.
The user can always say "keep going" — but they should be given that choice.
