---
name: moving-average-backtest
description: Backtests a long-only simple moving average (SMA) strategy on ONE or SEVERAL stocks/ETFs from one CSV or Excel price file per instrument and produces an HTML tearsheet. Supports price vs one SMA (e.g. the 200-day) and fast/slow SMA crossovers (e.g. 50/200, golden cross / death cross); several instruments trade as one portfolio with shared cash and optional rebalancing. Use whenever the user wants to test a moving average, SMA, MA crossover or trend-filter strategy; not for EMA, channel breakout or momentum rotation strategies.
---

# Moving Average Backtest (SMA)

Runs one fixed strategy family: stay invested while the trend is up according to a simple moving average, and sit in cash while it is down. All the math lives in `ma_backtest.py` (saved next to this SKILL.md), so every run uses identical rules. Do not change the rules, add optimisation, or test other parameter sets unless the user explicitly asks for a separate run.

The user is not a programmer: explain results in simple words, use a real-world analogy when a concept comes up (e.g. an SMA = "a smoothed trend line, like the average temperature of the last 50 days instead of today's weather"), and keep replies short.

## Step 1 - Get the price files
- The user gives one file, a list of files, or a folder: one `.csv` / `.xlsx` per instrument. If none are given, ask.
- Each file must contain `date`, `open`, `high`, `low`, `close` (case-insensitive, extra columns ignored). The date column may also be called `time` (TradingView exports). Dates may be dd/mm/yyyy or yyyy-mm-dd (or native Excel dates); they are never guessed. If anything is missing the script stops and says which file and column. Tell the user in plain words and do not guess or rebuild columns.
- Excel workbooks must have a single sheet each. Instrument name = file name, unless the user gives labels (`--names`, same order as the files).

## Step 2 - Scan, then confirm the start date
1. Run the scan (no backtest yet):
```
py ma_backtest.py --folder "<folder>" --scan
py ma_backtest.py --files "<a.csv>" ["<b.csv>" ...] [--names A B] --scan
```
2. Show a small table: each instrument's first date, last date and bar count, then the overlapping period (`overlap_start` to `overlap_end`).
3. Ask: start from the **overlap start date**, or a **different start date** on or after it? Dates are dd/mm/yyyy.
4. Tell the user that bars before the start date are used to warm up the SMA when they exist. If there are not enough, that instrument stays in cash until it has enough history, and the script reports the date.

## Step 3 - Ask for the parameters (always ask, never assume)
Ask in one message (use the multiple-choice question tool if available). Only commission and slippage have defaults.
1. **Rule** - **Price vs SMA** (in the market while the close is above one SMA) or **SMA crossover** (in the market while a fast SMA is above a slow SMA).
2. **Length** - for Price vs SMA, the SMA length in bars. For crossover, the fast and slow lengths in bars (fast must be shorter). Bars follow the file: daily files mean days, weekly files mean weeks.
3. **Starting capital** - for the whole portfolio.
4. **Commission** - flat amount per order, on every entry, exit and rebalance order. Default 0.
5. **Slippage** - percent of the fill price, against you on every fill. Default 0%.
6. **Rebalancing** - only ask with 2+ instruments: None, Monthly, Quarterly or Yearly.
7. **Output** - HTML only (default); PDF on request.

The same settings apply to every instrument (no per-instrument settings).

Fixed rules (do not ask, state them in the confirmation):
- Long only, at most one position per instrument, whole shares only. Leftover cash stays in the pool. An entry that cannot afford one share is skipped, flagged, and tried again on the next signal bar.
- **SMA** = plain average of the last N closes, including the signal bar, using each instrument's own bars.
- **Signal on the close, fill at the next bar's open.** Buy when flat and above; sell when held and below. Exactly equal means no new action.
- **Already above on the first possible signal counts as a buy.** The strategy follows the current trend state, so it does not wait for a fresh cross.
- **Shared cash pool:** a buy uses `cash in pool / number of empty slots`. Sells fill before buys.
- **Calendar:** every date on which at least one instrument traded is a portfolio date. A market with no bar that day cannot trade (orders wait for its next bar), and its last close is carried forward for valuation.
- **Rebalancing** (if chosen): at the close of the last trading day of the month / quarter / year, held instruments go back to `total equity / number of instruments` (trims first, then top-ups limited by cash). Positions with a pending exit are left alone. The unfinished final period is never rebalanced.
- An open position at the end is marked at the last close (no exit cost).
- Benchmark: buy-and-hold (equal-weight across instruments), bought at the first bar's open with the same costs and rebalanced on the same schedule.

Repeat the chosen settings back in one short line, then run.

## Step 4 - Run the script
1. Check `pandas`, `numpy`, `matplotlib` and `openpyxl` import; install any that are missing (`pip install <pkg>`).
2. Run (on Windows plain `python` may hit the Microsoft Store stub; use `py`, or the full path to the real Python):
```
py ma_backtest.py --folder "<folder>" | --files <f1> [<f2> ...] [--names <n1> <n2> ...] \
  [--start dd/mm/yyyy] --rule price-ma --ma <N> | --rule crossover --fast <F> --slow <S> \
  --capital <capital> --commission <commission> --slippage <slippage_pct> \
  [--rebalance none|monthly|quarterly|yearly] [--pdf] --outdir <output folder>
```
   `--slippage` is in percent: `0.05` means 0.05%. Leave `--start` out to use the overlap start.
3. The script prints JSON. If `"status": "error"`, explain the message in plain words and stop. Do not work around it.

## Step 5 - Present the results
- Say where the HTML tearsheet (and PDF if requested) and the trade list CSV were saved.
- Give 3-5 short lines: CAGR, max drawdown, Sharpe and number of trades for the strategy vs buy-and-hold, the best and worst instrument by contribution (if several), plus any data warnings. The tearsheet has the rest.
- Do not suggest "better" parameters or run extra variations unless asked.

## Tearsheet contents (dark theme)
Strategy rules; strategy vs buy-and-hold (final equity, total return, CAGR, volatility, Sharpe, Sortino, max drawdown, Calmar, longest drawdown); trade statistics; per-instrument table; equity curve; drawdown; open positions over time; monthly returns heatmap; price chart with SMA line(s) and trade markers per instrument; combined trade list. Ratios use a zero risk-free rate, annualised from the data frequency. All dates dd/mm/yyyy.
