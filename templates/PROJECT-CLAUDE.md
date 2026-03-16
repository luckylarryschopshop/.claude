# [Project Name] — Project CLAUDE.md
# Inherits: ~/.claude/CLAUDE.md (global behaviours)
# Repo: https://github.com/[org]/[project]
# Overrides: [list any global behaviour overrides, or "none"]

---

## Current State

Phase: 1 of [N]
Last session: [date]
Status: Not started

---

## Completed Phases

- [ ] Phase 1: [description]
- [ ] Phase 2: [description]
- [ ] Phase N: [description]

---

## Active Phase

**Phase 1 — [Name]**

Goal: [one sentence]

Tasks:
- [ ] [task]
- [ ] [task]

Session ends when: [specific, verifiable condition]

---

## Skills — Load Per Phase

Global skills: `~/.claude/skills/SKILL-[name].md`
Project skills: `./skills/SKILL-[name].md` (override global if same name)

| Phase | Load these skills |
|---|---|
| 1 | SKILL-git, SKILL-security, SKILL-code-quality |
| 2 | SKILL-tdd, SKILL-code-quality, SKILL-logging |
| N | [skills for phase N] |

Project-specific skills (if any):
| Skill | Contents | Load in phases |
|---|---|---|
| `./skills/SKILL-[name].md` | [what it covers] | [phases] |

---

## Architectural Decisions

[Updated each session — append only]

| Date | Decision | Rationale |
|---|---|---|
| - | [decision] | [why] |

---

## Known Issues

[None yet]

---

## Type Checker Status

[Updated each session — e.g. "mypy --strict domain/: 0 errors"]

---

## Do Not Touch

[List files/directories that should never be modified by Claude Code]

---

## Overrides to Global CLAUDE.md

[List any overrides, or "None — global defaults apply in full"]

Examples of valid overrides:
- "Do not use subagents — single context only (this project is small)"
- "Default log level is DEBUG for this project"
- "Commit frequency: every feature, not every function (faster iteration preferred)"
