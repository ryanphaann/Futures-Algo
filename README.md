# Systematic Trend-Breakout Research — Equity-Index Futures

A multi-year research project building and validating a systematic breakout strategy on
US equity-index futures, backtested over **20 years of 1-minute data**.

> **This repository is a methodology & results case study.** The alpha-defining signal
> logic is intentionally withheld — see [*What's deliberately omitted*](#whats-deliberately-omitted).
> What's documented here is *how the system was built, tested, and stress-tested*, and how
> honestly it held up — not the recipe.

---

## TL;DR

- A **20-year, 1-minute backtest engine** with exact intrabar fills and automated
  fill-integrity audits.
- A trend-conditioned breakout edge on **E-mini Nasdaq (NQ)** and **S&P (ES)** futures,
  confirmed **out-of-sample** by anchored walk-forward.
- Stress-tested across **9 instruments and 4 asset classes** — most fail. The honest map of
  *where the edge lives and where it doesn't* is the real deliverable.
- Supporting infrastructure: a multi-instrument continuous-contract data pipeline, an
  Interactive Brokers live paper-trading bot (signal-fidelity proven), and a Monte-Carlo
  prop-firm campaign model.

---

## Headline results

*Final configuration, full 20-year history, exact fills:*

| Metric | NQ (Nasdaq) | ES (S&P) |
|---|---:|---:|
| Profit factor | 1.53 | 1.30 |
| Sharpe (annualised) | 1.07 | 0.77 |
| Max drawdown | 26.6% | 27.3% |
| Win rate | ~36% | ~36% |
| Payoff (avg win ÷ avg loss) | 2.67 | 2.34 |
| Trades (20y) | 1,942 | 1,559 |

**Provenance matters.** Those figures are *in-sample* (parameters fit on the full history).
The realistic, **out-of-sample** number — anchored walk-forward, parameters re-fit on
past-only data per fold — is **profit factor ≈ 1.11, Sharpe ≈ 0.63, with 6 of 6 folds
positive**. That degradation (1.07 → 0.63 Sharpe) is expected, and it's the number to plan
around. It is disclosed rather than buried.

**Design philosophy:** a ~36% win rate is *intentional* — lose small often, win big
occasionally. The 2.67 payoff is what carries the system. This is a trend-following risk
profile, not a high-accuracy scalper.

---

## What makes this rigorous (the actual value)

The point of this project isn't a single number — it's the discipline behind it.

- **The 3-month mirage → 20-year reality check.** Early "edges" discovered on a single
  quarter of data were systematically re-tested on the full 20 years. Most evaporated. The
  lesson — *a short sample lies* — governed everything afterward.
- **Killed-edge discipline.** An idea was adopted only if it survived the entire history, and
  abandoned the moment it didn't. The graveyard includes a Hidden Markov regime model, a
  Hurst-exponent filter, a variance-ratio filter, a day-of-week filter, breakeven stops,
  trailing stops, a swept-level rule, and several time-of-day overlays — each tested and
  rejected on the evidence.
- **Anchored walk-forward + era-robustness.** Parameters re-fit on past-only data, performance
  pooled out-of-sample, and stability checked across market eras (2008 crisis, 2020 crash,
  2022 bear).
- **Execution realism.** A multi-timeframe exact-fill model honours protective stops at their
  *set* price rather than a lagged bar close, and **every backtest is followed by an automated
  fill audit** that verifies no trade filled at a price the market never printed — catching
  phantom fills and inflated pricing before they can flatter a result.

---

## Cross-asset transfer study

The same engine, run **unchanged**, across asset classes — a test of how general the edge is,
and a demonstration of *not fooling oneself*:

| Asset class | Result | Verdict |
|---|---|---|
| Equity-index futures (NQ, ES) | PF 1.30–1.53, out-of-sample validated | ✅ home turf |
| Silver | PF 1.18, Sharpe 0.54, 7/9 out-of-sample windows positive | ✅ transfers (the config never touched silver — genuinely OOS) |
| Gold | PF 1.12 all-hours; an hour-filtered variant **failed** its walk-forward | ⚠️ marginal |
| US mega-cap stocks | A long-only edge on 2 of 6 names; the short side loses on all 6 | ⚠️ narrow + bull-market beta |
| Russell 2000 | Breakeven over an 8.9-year (recent-only) sample | ⚠️ confounded by regime |
| FX futures (4 major pairs, incl. the most liquid) | 0-for-4, ~breakeven, ~30% win | ❌ wrong asset class |
| Sub-hourly timeframes | Monotonically worse — fixed costs dominate smaller moves | ❌ |

The findings generalize cleanly: this is an **equity-index trend phenomenon**, it requires
*trending* markets (it stalls in choppy / mean-reverting regimes), and pushing to finer
timeframes is a losing trade against commission. The system's **leverage settings are not
universal either** — porting the production sizing to a thin edge turned a winner into a
loser, a documented cautionary result.

---

## Engineering

- **Data pipeline.** Stitches hundreds of quarterly futures contracts into clean continuous
  series across multiple instruments — front-month rolls, delivery-month clipping, timezone
  normalization (verified empirically against exchange maintenance windows), and bad-tick
  removal — from raw 1-minute bars up to any signal timeframe.
- **Live trading bot.** An Interactive Brokers (ib_insync) paper-trading harness that
  reproduces the backtest's signals **bar-for-bar** — verified to within identical entry
  counts and timestamps against both the backtester and an independent TradingView port.
  Includes risk-based sizing, auto-reconnect with live position reconciliation, and
  good-till-cancelled bracket management for overnight holds.


---

## Honest caveats

This project is documented:

- Headline figures are **in-sample**; the realistic figure is the lower out-of-sample one.
- The large compounded dollar figures come from **compounding + pyramiding** 
- Performance has **decayed in the most recent regime** — the latest walk-forward window
  underperformed across instruments. Disclosed, not hidden.

---

