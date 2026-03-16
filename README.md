# .claude — Personal Claude Code Operating System
#
# Clone directly to ~/.claude — Claude Code reads this directory natively.
# No symlinks. No setup scripts. Just clone and go.

## Repository Structure

```
~/.claude/
  CLAUDE.md           ← global behaviours — inherited by all projects
  skills/
    SKILL-workflow.md       — plan node, subagent strategy, session discipline
    SKILL-code-quality.md   — three-layer arch, OOP/FP rules, types, naming
    SKILL-tdd.md            — test-first discipline, test ID format, fixtures
    SKILL-git.md            — conventional commits, push cadence, pre-commit
    SKILL-logging.md        — human-readable logs, error format, sidecars
    SKILL-api-design.md     — OpenAPI, pagination, route standards
    SKILL-frontend.md       — charts, print layout, responsive UI
    SKILL-security.md       — encryption, PII policy, secrets, gitignore
  templates/
    PROJECT-CLAUDE.md       — starter template for new projects
```

## Installation

```bash
# Clone directly to ~/.claude — Claude Code reads this natively
git clone https://github.com/[your-username]/.claude ~/.claude

# Verify Claude Code picks it up
cat ~/.claude/CLAUDE.md
```

That's it. No symlinks. No configuration. Claude Code reads `~/.claude/CLAUDE.md`
automatically at session start and inherits all global behaviours.

## Updating

```bash
cd ~/.claude
git pull
# Changes apply immediately to all projects
```

## How Projects Inherit This

Claude Code reads `~/.claude/CLAUDE.md` first, then the project's `./CLAUDE.md`.
The project CLAUDE.md specifies which skills to load per phase — Claude loads
only those files, keeping context lean.

Project skill files (`./skills/SKILL-*.md`) override global skills where names match.

## Setting Up a New Project

```bash
cd ~/your-project

# Copy the project CLAUDE.md template
cp ~/.claude/templates/PROJECT-CLAUDE.md ./CLAUDE.md

# Create the tasks and skills directories
mkdir -p skills tasks

# Edit CLAUDE.md — fill in project name, phases, and skill-per-phase table
```

## Adding a New Global Skill

```bash
cd ~/.claude

# Create the skill file
cat > skills/SKILL-myskill.md << 'SKILL'
---
name: myskill
description: When to load this and what it covers.
---
# My Skill
...
SKILL

# Add it to the skills table in CLAUDE.md
git add skills/SKILL-myskill.md CLAUDE.md
git commit -m "feat: add SKILL-myskill"
git push
```

## Token Efficiency

Skills are loaded on demand — never all at once.
Each phase in a project's CLAUDE.md lists which skills to load.

| What | Approx tokens | When loaded |
|---|---|---|
| Global CLAUDE.md | ~500 | Every session start |
| Project CLAUDE.md | ~300–500 | Every session start |
| Each skill file | ~300–600 | Once per phase, on demand |

Compare to a monolithic prompt: thousands of tokens loaded every turn.

## Skill Override Precedence

```
Project ./skills/SKILL-X.md   ← highest priority
~/.claude/skills/SKILL-X.md   ← global fallback
```

If a project needs different behaviour than the global default for a skill,
create `./skills/SKILL-X.md` in the project — it shadows the global version.
