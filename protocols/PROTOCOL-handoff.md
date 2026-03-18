---
name: handoff
description: Agent-to-agent handoff protocol — format, rules, and file naming conventions.
---

# Agent Handoff Protocol

## Purpose
Handoffs are the connective tissue between agents. A good handoff means the receiving agent can start immediately with full context. A bad handoff means the receiving agent re-does work, makes wrong assumptions, or asks clarifying questions that should have been answered before the handoff.

## When to Create a Handoff
Create a handoff file whenever:
- One agent's phase is complete and another agent takes over
- An agent needs to block until the receiving agent completes something
- A decision was made that the next agent must not re-open

## File Location and Naming
```
[project]/agent-notes/handoff-[from]-to-[to]-[YYYY-MM-DD].md
```

Examples:
```
agent-notes/handoff-architect-to-backend-2026-03-18.md
agent-notes/handoff-backend-to-tester-2026-03-18.md
agent-notes/handoff-pm-to-architect-2026-03-18.md
```

If multiple handoffs happen on the same date:
```
agent-notes/handoff-architect-to-backend-2026-03-18-v2.md
```

## Handoff File Format
See `templates/HANDOFF-agent.md` for the full template.

The four required sections:

### 1. FINAL (Do Not Reopen)
Decisions that are settled. The receiving agent must honour these.
If a final decision appears wrong, raise it as a new question — do not silently override.

### 2. OPEN (Your Latitude)
Areas where the sending agent intentionally left decisions to the receiving agent.
This is where the receiving agent should focus their expertise.

### 3. CONSTRAINTS
Technical, business, or regulatory constraints the receiving agent must operate within.
These are facts about the world, not preferences.

### 4. FIRST TASK
The single most important thing the receiving agent should do first.
One task only. Ordering matters.

## Handoff Rules
1. **The sending agent writes the handoff** — it is part of the sending agent's done criteria
2. **Handoff = commit** — the handoff file must be committed to the project repo
3. **No handoff without tester approval** — the Tester agent must approve before handoff
4. **One receiving agent per handoff file** — do not write multi-agent handoffs in one file
5. **Read before building** — the receiving agent must read all listed "Files to read first" before writing any code

## Handoff Receipt
When receiving a handoff:
1. Read the handoff file fully
2. Read all listed "Files to read first"
3. Confirm understanding by summarising the FINAL decisions and FIRST TASK in one paragraph before starting work
4. Note any CONSTRAINTS that require clarification — raise them immediately, before building

## Blocking on Handoff
If the handoff contains unanswered questions that would change how you build:
- Write the question to `HANDOFF.md` under BLOCKING QUESTION
- Do not proceed past the point where the answer matters
- Commit current state and notify the user
