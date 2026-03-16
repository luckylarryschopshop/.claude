---
name: workflow
description: Multi-phase build orchestration, session discipline, context loop,
  handoff protocol, and phase-based task management. Load this skill at the start
  of any multi-phase project or when resuming a build session.
---

# Workflow Skill

## Context Loop (Run Every Session Start)

Before any action in a new session:

  STEP 1 — Re-read the project CLAUDE.md in full
  STEP 2 — Re-read HANDOFF.md (what was last built, what is next, gotchas)
            If HANDOFF.md does not exist (Phase 1, fresh project): skip this step.
            Note the absence in the status report — do not treat it as an error.
  STEP 3 — Read tasks/lessons.md (accumulated corrections — treat as standing rules)
  STEP 4 — Run: git log --oneline -10
  STEP 5 — Run the project's test suite (pytest / npm test / swift test)
  STEP 6 — Report status:
            "Phase X. Last commit: [msg]. Tests: X passed, Y failed.
             Lessons loaded: [N]. Ready to continue with: [next task]."
            Wait for confirmation before writing code.

Never skip or abbreviate this loop — stale assumptions cause silent regressions.

---

## Phase Management

Each phase is one session's work. Phases must not be combined.

### Starting a phase
1. Read the phase definition from project CLAUDE.md
2. Load only the skill files listed for this phase
3. Write tests FIRST (they will fail — that is expected)
4. Commit the test skeletons: `test(phaseN): add [phase] test skeletons`
5. Implement until tests pass
6. Commit after each passing test group

### Ending a phase (mandatory before stopping)
1. Run full test suite — note any failures in project CLAUDE.md
2. Run type checker (mypy / tsc / swiftc) on core logic — note failures
3. Update project CLAUDE.md:
   - Mark phase complete
   - Update "current state" section
   - Record any architectural decisions made
   - List any known issues
4. Write HANDOFF.md:
   - What was built this session
   - What is working and verified
   - What is next (exact first task of next phase)
   - Any gotchas or non-obvious decisions
   - Exact filenames changed
5. Commit: `feat(phaseN): [phase description] — all tests pass`
6. Push to origin

### Resuming after a break
Run the context loop above. Do not write a single line of code before completing it.

---

## tasks/ Directory

Every project should maintain:

```
tasks/
  todo.md       ← current phase plan with checkable items [ ] / [x]
  lessons.md    ← accumulated corrections from user feedback
  decisions.md  ← architectural decisions with rationale (append-only)
```

### todo.md format
```markdown
## Phase N — [Phase name]
Goal: [one sentence]

### Tasks
- [ ] Write test skeletons for FEAT-01 through FEAT-06
- [x] Implement domain/dedup.py — DEDUP-DOMAIN-01 through 04 pass
- [ ] Wire dedup service into pipeline

### Blockers
- None

### Review
[Filled in at phase end]
```

### lessons.md format
```markdown
## Lessons

### [Date] — [Brief description of mistake]
**What happened**: [description]
**Root cause**: [why it happened]
**Rule**: [standing instruction to prevent recurrence]
**Applies to**: [all projects | project-name only]
```

### decisions.md format
```markdown
## [Date] — [Decision title]
**Context**: [what problem this solves]
**Decision**: [what was chosen]
**Alternatives considered**: [what was rejected and why]
**Consequences**: [what this enables or constrains]
```

### HANDOFF.md format
```markdown
# Handoff — [Date] — Phase [N]

## What was built this session
[Bullet list of files created or meaningfully changed]
- `path/to/file.py` — [what it does]
- `tests/test_module.py` — [test IDs added: X-01 through X-06]

## What is working and verified
[Exactly what can be demonstrated to work right now]
- Tests passing: [list test IDs]
- Manual verification: [what was tested by hand]
- Type checker status: [clean / N errors — mypy, tsc, swiftc, etc.]

## What is next (first task of next phase)
[Single specific task — not a phase summary]
Example: "Write test skeletons for NORM-01 through NORM-06 in test_normaliser.py"

## Gotchas and non-obvious decisions
[Things that would trip up someone resuming cold]
- [gotcha 1]
- [gotcha 2]

## Exact files changed this session
[Complete list — used by context loop to know what to load next session]
- created: [file]
- modified: [file]
- deleted: [file]

## Blocking questions (if any)
[If session ended on a blocker, state it clearly here]
BLOCKING: [exact question]
Reason: [why this cannot be decided without user input]
```

---

## Memory Protocol

Use Claude's memory system to persist:
- Recurring user preferences (tone, formatting, tooling choices)
- Project-level patterns that recur across sessions
- User corrections that apply globally (not just to one project)

Do NOT store in memory:
- Sensitive data (passwords, API keys, transaction data)
- Anything that belongs in code or config files
- Temporary decisions specific to one task

Trigger memory update when:
- User corrects the same behaviour twice in the same project
- User explicitly states a preference ("always do X", "never do Y")
- A lesson in lessons.md applies to all projects, not just this one

---

## Subagent Usage Patterns

### When to spawn a subagent
- Research tasks that would pollute the main context (reading docs, exploring an API)
- Parallel independent work (writing tests for module A while implementing module B)
- Validation tasks (running a large test suite, checking all routes have operation_id)

### Subagent briefing template
```
Task: [single specific task — one task only]
Input: [exactly what to use as input]
Output: [exactly what to return — format specified]
Constraints: [what NOT to do, what NOT to load]
Success condition: [how to know it is done]
```

### Subagent result handling
- Summarise results back into main context — never dump raw subagent output
- If a subagent hits a blocker, surface it to the user — do not chain another subagent to guess
