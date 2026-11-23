# Moving Average Backtest Skill

A Claude skill that backtests a **simple moving average (SMA) strategy** on one or several stocks/ETFs.

**In plain words:** an SMA is like judging the season by the average temperature of the last 50 days instead of today's weather. This skill stays invested while the trend looks warm, moves to cash when it turns cold, and shows how that would have done versus simply holding.

## What it does
- You give it one price file (CSV or Excel) per instrument. One file or many both work.
- Each run, choose **Price vs SMA** (e.g. the 200-day) or **SMA crossover** (e.g. 50/200).
- Several instruments trade as one portfolio with one shared pot of cash, with optional rebalancing.
- Output: a dark-theme HTML tearsheet (optional PDF) and a trade list CSV.

## Files
| File | Purpose |
|---|---|
| `SKILL.md` | The instructions Claude follows |
| `ma_backtest.py` | The script that does all the maths and builds the report |
| `docs/PRD.md` | What the skill must do |
| `docs/implementation_plan.md` | How it was built |
| `docs/task.md` | To-do / progress |

## How to use
Ask Claude for a moving average backtest and give it your price files. Claude scans them, asks for the settings, runs the script and summarises the results.

Direct commands (Windows):
```
py ma_backtest.py --files "SPY.csv" --rule price-ma --ma 200 --capital 100000 --outdir "<out>"
py ma_backtest.py --folder "<folder>" --rule crossover --fast 50 --slow 200 --capital 100000 --rebalance quarterly --outdir "<out>"
```

## Changelog
- **[23/11/2026]** Published to GitHub.
- **[23/11/2026]** First version: price vs SMA and SMA crossover, one or many instruments, HTML/PDF tearsheet.
