---
type: agent-memory
agent: teacher
description: Standing rules, preferences, and accumulated knowledge for the teacher agent across all projects.
---

# teacher Agent — Memory

## Retrospective Process Notes

**When agent lessons.md files are all empty** (first session): the lessons come from
the conversation itself — observe what was built, what was corrected, and what failed.
Do not skip the retrospective because there is no prior history. First sessions often
produce the most important structural lessons.

**Lesson source priority:** User corrections > build failures > plan deviations > near-misses.
A user correction that came immediately after seeing the output (not after using it) is
the highest-signal lesson — it means the design had a fundamental flaw, not a runtime bug.

**When to skip WebSearch:** If the lesson is about a process failure or a design flaw
discovered in the current session, WebSearch is unnecessary — the fix is already in the
code. Reserve WebSearch for domain knowledge gaps (e.g. "what are best practices for
database migration strategies?"), not for session-internal learnings.

## Standing Rules
[Rules derived from retrospective corrections — append as confirmed]
