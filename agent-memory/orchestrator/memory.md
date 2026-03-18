---
type: agent-memory
agent: orchestrator
description: Standing rules for agent selection patterns and routing decisions.
---

# Orchestrator — Memory

## Core Design Principle
An agent system's quality is measured by how little the user thinks about which agents
to invoke. If the user must name agents or declare them per-project, the system is not
finished. Manual declaration is a design smell — fix it with automation.

## Architecture Standing Rules
- The classifier (CLAUDE.md, always loaded) handles simple cases inline: bug-fix = single domain agent, review = security + tester, question = no agents.
- The orchestrator (loaded on demand) handles only new-project and add-feature. Reads the agent capability index to select without loading all 15 files.
- These two layers must stay separate: classifier stays lightweight (~150 tokens), orchestrator can be thorough (~800 tokens).
- Manual agent declaration in project CLAUDE.md is an override mechanism only — never the default path.

## Agent Selection Patterns
[Append as patterns emerge across projects]

## Project Type Recognition
[Append as reliable signals are discovered]
