# Session prompts — 2026-07-28

Verbatim prompts that drove this session's research thread (momentum backtest → lookback/MA comparison → overfitting check).

## 1. Initial backtest

> You are my quant research assistant. Using your code tool, do this and
> SHOW me the results and an equity-curve chart:
>
> 1. Create ~10 years of daily prices for a stock index. If we have real
>    data connected, use it; otherwise simulate a realistic price series.
> 2. Build a time-series MOMENTUM strategy: go long when the trailing
>    126-day return is positive, short when negative.
> 3. Backtest it CORRECTLY: trade on the NEXT day (no look-ahead) and
>    charge 1 basis point of cost per trade.
> 4. Report: annualised Sharpe, max drawdown, and turnover, and plot the
>    equity curve. Explain each number in one line

## 2. Lookback comparison + MA crossover + self-check

> Now compare three lookbacks (63, 126, 252 days) in one table of
> Sharpe / max drawdown / turnover. Then add a moving-average crossover
> strategy (20 vs 100 day) and add it to the table. Keep costs and the
> no-look-ahead rule the same.
> PROMPT 1.3 — make it check its own work

## 3. Overfitting check

> Run a full overfitting check on the strategy from our last backtest and
> give me a clear VERDICT. Do all of the following and show the numbers:
> 1. Walk-forward test: pick parameters on earlier data, measure on later
>    data it never saw; repeat across several windows. Report out-of
>    sample Sharpe.
> 2. Deflated Sharpe Ratio: adjust the Sharpe for the number of parameter
>    settings we tried (multiple testing).
> 3. Probability of Backtest Overfitting (PBO): estimate how likely the
>    'best' setting is just noise.
> 4. Cost sensitivity: recompute Sharpe at 0, 1, 5, 10, and 20 bps.
> 5. VERDICT: PASS or FAIL, with blockers (weak out-of-sample) vs
>    watch-outs (unstable settings) listed separately.
> Then write a short dated research report I can keep.

## Notes

- All backtests use the same simulated 10-year daily index (GBM drift 8%/yr + GARCH(1,1) vol clustering, seed 20260728) — no live market data was connected during this session.
- Full methodology and numbers for each step are in the corresponding artifact/report, not reproduced here.
