---
name: game-design
description: Game design methodology — game feel, progression systems, tutorial design, HUD/UI. Load when acting as Designer or UX agent on game projects.
---

# Game Design Skill

## Core Methodology

### Game Feel Fundamentals
"Game feel" is the tactile sensation of controlling something in a game. It depends on:
- **Responsiveness**: input → visual feedback latency < 100ms feels instant; > 200ms feels sluggish
- **Feedback**: every action has a clear response (animation, sound, particles, screen shake)
- **Anticipation**: windup before a big action telegraphs it and builds tension
- **Follow-through**: animation continues slightly past the endpoint for weight
- **Squash and stretch**: exaggerates motion to imply mass and elasticity

Juice (feel improvements) checklist:
- [ ] Screen shake on impact
- [ ] Particle burst on collection/destruction
- [ ] Sound effect on every meaningful player action
- [ ] UI elements animate in, not just appear
- [ ] Camera eases, not cuts

### Progression System Design
Progression keeps players engaged. Types:
| Type | Mechanism | Risk |
|------|-----------|------|
| **Skill progression** | Player gets better | Needs good tutorial and difficulty curve |
| **Character progression** | Stats/abilities unlock | Risk: "pay to win" perception |
| **Content unlocks** | New areas/levels | Risk: players plateau at each gate |
| **Narrative progression** | Story advances | Risk: players who don't engage with story disengage |

**Difficulty curve**: start easy (teach), ramp (challenge), peak (test), valley (reward, breathe).
Design the valley deliberately — without recovery moments, players burn out.

### Tutorial Design Principles
1. **"Do, don't tell"** — let the player perform the action, not read about it
2. **Contextual hints** — show the hint when the mechanic is first needed, not before
3. **Skippable** — experienced players must not be forced through basics
4. **Failure is safe** — first introduction of a mechanic should have zero or low stakes
5. **One mechanic at a time** — never introduce two new mechanics simultaneously
6. **IKEA test** — can a player complete the tutorial without reading any text?

### HUD and UI Design Rules
HUD (Heads-Up Display):
- **Minimise at rest**: only show what player needs right now; fade or hide what's not urgent
- **Highlight on change**: flash/animate stats when they change (HP drops, ammo increases)
- **Consistent position**: critical info (HP, timer) must always be in the same screen position
- **Diegetic where possible**: information shown in the game world (ammo on gun, health in character animation) reduces UI clutter

Menus:
- Navigation must be completable by controller/keyboard (mobile navigation becomes PC navigation)
- Never more than 3 levels deep
- Settings must be accessible from pause menu in exactly 2 button presses
- Destructive actions (delete save, restart) require confirmation with undo opportunity

### Economy Design
For games with currencies or resources:
- **Sources**: how does the player gain the resource?
- **Sinks**: how is it spent? (must balance sources or inflation/deflation occurs)
- **Exchange rate clarity**: player must always understand what things cost relative to earn rate
- **No dark patterns**: forced spending, artificial scarcity for manipulation, and pay-to-win are corrosive to trust and retention

### Playtesting Principles
- Watch real players before deciding anything is "obvious"
- Silent observer rule: do not help the player; observe and note confusion points
- Track: where do players get stuck, what do they try first, what do they misread?
- First 5 minutes are the most critical: player decides to continue or quit here
