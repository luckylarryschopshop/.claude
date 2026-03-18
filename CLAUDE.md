# Global Claude Behaviour
# Applies to ALL projects. Project CLAUDE.md overrides specific sections.
# Location: ~/.claude/CLAUDE.md

---

## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write detailed specs upfront to reduce ambiguity — do not start building until the plan is clear
- If something goes sideways: STOP and re-plan immediately — do not keep pushing
- Use plan mode for verification steps, not just building
- For tasks with clear unknowns, surface them in the plan before any code is written

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution
- Subagent results are summarised back into main context — never dump raw output

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake recurring
- Ruthlessly iterate on these lessons until mistake rate drops
- Review `tasks/lessons.md` at session start for every project that has one
- Lessons are persistent across sessions — treat them as standing instructions

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behaviour between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
- A green test suite is necessary but not sufficient — manually verify behaviour too

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — do not over-engineer
- Challenge your own work before presenting it
- Elegance means clear, maintainable, and correct — not clever

### 6. Autonomous Bug Fixing
- When given a bug report: fix it — do not ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how
- If a fix requires a decision, make the reasonable one, document it, continue

---

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items before any implementation
2. **Verify Plan**: Check in before starting implementation on non-trivial work
3. **Track Progress**: Mark items complete as you go — `[x]` not `[ ]`
4. **Explain Changes**: High-level summary at each meaningful step
5. **Document Results**: Add review section to `tasks/todo.md` on completion
6. **Capture Lessons**: Update `tasks/lessons.md` after any correction from user

---

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what is necessary. Avoid introducing bugs.
- **Honest Assessment**: If something is wrong or unclear, say so — do not paper over it
- **Reversibility**: Prefer changes that are easy to undo. Commit before risky operations.

---

## Hard Stops (Universal)

Stop building and notify the user when:

1. **Usage limit approaching**
   → Complete current atomic unit of work (current function or test — not the whole phase)
   → Full handoff: update project CLAUDE.md + HANDOFF.md
   → Commit and push all work
   → Notify: "SESSION LIMIT REACHED. [Phase/task]. Committed [hash]. Resume by re-reading CLAUDE.md."

2. **Genuine blocker**
   A question where the answer is not in any loaded file AND proceeding would
   invent a decision affecting data model, user-facing behaviour, or security.
   → Stop immediately. Do not guess.
   → Write the question clearly under "BLOCKING QUESTION" in HANDOFF.md
   → Commit current state
   → Notify: "BLOCKING QUESTION — see HANDOFF.md"

   NOT blockers (decide, document, continue):
   HTTP status codes, poll intervals, sort orders, naming conventions,
   log verbosity defaults, any purely internal implementation detail.

---

## Skills Available (Global)

Load the relevant skill file at the start of the task that needs it.
Do not load all skills at once — load only what the current task requires.

| Skill | Load when |
|---|---|
| `~/.claude/skills/SKILL-workflow.md` | Planning a multi-phase build or complex task |
| `~/.claude/skills/SKILL-code-quality.md` | Writing any service, domain function, or non-trivial code |
| `~/.claude/skills/SKILL-tdd.md` | Writing tests or implementing against a test spec |
| `~/.claude/skills/SKILL-git.md` | Committing, branching, or setting up a repository |
| `~/.claude/skills/SKILL-logging.md` | Adding logging, error handling, or failure reporting |
| `~/.claude/skills/SKILL-api-design.md` | Writing API routes, OpenAPI spec, or pagination |
| `~/.claude/skills/SKILL-frontend.md` | Building any browser UI, charts, or print layout |
| `~/.claude/skills/SKILL-security.md` | Encryption, secrets, gitignore, PII handling |

Project-specific skills override global skills where they conflict.
Project CLAUDE.md lists which project skills exist and when to load them.

---

## Auto-Orchestration (Always Active)

**On every task, classify first — do not start work until classification is done.**

| Task signals | Type | Action |
|---|---|---|
| "explain", "what is", "how does", "teach me", "why does" | question | Answer directly — no agents |
| error message, stack trace, "not working", "broken", "failing", "crash" | bug-fix | Load single domain agent (see map below) → Tester after |
| "review", "audit", "check", "assess", "analyse this" | review | Load Security → Tester |
| "build", "create", "new project", "scaffold", "from scratch" | new-project | Read `~/.claude/agents/AGENT-orchestrator.md` |
| "add", "implement", "feature", "extend", "integrate" | add-feature | Read `~/.claude/agents/AGENT-orchestrator.md` |
| ambiguous | — | Treat as add-feature; read orchestrator |

**Bug-fix domain map** — load the ONE most relevant agent:
- API / server / endpoint / auth error → backend
- UI / component / browser / CSS → frontend
- Query / migration / constraint / ORM → database
- Deploy / CI / container / k8s → devops
- Server / OS / networking → sysadmin
- Multi-domain or unclear → read orchestrator instead

**Tester is mandatory after every phase regardless of task type. No exceptions.**

Do NOT ask the user to declare agents or load files manually.
The orchestrator and this classifier replace per-project agent declarations entirely.

---

## Agents Available

15 agents in `~/.claude/agents/`. Selected automatically by the orchestrator — do not load manually unless overriding.
Full roster and skill inheritance chain: see `~/.claude/agents/AGENT-orchestrator.md`.

### Skill Inheritance Chain (highest → lowest priority)
```
./skills/SKILL-X.md               ← project-level override
~/.claude/agent-skills/SKILL-X.md ← agent-domain skill
~/.claude/skills/SKILL-X.md       ← global baseline
```

### Project Framework Templates
For opinionated agent+phase tables by project type:
`~/.claude/frameworks/FRAMEWORK-{webapp,mobile,game,cli,api-service,finance-tool,data-pipeline}.md`
