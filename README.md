# Model Portfolio Tool

A Python project that builds and tests three model portfolios for different levels of risk.

## What it does

It starts from a handful of low-cost ETFs covering the main asset classes (UK equity,
global equity, emerging market equity, global bonds, UK gilts, gold) and combines them into three portfolios (Cautious, Balanced, Adventurous). It then backtets each one, rebalances them once a year, and works out where the returns came from. The output is a one-page summary for each portfolio.

## How it works

Each asset class is represented by one ETF, with prices pulled from Yahoo Finance. The weights are set so that as risk tolerance goes up, the equity share rises and the bond share falls, with a small fixed holding in gold for diversification. The backtest sets each portfolio to its target weights and then lets those weights drfit as the market moves, resetting them to target once a year. Modelling the drift matters. If you skip this, the test quietly assumes the portfolio stays perfectly balanced, which understates the risk it actually builds up over time.

To explain returns, the project uses Brinson attribution. This splits a portfolio's return against a 60/40 benchmark into two parts: how much came from the asset allocation, meaning how much sits in each asset class, and how much came from selection, meaning what is held within each class. Because the portfolios use index trackers, the selection part comes out close to zero, which is what you would expect when you are not picking active funds. 

Risk and return are measured with the usual figures: annual return, volatility, Sharpe and Sortino ratios, and maximum drawdown.

## The charts

* Correlation between the asset classes
* Risk against return for each building block
* The asset mix for each portfolio
* Growth of £100 over the period
* Drawdowns through market falls, including 2022
* Returns by calendar year
* A return attribution breakdown
* A one-page summary per portfolio

## Running it

Open the notebook in Google Colab, run the first cell to install the one extra library it needs, then run the rest in order. It uses free data and needs no API key.

## What I'd add next

* Swap one tracket for an active fun, so the selectio side of attribution actually shows something
* Use a real cash rate instead of zero in the risk figures
* Add fund and platform fees to turn the gross returns into net ones

## Limitations

The history runs from around 2018, set by the youngest ETF, so it covers the 2020 COVID shock and the 2022 fall but not earlier ones. Returns are before fees and tax, and rebalancing is treated as free unless the cost setting is turned on. Some holdings are priced in dollars, so there is some currency movement in the returns that is not hedged.
