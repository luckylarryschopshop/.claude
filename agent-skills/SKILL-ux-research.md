---
name: ux-research
description: UX methodology — user research, journey mapping, interaction specification, usability review. Load when acting as UX agent.
---

# UX Research Skill

## Core Methodology

### Jobs-To-Be-Done Framework
For every persona, define:
- **Functional job**: the practical task they're trying to do ("file my quarterly taxes")
- **Emotional job**: how they want to feel doing it ("confident I won't make a mistake")
- **Social job**: how they want to appear to others ("look financially organised")

JTBD replaces feature-thinking with outcome-thinking. Design flows against the job, not the feature.

### User Journey Mapping
Steps:
1. Define the job/goal at the journey level
2. Break into phases (Awareness → Consideration → Decision → Use → Support)
3. For each phase: user actions, thoughts, feelings, touchpoints
4. Identify **moments of truth** — where the experience makes or breaks the relationship
5. Identify **pain points** — friction, confusion, or failure states
6. Identify **opportunity gaps** — where competitors fail and you can win

### Nielsen's 10 Heuristics (Review Checklist)
1. **Visibility of system status** — user always knows what's happening
2. **Match between system and real world** — use user language, not system language
3. **User control and freedom** — easy undo, back, cancel
4. **Consistency and standards** — same label = same action everywhere
5. **Error prevention** — prevent mistakes before they happen (better than error messages)
6. **Recognition over recall** — show options, don't require memorisation
7. **Flexibility and efficiency** — power users can shortcut; new users can discover
8. **Aesthetic and minimalist design** — every UI element needs a reason to exist
9. **Help users recognise, diagnose, and recover from errors** — plain language, actionable
10. **Help and documentation** — findable, searchable, task-oriented

### Interaction Specification Standards
Every interaction spec must cover:
- **Happy path**: the expected user flow
- **Error path**: what happens when user input is invalid
- **Empty state**: what user sees with no data
- **Loading state**: what user sees while waiting
- **Edge cases**: user does something unexpected

Touch interaction rules (mobile):
- Minimum touch target: 44×44px (Apple HIG) / 48×48dp (Material)
- Touch targets must not overlap
- Swipe gestures need visual affordance
- Never rely on hover state for touch interfaces

### Usability Anti-Patterns to Avoid
| Anti-pattern | Why it fails |
|--------------|-------------|
| Dark pattern CTA | Destroys trust; damages retention |
| Forced registration before value | Increases bounce; delays first success |
| Vague error messages ("Something went wrong") | User cannot self-recover |
| Too many options per screen | Decision paralysis; "paradox of choice" |
| Modal on first visit | Interrupts before user has context |
| Infinite scroll without position restore | User loses place after navigation |
| Form that clears on error | Punishes user for validation mistakes |

### Onboarding Design Principles
1. **Deliver value in the first session** — user must feel they accomplished something
2. **Progressive disclosure** — reveal complexity only as user needs it
3. **Empty state as invitation** — first empty state should guide the user to first action
4. **Skip option always available** — never force a tutorial; offer it
5. **Set expectations early** — show what the product does before asking for sign-up

### Accessibility Requirements (UX Layer)
- Reading level: aim for Grade 8 (Flesch-Kincaid)
- Error messages: say what went wrong AND how to fix it
- Focus management: after modal opens, focus moves to it; after close, returns to trigger
- Keyboard navigation: every flow completable without a mouse
- Screen reader: meaningful alt text for all images that convey information
