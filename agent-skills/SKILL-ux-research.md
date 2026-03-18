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

---

### Kano Model — Feature Categorisation

Use when researching which features to build next. Kano categorises features by the relationship
between their presence/absence and user satisfaction.

| Category | Absent | Present | Strategy |
|----------|--------|---------|----------|
| **Must-be** (basic expectations) | Very dissatisfied | Neutral (taken for granted) | Ship first; not differentiating |
| **One-dimensional** (performance) | Proportionally dissatisfied | Proportionally satisfied | More = better; invest in proportion to ROI |
| **Attractive** (delighters) | Neutral (users don't know they want it) | Delighted | High impact; hard to copy; creates fans |
| **Indifferent** | Neutral | Neutral | Don't build; opportunity cost too high |
| **Reverse** | Some users prefer absence | Some users dislike it | Research before building |

**How to apply:** in user interviews, ask two versions per feature:
- Functional: "How would you feel if this feature existed?"
- Dysfunctional: "How would you feel if this feature did NOT exist?"

Map answers (delighted/expected/neutral/tolerable/dissatisfied) to the Kano matrix to classify.

---

### Cognitive Load Principles

Reduce the mental effort required to use the interface.

**Miller's Law:** working memory holds 7 ± 2 items. Limit choices, steps, and items per screen.

**Chunking:** group related information visually. Users process chunks, not individual items.
```
Bad:  12 navigation items in a flat list
Good: 4 groups of 3 items with visual separators
```

**Progressive disclosure:** show only what the user needs for the current step.
- Default view: essential actions only
- Secondary actions: revealed on demand (expand, "More options", contextual menus)
- Expert controls: available but not prominent

**Cognitive load anti-patterns:**
| Anti-pattern | Why it overloads | Fix |
|-------------|-----------------|-----|
| Form with 15+ fields | Too many decisions at once | Multi-step form, show only relevant fields |
| Dashboard with 20 metrics | Can't identify what matters | Hierarchy: primary KPIs large, secondary small |
| Action confirmed by clicking OK in a confirmation modal | Dialog adds no information | Inline confirmation with specific action text ("Delete 3 items") |
| Error message that requires reading to understand action | Parsing + decision while stressed | Lead with action: "Try again" before explanation |

---

### Error Recovery Patterns (Norman's Design Principles)

Good error design prevents errors AND helps users recover gracefully when they do occur.

**Error prevention first (best):**
- Constrain inputs: use select menus, date pickers, sliders instead of free text where possible
- Validate on blur (not on submit) — catch errors before the user moves on
- Confirm destructive actions with explicit object name: "Delete 'Invoice 2024-01'" not "Delete item"

**Error recovery (when prevention fails):**

| Recovery pattern | Use when | Example |
|-----------------|----------|---------|
| **Undo** | Action is reversible and user may regret it | Email "send" with 5s undo snackbar |
| **Inline correction** | Form validation failure | Highlight field + specific message + how to fix |
| **Suggested correction** | User input close to valid value | "Did you mean: john@example.com?" |
| **Safe defaults** | User left a required field blank | Pre-fill with sensible default, make it editable |
| **Graceful degradation** | System failure | Show stale data with "Last updated X" rather than blank |

**Error message formula:**
1. What went wrong (specific, not "an error occurred")
2. Why it went wrong (if it helps the user)
3. How to fix it (always)
4. Who to contact if they can't fix it (for blocking errors)

```
Bad:  "Invalid input"
Good: "Password must be at least 8 characters. Use a mix of letters and numbers."
```
