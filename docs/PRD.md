# PRD - Moving Average Backtest

## Goal
Let the user test a simple moving average trend strategy on one or several instruments and read the result in a clear tearsheet.

## Requirements
- Input: one CSV/Excel file per instrument with `date`, `open`, `high`, `low`, `close`. One or many instruments. Date column may be named `time`; dates dd/mm/yyyy or yyyy-mm-dd.
- Rule chosen each run: Price vs SMA, or fast/slow SMA crossover.
- SMA only. Lengths are in bars of the file's frequency.
- Long only. In the market while above, in cash while below. Already above on the first possible signal counts as a buy.
- Signal on the close, fill at the next bar's open. Commission and slippage on every fill. Whole shares.
- Several instruments share one cash pool (cash / empty slots), with optional monthly, quarterly or yearly rebalancing.
- History before the chosen start date warms up the SMA when it exists.
- Output: dark HTML tearsheet (optional PDF) and trade list CSV. Dates dd/mm/yyyy.
- Benchmark: buy-and-hold (equal-weight for several instruments) with the same costs and rebalancing schedule.

## Out of scope
EMA and other average types, short selling, parameter optimisation, per-instrument settings, stop losses.

## Changelog
- **[23/11/2026]** Published to GitHub. No requirement changes.
- **[23/11/2026]** First version of requirements, agreed with the user: one or many instruments, rule picked each run, SMA only, tested on made-up data.
