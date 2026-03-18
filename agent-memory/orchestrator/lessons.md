---
type: agent-lessons
agent: orchestrator
description: Correction log for routing mistakes.
---

# Orchestrator — Lessons

## 2026-03-18 — Initial design required manual agent declaration

**Rule:** Never design an agent system where users must declare or invoke agents manually.
Auto-select from task context at all times.

**Why:** Built a framework requiring `"load architect agent"` commands and per-project
agent declaration tables. User immediately identified this as friction that would cause
missed steps. The whole value of a multi-agent system is invisible expertise, not a
menu the user navigates.

**How to apply:** When designing any invocation pattern, ask: "Would a user who does not
know this system exists get the right agents automatically?" If no, it is not done.
