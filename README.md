# quant-research

Research notes, backtest reports, and the prompts that produced them.

## Convention: "save my work"

When asked to "save my work", commit into this repo:
- `reports/` — the latest dated research report (markdown)
- `prompts/` — the session's prompts that produced it
- any associated Skill, once one exists for this workflow

Each save is one commit with a short, dated message.

## Structure

```
reports/   dated research reports (one per research session)
prompts/   verbatim prompts behind each report, dated to match
code/      self-contained HTML/JS tools that generated the reports — open directly in a browser
data/      cached market data (e.g. Yahoo Finance pulls), so re-runs don't re-download
```

## code/

- `momentum_backtest.html` — 10-year simulated-index backtest engine: momentum signal (configurable lookback), 20/100-day MA crossover, next-day execution, 1bp cost, Sharpe/max-drawdown/turnover, lookback comparison table, and a self-check panel that asserts no look-ahead on the live output.
- `overfitting_check.html` — walk-forward test, Deflated Sharpe Ratio, Probability of Backtest Overfitting (CSCV), cost sensitivity, and a PASS/FAIL verdict, all computed against the same simulated price path (same seed) as the backtest above.

Both run entirely client-side (no server, no dependencies) — double-click to open, or serve statically.
