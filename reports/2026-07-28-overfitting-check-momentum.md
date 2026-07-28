# Overfitting Check: Time-Series Momentum

**Date:** 28 July 2026
**Scope:** Follow-up to the momentum-lookback backtest (63/126/252-day momentum + 20/100-day MA crossover), same simulated 10-year index, same seed (20260728), same 1bp cost and 1-day execution lag.

## VERDICT: FAIL

**2 blocking issues.** As specified, this strategy should not be traded on this evidence alone.

| Check | Result | Read |
|---|---|---|
| Walk-forward OOS Sharpe | **0.01** | watch-out |
| Deflated Sharpe Ratio | **0.878** | **blocker** (< 0.90) |
| PBO | **90.4%** | **blocker** (> 50%) |
| Cost breakeven (headline) | > 50bp | clear |

---

## 1. Walk-forward test

Anchored (expanding) walk-forward: pick the best-performing rule on data before a window, apply it unchanged to the window it has never seen. 5 folds total — the first is a 453-day training-only burn-in, followed by 4 out-of-sample folds.

| Fold | Train window | Test window | Selected on train | In-sample Sharpe | Out-of-sample Sharpe |
|---|---|---|---|---|---|
| 1 | Nov 2017 → Aug 2019 | Aug 2019 → May 2021 | 84d momentum | 2.05 | 0.61 |
| 2 | Nov 2017 → May 2021 | May 2021 → Feb 2023 | 84d momentum | 1.20 | 0.04 |
| 3 | Nov 2017 → Feb 2023 | Feb 2023 → Oct 2024 | 84d momentum | 0.72 | **-0.82** |
| 4 | Nov 2017 → Oct 2024 | Oct 2024 → Jul 2026 | 252d momentum | 0.64 | 0.15 |

- **Pooled out-of-sample Sharpe: 0.01** (all 4 test windows stitched into one return series)
- **Mean in-sample Sharpe: 1.15**
- **OOS / in-sample ratio: ~1%** — training-set performance essentially evaporates once the rule sees new data.

## 2. Deflated Sharpe Ratio (DSR)

Corrects the best full-sample Sharpe for how many parameter settings were tried. Trial universe: momentum lookbacks {21, 42, 63, 84, 126, 168, 189, 252} plus 20/100 MA crossover — **9 configurations**, a realistic stand-in for the search space behind the earlier 63/126/252 comparison.

- Best full-sample trial: **84-day momentum**, annualised Sharpe 0.59
- Expected max Sharpe from 9 trials with zero true skill: 0.20 (annualised)
- Skew / kurtosis of the winning strategy's daily returns: 0.06 / 4.93
- **Deflated Sharpe Ratio: 0.878** — below the conventional 0.95 confidence bar, and below the 0.90 line used as a blocker here. After accounting for the multiple-testing penalty, the best result in this family cannot be reliably distinguished from noise.

## 3. Probability of Backtest Overfitting (PBO)

Combinatorially Symmetric Cross-Validation (Bailey, Borwein, López de Prado & Zhu, 2014): split the sample into 16 blocks, form every 8-vs-8 train/test split (12,870 combinations, enumerated exactly), pick the in-sample winner in each split, and check where it ranks out-of-sample.

- **PBO: 90.4%** — in 9 out of 10 resampled splits, whichever setting looked best in-sample went on to underperform the out-of-sample median. This is well past the 50% red line from the original paper: the *selection process itself*, not just any one parameter, is unreliable here.

## 4. Cost sensitivity

Sharpe recomputed at 0 / 1 / 5 / 10 / 20bp per unit of position traded (full sample):

| Strategy | 0bp | 1bp | 5bp | 10bp | 20bp |
|---|---|---|---|---|---|
| 63-day momentum | 0.35 | 0.34 | 0.29 | 0.23 | 0.10 |
| **126-day momentum (headline)** | 0.41 | 0.41 | 0.38 | 0.36 | 0.30 |
| 252-day momentum | 0.51 | 0.50 | 0.48 | 0.46 | 0.42 |
| 20/100 MA crossover | 0.15 | 0.14 | 0.13 | 0.11 | 0.07 |

None of the four break even inside 50bp — cost is not the weak link here. The lower-turnover strategies (252-day momentum, MA crossover) are structurally more cost-robust than the fast ones, as expected.

## Parameter stability (supporting evidence)

| Lookback | Sharpe | CAGR | Turnover (ann.) |
|---|---|---|---|
| 21d | 0.56 | 10.0% | 47.8x |
| 42d | 0.43 | 6.9% | 34.0x |
| 63d | 0.34 | 4.9% | 25.3x |
| **84d** | **0.59** | 10.6% | 16.0x |
| 126d | 0.41 | 6.5% | 12.0x |
| 168d | 0.44 | 7.2% | 20.5x |
| 189d | 0.41 | 6.5% | 11.8x |
| 252d | 0.50 | 8.6% | 9.2x |

Largest jump between adjacent lookbacks: **0.26 Sharpe** (63d → 84d). Sharpe is not a smooth function of the lookback — 84 sits as a spike between two much weaker neighbours (63d and 126d), which is itself a classic overfitting fingerprint.

## Blockers vs. watch-outs

**Blockers** (reasons not to trade this as-is):
- Deflated Sharpe Ratio 0.878 — after penalising for 9 trials, the best result isn't distinguishable from luck at a reasonable confidence level.
- PBO 90.4% — the in-sample winner underperforms out-of-sample most of the time; the selection process is not reliable.

**Watch-outs** (reasons to size small / keep investigating, not disqualifying alone):
- Weak walk-forward Sharpe (0.01) — positive but thin, would likely be erased by real-world frictions beyond the 1bp cost model.
- Large in-sample → out-of-sample drop-off (only ~1% of in-sample Sharpe survives).
- Unstable parameter surface — a 0.26 Sharpe swing between neighbouring lookbacks suggests 84-day is a lucky grid point, not a stable edge.

## Methodology notes & limitations

- Identical simulated price path as the prior backtest: GBM drift (8%/yr) + GARCH(1,1) vol clustering (18%/yr long-run vol, α=0.08, β=0.90), seed 20260728, 2,520 weekday observations, no live market data connected.
- DSR follows Bailey & López de Prado (2014); PBO follows the CSCV method of Bailey, Borwein, López de Prado & Zhu (2014).
- Verdict thresholds (stated so they can be contested, not black-boxed): **blocker** if walk-forward OOS Sharpe ≤ 0, or DSR < 0.90, or PBO > 50%, or cost-breakeven ≤ 10bp. **Watch-out** if OOS Sharpe < 0.30, DSR < 0.95, PBO 20–50%, cost-breakeven 10–20bp, or adjacent-lookback Sharpe swing > 0.25.
- This is a single simulated price path — conclusions describe this backtest's robustness to search and cost assumptions, not a guarantee about live markets.
- The 9-configuration trial grid is a stated assumption, not a fact recovered from the prior note; a different (larger or smaller) search space would change the DSR/PBO deflation.

*Interactive version with hoverable tables: [Overfitting Check artifact](https://claude.ai/code/artifact/7cb479e4-8eba-4650-be2b-701bf672ee98)*
