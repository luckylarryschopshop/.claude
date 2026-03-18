---
name: game
description: Game development framework — agents, phases, and skill loading for game projects (Unity, Godot, web games).
---

# Framework: Game Development

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM | 1 | Scope and feature list (GDD) |
| Designer | 1–2 | Visual style, HUD, UI/menus |
| UX | 1–2 | Tutorial, progression, game feel |
| Architect | 2 | Technical architecture, engine decisions |
| Backend | 3+ | Multiplayer, leaderboards, persistence (if needed) |
| Frontend | 3–4 | Game client implementation |
| Audio | optional | Sound design direction |
| DevOps | 4–5 | Build pipeline, platform submissions |
| Tester | All | After every milestone — mandatory gate |

## Phase Structure

### Phase 1 — Game Design Document (GDD)
Goal: Core loop, mechanics, and scope defined and committed.
Agents: PM → Designer + UX (parallel)
Skills: SKILL-requirements, SKILL-game-design, SKILL-ux-research
Key outputs:
- Core loop: what does the player do every 30 seconds?
- Win/fail condition
- Scope: how many levels/modes/enemies for v1?
- Art style reference (visual comps or reference images described)
- Platform targets

### Phase 2 — Prototype (Vertical Slice)
Goal: Core mechanic feels good on real hardware; art direction proven.
Agents: Frontend (game dev) + Designer — parallel
Skills: SKILL-game-design, SKILL-visual-design, SKILL-frontend, SKILL-code-quality
Done when: Tester plays a complete (short) loop and approves game feel

### Phase 3 — Content Build
Goal: All planned levels/content implemented with placeholder or final art.
Agents: Frontend (game dev)
Skills: SKILL-game-design, SKILL-code-quality, SKILL-tdd
Done when: Tester completes every planned level/mode; TESTER APPROVED

### Phase 4 — Polish and Juice
Goal: Game feel is excellent; tutorial complete; edge cases handled.
Agents: Frontend + Designer (parallel) → UX (tutorial review)
Skills: SKILL-game-design, SKILL-visual-design, SKILL-ux-research
Done when: Juice checklist complete; tutorial IKEA test passed; TESTER APPROVED

### Phase 5 — Backend (if multiplayer/persistent)
Goal: Leaderboards, save sync, multiplayer working.
Agents: Backend → Architect (review) → Security → Tester
Skills: SKILL-api-design, SKILL-security, SKILL-code-quality

### Phase 6 — Build and Ship
Goal: Automated build pipeline; submitted to platform stores.
Agents: DevOps → Tester
Skills: SKILL-devops
Platforms: Steam, itch.io, App Store, Play Store, Web (itch.io/Newgrounds)

## Game-Specific Considerations
- **Save system**: design in Phase 1; hardest to retrofit; decide: local, cloud, or both
- **Input handling**: decide in Phase 2; support controller, keyboard/mouse, touch as appropriate
- **Performance**: 60fps is a hard requirement; profile from Phase 2, not after
- **Playtesting**: real players in Phase 2 (prototype) and Phase 4 (polish); designer's intuition is not enough
- **Localisation**: if targeting multiple languages, instrument strings from Phase 3

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-game-design |
| 2 | SKILL-game-design, SKILL-visual-design, SKILL-frontend |
| 3–4 | SKILL-game-design, SKILL-code-quality, SKILL-tdd |
| 5 | SKILL-api-design, SKILL-security (if backend) |
| 6 | SKILL-devops |
