# Implementation Plan - Moving Average Backtest

## Approach
Copy the structure of `multi-instrument-channel-breakout-backtest/mib_backtest.py` (loading, calendar, portfolio accounting, metrics, dark tearsheet) and replace the channel signals with SMA signals. The existing skills are untouched.

## Components in `ma_backtest.py`
1. `load` / `collect` / `align` - read and validate files, accept one or many, build the shared calendar. Zero or negative prices are refused.
2. `sma` - average of the last N closes including the current bar, rounded to 10 decimals so calculation noise cannot create a false cross.
3. `run` - compares fast and slow lines each close and fills at the next open. Price vs SMA is treated as a fast line of length 1 (the close itself).
4. Metrics, charts, HTML and PDF - reused, with SMA lines on the price charts and a "Buy & hold" label for single instruments.
5. `main` - arguments, validation, warm-up warnings, outputs.

## Design decisions
- Trend-state signals: the strategy holds whenever the trend is up, so it can enter on the first possible signal instead of waiting for a fresh cross.
- Skipped entries are counted once per above-average spell, so the warning is not inflated by daily retries.
- Rebalancing is switched off with a warning for a single instrument, because there is nothing to balance against.

## Changelog
- **[23/11/2026]** Published to GitHub. No code changes.
- **[23/11/2026]** Plan written and implemented. Tested on made-up data, including an independent recalculation of final equity and trade counts.
