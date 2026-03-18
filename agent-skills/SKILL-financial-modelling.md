---
name: financial-modelling
description: Financial model construction — P&L, unit economics, cash flow, scenario analysis. Load when acting as Finance agent.
---

# Financial Modelling Skill

## Core Methodology

### Model Construction Rules
1. **Assumptions first** — list every assumption before building any formula
2. **Label confidence** — tag each assumption as: Historical (measured), Industry benchmark, Estimated (your best guess), or Speculative
3. **Single source of truth** — each variable appears in exactly one place; all other references point to it
4. **Never hard-code numbers in formulas** — every constant must be a named variable with a comment

### Unit Economics Framework
For any product/service, establish these metrics first:
- **CAC** (Customer Acquisition Cost) = Total acquisition spend / New customers
- **LTV** (Lifetime Value) = Average order value × Purchase frequency × Customer lifespan
- **LTV:CAC ratio** — target ≥ 3:1 for a viable business
- **Payback period** = CAC / (Monthly revenue per customer × Gross margin)
- **Gross margin** = (Revenue - COGS) / Revenue

### Scenario Framework
Always model three scenarios:
- **Base case**: most likely outcome based on current evidence
- **Upside**: what if the top 2 assumptions are more favourable by [N]%?
- **Downside**: what if the top 2 assumptions are worse by [N]%?

The range between upside and downside is more important than the base case number.

### Sensitivity Analysis
After building the base model:
1. Identify the 5 most impactful assumptions
2. For each: calculate output change per 10% input change
3. Rank by sensitivity — the top 2–3 are where to invest in better data

### Cash Flow vs P&L
- **P&L** shows profitability (accrual — when revenue is earned/costs incurred)
- **Cash flow** shows liquidity (cash — when money moves)
- A profitable company can go bankrupt; model both

### Financial Model File Format
Structure any model file as:
```
1. Assumptions sheet/section — all variables with labels and confidence
2. Revenue model — how revenue is built up from unit economics
3. Cost model — fixed costs + variable costs separately
4. P&L summary — monthly for year 1, quarterly for years 2–3
5. Cash flow — monthly
6. Scenarios comparison — side-by-side base/upside/downside
7. Key metrics summary — single-page executive view
```

### Common Modelling Errors to Avoid
- **Hockey stick revenue** without a specific driver for the inflection point
- **Fixed costs treated as variable** (or vice versa) — categorise correctly
- **Ignoring churn** in subscription models
- **Forgetting working capital** — growth consumes cash before revenue arrives
- **Circular references** — break them or model cash separately from accrual
