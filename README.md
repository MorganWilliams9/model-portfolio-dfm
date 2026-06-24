# Model Portfolio Construction and Management

A discretionary fund management (DFM) workflow built in Python. It constructs
three risk-graded model portfolios from low-cost ETFs, backtests them through
real market stress, attributes the returns to allocation versus selection
decisions, and produces a client-ready report.

This is built to show the judgment a discretionary manager makes, not to look
clever in code. Every decision in the notebook is explained in plain English.

---

## 1. Project Overview

The project plays the role of a discretionary fund manager running a set of model
portfolios. It takes a small menu of asset-class building blocks (UK equity,
global equity, emerging market equity, global bonds, UK gilts, gold), combines
them into three portfolios graded by risk appetite (Cautious, Balanced,
Adventurous), and then does what a real manager has to do after building them:
prove how they would have performed, explain where the returns came from, and
write it up for a client.

The whole thing runs in a single Google Colab notebook with no paid data and no
setup beyond one install line.

## 2. Real-World Finance Use Case

This is, in miniature, the actual job at a DFM or a discretionary wealth manager.
Firms like these run a ladder of model portfolios mapped to client risk profiles.
An adviser sits a client somewhere on that ladder, and the investment team is
responsible for the asset allocation, the choice of underlying funds, the
ongoing rebalancing, and the reporting that explains it all. The skills shown
here map directly onto that: strategic asset allocation, fund selection,
performance attribution, and client communication. It is deliberately not a
quant signal-generation project, because that is a different job.

## 3. System Architecture

A simple linear pipeline, each stage feeding the next:

```
  Yahoo Finance (yfinance)
          |
          v
  [1] Data collection      download_prices()      -> raw prices
          |
          v
  [2] Cleaning + features   clean_prices() / to_returns()  -> daily returns
          |
          v
  [3] Building-block analysis   stats, correlations, risk/return
          |
          v
  [4] Portfolio construction    backtest_portfolio()  (drift + yearly rebalance)
          |
          v
  [5] Backtest + stress     performance_summary(), drawdowns, calendar years
          |
          v
  [6] Attribution           brinson_attribution()  (allocation vs selection)
          |
          v
  [7] Client report         build_tearsheet()  -> one page per profile
```

The analytics are written as small pure functions, separate from the data
download, so each one can be tested on its own.

## 4. Required APIs and Data Sources

- **Yahoo Finance** via the `yfinance` library. This is the only data source and
  needs no API key. It provides daily adjusted-close prices for every ETF.
- No SEC EDGAR, FRED, FMP, Alpha Vantage or Polygon key is needed for the base
  project. Some of the upgrades in section 15 add FRED (for a risk-free rate and
  macro overlays) and FMP (for fund-level fundamentals), but the core runs on
  Yahoo alone.

## 5. Required Python Libraries

- `yfinance` for price data (the only non-default install)
- `pandas` and `numpy` for all the data handling and maths
- `matplotlib` for the charts and the tearsheet

That is the whole dependency list. It is kept short on purpose so the notebook
runs first time on a clean Colab session.

## 6. Folder / File Structure

The project ships as a notebook, but it is organised as if it were a small repo
so it reads cleanly on GitHub:

```
model-portfolio-dfm/
├── README.md                     # this file
├── Model_Portfolio_DFM.ipynb     # the full Colab notebook
├── model_portfolio.py            # the same code as a plain script (optional)
└── outputs/                      # exported tearsheets and charts (optional)
```

Inside the notebook the logic is split into a CONFIG block, a PURE FUNCTIONS
block, and then numbered sections that run the pipeline. To turn it into a real
package later you would lift the pure functions into a `src/` module and import
them, which is noted as an upgrade.

## 7. Step-by-Step Build Guide

1. Open the notebook in Colab and run the install cell once.
2. Set your building blocks and model weights in the CONFIG block. The defaults
   are sensible UK-listed ETFs and realistic risk-graded weights.
3. Run the data collection cell. It downloads each ETF, retries on failure, and
   reports anything it had to drop.
4. Run the cleaning cell to align everything onto a common calendar and convert
   prices to daily returns.
5. Run the building-block analysis to see how each brick behaves and how they
   correlate. This is where you justify why each asset class is in the mix.
6. Run the portfolio construction cell to build the three profiles with yearly
   rebalancing.
7. Run the backtest and stress cells for the metrics table, the drawdown chart,
   and calendar-year returns.
8. Run the attribution cell to split the Balanced portfolio's return into
   allocation and selection.
9. Run the client report cell to produce a one-page tearsheet for each profile.

## 8. Data Collection Pipeline

`download_prices()` loops over the building blocks and pulls adjusted-close
prices one ticker at a time. Pulling them individually, rather than in one batch,
keeps the column shape predictable and means a single bad ticker cannot corrupt
the whole frame. Each download gets up to three retries because Yahoo
occasionally times out. A helper, `_extract_close()`, handles the fact that
yfinance returns different column layouts across versions. Anything that still
fails after its retries is reported and dropped rather than crashing the run.

## 9. Data Cleaning and Feature Engineering

`clean_prices()` sorts the data, removes duplicate dates, forward-fills only
short gaps (caused by exchanges having different holidays) while refusing to fill
long gaps that would invent prices, and then trims the lead-in so the analysis
only runs over the window where every building block actually exists. The single
engineered feature everything else depends on is the daily simple return, built
by `to_returns()`. Working in returns rather than price levels also means any
currency or pence quirks in the raw quotes wash out.

## 10. Core Models and Algorithms

- **Backtest with weight drift and rebalancing.** `backtest_portfolio()` applies
  target weights but lets them drift with the market between rebalance dates, then
  resets to target each year. This is the detail most amateur backtests get wrong,
  and it can optionally charge a turnover cost on rebalance.
- **Risk and return metrics.** CAGR, annualised volatility, Sharpe, Sortino,
  maximum drawdown and Calmar, each as its own small function.
- **Brinson-Hood-Beebower attribution.** `brinson_attribution()` decomposes the
  portfolio's return over a 60/40 benchmark into an allocation effect, a selection
  effect, and an interaction term, which by construction sum exactly to the active
  return. This is the centrepiece that signals an understanding of the DFM job.

## 11. Visualisations and Dashboard Components

- Correlation heatmap of the building blocks
- Risk versus return scatter of the building blocks
- Stacked allocation chart across the three risk profiles
- Growth of 100 pounds for each portfolio
- Underwater drawdown chart
- Calendar-year return bars
- Attribution waterfall (allocation, selection, interaction)
- A one-page client tearsheet per profile combining growth, drawdown, allocation
  and a metrics table

## 12. Performance Metrics

For every portfolio the notebook reports compound annual growth rate, annualised
volatility, Sharpe ratio, Sortino ratio, maximum drawdown, Calmar ratio, best and
worst calendar year, and the share of months that were positive. The attribution
step adds allocation, selection and interaction contributions in percentage terms.

## 13. Final Deliverables

- A reproducible Colab notebook that runs end to end on free data
- Three constructed model portfolios with documented strategic weights
- A full backtest with stress periods and a metrics table
- A Brinson attribution of the Balanced portfolio versus a 60/40 benchmark
- A client-ready tearsheet for each risk profile

## 14. Resume Description

> Built a discretionary fund management workflow in Python that constructs three
> risk-graded multi-asset model portfolios from low-cost ETFs, backtests them with
> realistic weight drift and annual rebalancing through the 2022 sell-off, and
> applies Brinson-Hood-Beebower attribution to separate asset-allocation decisions
> from fund-selection decisions. Produced client-ready performance tearsheets and
> a full risk and return metric suite (Sharpe, Sortino, drawdown, Calmar).

## 15. Potential Upgrades

- **Active fund selection.** Swap one index tracker for an active fund and watch
  the selection effect in the attribution come alive.
- **Rebalancing study.** Compare yearly, quarterly and threshold-based
  rebalancing on net-of-cost returns.
- **A proper risk-free rate.** Pull SONIA or a T-bill rate from FRED so Sharpe and
  Sortino use a real cash benchmark instead of zero.
- **Forward-looking allocation.** Add a mean-variance or risk-parity optimiser as
  an alternative to the fixed strategic weights, and compare.
- **Fees and tax.** Layer in platform fees, fund OCFs and a simple tax assumption
  to move from gross to net returns.
- **Monte Carlo projection.** Simulate forward outcomes for each profile to show a
  client a range of possible paths, not just the past.
- **Packaging.** Lift the pure functions into a `src/` module with unit tests so
  the repo looks like production code, not just a notebook.

---

### Honest limitations

The backtest window is set by the youngest ETF, so it spans a few years rather
than decades, though it does cover the 2020 COVID shock and the 2022
stock-and-bond sell-off. The holdings are index trackers, so the selection effect
is small by design. Returns ignore fees and tax and assume costless rebalancing
unless the cost setting is switched on. These are deliberate simplifications, and
each one is a clean place to extend the project.
