---
name: visual-design
description: Visual design methodology — design systems, colour, typography, component specification. Load when acting as Designer agent.
---

# Visual Design Skill

## Core Methodology

### Design System Construction Order
Always build in this order (each layer depends on the one below):
1. **Colour system** — semantic colour map (not just palette)
2. **Typography scale** — sizes, weights, line heights
3. **Spacing and grid** — consistent spatial rhythm
4. **Elevation and shadow** — depth system
5. **Border and radius** — shape vocabulary
6. **Component tokens** — component-specific compositions of the above
7. **Component specifications** — all states, variants, and sizes

### Colour System Structure
```
Primitive palette (raw hex values):
  --blue-100 through --blue-900 (11 steps)
  [repeat for each hue]

Semantic tokens (reference primitives):
  --color-primary:         var(--blue-600)  /* primary actions */
  --color-primary-hover:   var(--blue-700)
  --color-primary-text:    var(--blue-50)   /* text on primary bg */
  --color-background:      var(--gray-50)
  --color-surface:         white
  --color-border:          var(--gray-200)
  --color-text-primary:    var(--gray-900)
  --color-text-secondary:  var(--gray-600)
  --color-error:           var(--red-600)
  --color-success:         var(--green-600)
  --color-warning:         var(--amber-500)
```

**Never use primitive tokens in component specs.** Always use semantic tokens so dark mode can be added by remapping tokens.

### Accessibility — Colour Contrast
| Use | Required ratio | WCAG level |
|-----|---------------|------------|
| Body text (≥18px or ≥14px bold) | 3:1 | AA |
| Body text (< 18px) | 4.5:1 | AA |
| Body text (< 18px, AAA) | 7:1 | AAA |
| UI components (buttons, inputs) | 3:1 | AA |

Check all colour pairs: text on background, text on primary, text on error, etc.

### Typography Scale
Use a modular scale (ratio between sizes):
- Ratio 1.25 (Major Third): 12, 15, 19, 24, 30, 38, 48px
- Ratio 1.333 (Perfect Fourth): 12, 16, 21, 28, 37, 50px

Required roles:
- `display`: hero/headline text — largest
- `heading-1` through `heading-4`: section hierarchy
- `body-lg`, `body-md`, `body-sm`: reading text
- `label`: form labels and metadata
- `caption`: supporting text, timestamps

### Component Specification Template
For every component:
```
Component: [Name]
Purpose: [one-line]

Variants: [primary | secondary | danger | ghost]
Sizes: [sm | md | lg]

States:
  default:  bg: --color-primary, text: white, border: none
  hover:    bg: --color-primary-hover
  focus:    bg: --color-primary-hover, outline: 2px --color-focus-ring
  active:   bg: --color-primary-pressed
  disabled: bg: --color-disabled, text: --color-text-disabled, cursor: not-allowed
  loading:  [spinner position and size]

Spacing:
  padding: [space-2] [space-4]   (vertical, horizontal)
  min-width: [N]px
  min-height: [N]px (touch target ≥ 44×44px)

Typography: font-size-md, font-weight-semibold
Radius: --radius-md
```

### Dark Mode Design
- Design light mode first; dark mode is a semantic token remap
- Never use `white` or `black` directly — use semantic tokens
- Dark mode is not light mode inverted — use mid-range backgrounds (gray-800/900), not black
- Elevation in dark mode: lighter backgrounds for higher elevation (opposite of light mode)
