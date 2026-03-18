---
name: trading-strategy
description: Quantitative trading methodology — strategy design, backtesting standards, risk management rules. Load when acting as Trader agent.
---

# Trading Strategy Skill

## Core Methodology

### Strategy Development Process
1. **Hypothesis first** — state the inefficiency you believe exists before looking at data
2. **Define rules precisely** — every condition must be binary (true/false), not subjective
3. **Out-of-sample testing** — use a hold-out period; never tune and test on the same data
4. **Realistic simulation** — model transaction costs, slippage, and market impact
5. **Walk-forward testing** — refit parameters on rolling windows to test robustness

### Lookahead Bias Prevention (Critical)
The most common and costly backtesting mistake:
- Never use future data in signal calculation (e.g., using today's close in a signal evaluated at open)
- Shift all data series by at least 1 bar before using in entry signals
- Be explicit about the execution bar: signal on close N, execute on open N+1
- Use point-in-time data for fundamentals (not restated figures)

### Position Sizing Methods
| Method | Formula | Use when |
|--------|---------|----------|
| Fixed fraction | Position = Portfolio × f | Simple, predictable |
| Kelly criterion | f = (p × b - q) / b | Maximise growth (use half-Kelly for safety) |
| Volatility-targeted | Position = (Target vol / Asset vol) × Capital | Normalise risk across assets |
| Max drawdown limit | Size such that 3σ loss ≤ max single-loss rule | Risk-parity approach |

**Default: fixed fraction (1–2% per trade) until strategy has 200+ live trades of history.**

### Performance Metrics (Required Set)
- **CAGR** — compound annual growth rate
- **Sharpe ratio** — risk-adjusted return (target > 1.0, > 1.5 is strong)
- **Sortino ratio** — penalises only downside volatility
- **Max drawdown** — largest peak-to-trough decline (must be tolerable before deployment)
- **Max drawdown duration** — how long was recovery? (>12 months is psychologically damaging)
- **Win rate** — % of trades profitable (alone, meaningless — always pair with profit factor)
- **Profit factor** — gross profit / gross loss (target > 1.5)
- **Calmar ratio** — CAGR / Max drawdown

### Risk Management Rules (Non-Negotiable)
- **Max position size**: no single position > 5% of portfolio (unless explicitly justified)
- **Max daily drawdown**: stop trading the day if portfolio falls > [N]%
- **Max sector concentration**: no more than 25% in correlated positions
- **Correlation check**: before adding a position, measure its correlation to existing book
- **Kill switch**: define the condition under which the strategy is suspended (e.g. 3× expected daily drawdown in one week)

### Backtesting Report Format
```
# Backtest: [Strategy Name]
Period: [YYYY-MM-DD] to [YYYY-MM-DD]
In-sample: [period] | Out-of-sample: [period]

## Parameters
[list all tuned parameters with values]

## Performance
| Metric | In-Sample | Out-of-Sample |
|--------|-----------|---------------|
| CAGR | | |
| Sharpe | | |
| Max DD | | |
| Win Rate | | |

## Trade Analysis
Total trades: N
Average holding period: [N] days
Best trade: +[N]% | Worst trade: -[N]%

## Degradation
In-sample to out-of-sample Sharpe degradation: [N]%
[<30% degradation is acceptable; >50% suggests overfitting]
```
