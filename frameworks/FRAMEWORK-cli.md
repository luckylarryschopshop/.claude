---
name: cli
description: CLI tool framework — agents, phases, and skill loading for command-line applications.
---

# Framework: CLI Tool

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM | 1 | Define commands, flags, and user journeys |
| Architect | 1–2 | Command structure, plugin model, config |
| Backend | 2–3 | Core implementation |
| Security | 3 | Credential handling, secrets, permissions |
| DevOps | 3–4 | Cross-platform build, package registry publishing |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — Design
Goal: Command interface defined; user journeys documented.
Agents: PM → Architect
Skills: SKILL-requirements, SKILL-system-design
Key design decisions:
- Command hierarchy: `tool <command> <subcommand> [flags]`
- Configuration: file (YAML/TOML/JSON) vs env vars vs flags (precedence order)
- Output format: human-readable vs `--json` flag vs both
- Authentication: if needed, what mechanism?
- Plugin/extension system: yes/no

### Phase 2 — Core Implementation
Goal: Happy path commands working; tested.
Agents: Backend → Tester
Skills: SKILL-code-quality, SKILL-tdd, SKILL-logging
Standards:
- Exit codes: 0 = success, 1 = general error, 2 = misuse (invalid flags)
- stderr for errors and warnings; stdout for output (allows piping)
- `--json` flag for all output-producing commands (machine-readable)
- `--quiet` flag to suppress non-essential output

### Phase 3 — Complete, Secure, and Polished
Goal: All commands complete; error messages useful; security reviewed.
Agents: Backend (remaining) → Security → Tester
Skills: SKILL-security, SKILL-logging
Security checklist for CLIs:
- No credentials in shell history (use env vars or prompt, never flags for secrets)
- No credentials in log output
- Config files with secrets must have 0600 permissions
- Warn if config file is world-readable

### Phase 4 — Distribution
Goal: Cross-platform binaries; published to package registry.
Agents: DevOps → Tester
Skills: SKILL-devops
Distribution options: `npm`, `pip`, `brew`, `cargo`, `apt`, GitHub Releases binary

## CLI UX Standards
- `--help` available on every command and subcommand
- `--version` at top level
- Colour output with `NO_COLOR` env var support (POSIX standard)
- Progress indicators for operations > 1 second
- Confirmation prompts for destructive operations (with `--yes`/`--force` bypass flag)
- Errors include: what went wrong + how to fix it + docs URL if available

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-system-design |
| 2 | SKILL-code-quality, SKILL-tdd, SKILL-logging |
| 3 | SKILL-security |
| 4 | SKILL-devops |
