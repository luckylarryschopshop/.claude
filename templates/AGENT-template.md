---
name: [agent-name]
role: [Human-readable role title]
description: >
  [One-paragraph description: what this agent does and when to invoke it.
  Include explicit trigger conditions.]
skills:
  global: [SKILL-name1, SKILL-name2]   # from ~/.claude/skills/
  agent: [SKILL-agent-name]            # from ~/.claude/agent-skills/
memory: ~/.claude/agent-memory/[agent-name]/
min_model_tier: large | medium | small
collaboration:
  hands-off-to: [agent1, agent2]
  receives-from: [agent3, agent4]
---

# [Role Title] Agent

## Identity
[One paragraph: who you are, what you care about, your core philosophy.
Make this specific enough that the agent has a distinct perspective.
Avoid generic "I am helpful and thorough" — describe the specific values this agent holds.]

## Scope
IN SCOPE:
- [Explicit responsibility]
- [Explicit responsibility]
- [Explicit responsibility]

OUT OF SCOPE:
- [Explicit exclusion — what this agent does NOT do and why]
- [Explicit exclusion — which agent takes over]
- [This list prevents agent drift — be specific]

## Default Approach
[Numbered steps — the agent's default method for tackling its domain.
Each step should be concrete, not abstract. Include specific commands, tools, or formats where known.
Typical: 5–8 steps.]

1. [First step — often: read requirements/handoff before anything else]
2. [Step]
3. [Step]
...

## [Key Standards / Format Section]
[Include the primary output format this agent produces — the thing it hands off.
Use a code block showing the exact format with placeholder values.
This is what the receiving agent will read — make it unambiguous.]

## Memory Protocol
On session start: read memory.md + lessons.md + ./agent-notes/[agent-name].md
On session end: write [what goes to memory.md] to memory.md, [what goes to lessons.md] to lessons.md

## Handoff Template
When handing off to [Agent1]:
→ Provide: [specific outputs — files, docs, specs]
→ State: [what is final vs what has latitude]
→ Flag: [constraints the next agent must honour]

When handing off to [Agent2]:
→ Provide: [...]
→ State: [...]
→ Flag: [...]

---
<!-- AUTHORING NOTES (delete before saving the agent file)

Target size: 400–700 tokens for the full agent file.
Use the AGENT-*.md files in ~/.claude/agents/ as reference implementations.

Checklist before saving:
- [ ] Identity paragraph has a specific philosophical stance, not generic helpfulness
- [ ] OUT OF SCOPE list prevents the 3 most likely drift scenarios for this role
- [ ] Default Approach starts with "read requirements/handoff" or equivalent orientation step
- [ ] Output format is fully specified with no ambiguous fields
- [ ] Memory Protocol says what TYPE of information goes where
- [ ] Handoff Template has at least one → Flag entry (what must NOT be ignored)
- [ ] File size is under 700 tokens (check: wc -w [file] × 1.3 ≈ tokens)
-->
