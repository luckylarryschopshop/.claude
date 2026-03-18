# .claude — Personal Claude Code Operating System
#
# Clone directly to ~/.claude — Claude Code reads this directory natively.
# No symlinks. No setup scripts. Just clone and go.

## Repository Structure

```
~/.claude/
  CLAUDE.md           ← global behaviours — inherited by all projects
  skills/             ← 8 global skills, loaded on demand per phase
    SKILL-workflow.md       — plan node, subagent strategy, session discipline
    SKILL-code-quality.md   — three-layer arch, OOP/FP rules, types, naming
    SKILL-tdd.md            — test-first discipline, test ID format, fixtures
    SKILL-git.md            — conventional commits, push cadence, pre-commit
    SKILL-logging.md        — human-readable logs, error format, sidecars
    SKILL-api-design.md     — OpenAPI, pagination, route standards
    SKILL-frontend.md       — charts, print layout, responsive UI
    SKILL-security.md       — encryption, PII policy, secrets, gitignore
  agents/             ← 15 specialised agents, loaded on demand
    AGENT-tester.md         — mandatory post-phase validation gate
    AGENT-teacher.md        — retrospective and knowledge synthesis
    AGENT-architect.md      — system design and ADRs
    AGENT-backend.md        — API and business logic implementation
    AGENT-frontend.md       — browser and mobile UI implementation
    AGENT-database.md       — schema, migrations, query optimisation
    AGENT-devops.md         — CI/CD, containers, cloud infrastructure
    AGENT-sysadmin.md       — servers, OS, networking, bare-metal
    AGENT-security.md       — audits, threat modelling, vulnerabilities
    AGENT-designer.md       — visual design, design systems, components
    AGENT-ux.md             — user flows, interaction specs, usability
    AGENT-pm.md             — requirements, scope, user stories
    AGENT-analyst.md        — data analysis, reporting, requirements
    AGENT-finance.md        — financial modelling, unit economics
    AGENT-trader.md         — trading strategies, backtesting, risk
  agent-skills/       ← domain-specific skills for agents
    SKILL-system-design.md        — architecture methodology (Architect)
    SKILL-financial-modelling.md  — financial model construction (Finance)
    SKILL-trading-strategy.md     — quant trading methodology (Trader)
    SKILL-security-audit.md       — STRIDE, OWASP, CVE scanning (Security)
    SKILL-infrastructure.md       — Linux hardening, runbooks (SysAdmin)
    SKILL-devops.md               — CI/CD, Docker, IaC, observability (DevOps)
    SKILL-database.md             — schema design, indexing, migrations (Database)
    SKILL-visual-design.md        — design systems, colour, typography (Designer)
    SKILL-ux-research.md          — JTBD, journey maps, heuristics (UX)
    SKILL-requirements.md         — user stories, acceptance criteria (PM/Analyst)
    SKILL-project-mgmt.md         — phase planning, risk management (PM)
    SKILL-mobile.md               — React Native, platform conventions (Frontend)
    SKILL-game-design.md          — game feel, progression, HUD (Designer/UX)
  agent-memory/       ← persistent agent knowledge (gitignored: finance/, trader/)
    [agent]/memory.md   — standing rules and accumulated knowledge
    [agent]/lessons.md  — correction log (same format as tasks/lessons.md)
  frameworks/         ← opinionated agent+phase tables by project type
    FRAMEWORK-webapp.md
    FRAMEWORK-mobile.md
    FRAMEWORK-game.md
    FRAMEWORK-cli.md
    FRAMEWORK-api-service.md
    FRAMEWORK-finance-tool.md
    FRAMEWORK-data-pipeline.md
  protocols/          ← multi-agent coordination rules
    PROTOCOL-handoff.md       — agent-to-agent handoff format and rules
    PROTOCOL-collaboration.md — session orchestration, Tester gate, Teacher invocation
  templates/
    PROJECT-CLAUDE.md       — starter template for new projects (includes Agents section)
    AGENT-template.md       — blank agent authoring template
    HANDOFF-agent.md        — agent-to-agent handoff file template
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

# Create the tasks, skills, and agent-notes directories
mkdir -p skills tasks agent-notes

# Edit CLAUDE.md — fill in project name, phases, skill table, and agents table
# Choose a framework template for your project type and copy its phase/agent table
```

## Using Agents

Agents are loaded exactly like skills — read the file when starting a phase.

```markdown
# In project CLAUDE.md, declare active agents:
## Agents — Active This Project
| Agent | Load when | Phase |
|---|---|---|
| ~/.claude/agents/AGENT-architect.md | System design | 1 |
| ~/.claude/agents/AGENT-backend.md   | API implementation | 2–3 |
| ~/.claude/agents/AGENT-tester.md    | After every phase | All |
```

The Tester agent is mandatory after every phase. The collaboration protocol
(`~/.claude/protocols/PROTOCOL-collaboration.md`) governs agent sequencing.

## Adding a New Agent

```bash
# Copy the template
cp ~/.claude/templates/AGENT-template.md ~/.claude/agents/AGENT-myagent.md

# Create memory scaffold
mkdir -p ~/.claude/agent-memory/myagent
touch ~/.claude/agent-memory/myagent/memory.md
touch ~/.claude/agent-memory/myagent/lessons.md

# Add to the Agents table in CLAUDE.md
```

## Adding a New Global Skill

```bash
cd ~/.claude

# Create the skill file
cp /dev/null skills/SKILL-myskill.md
# Edit the file with the skill content

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
