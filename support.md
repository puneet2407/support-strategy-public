# Support Bounce — US + Canada

**Goal:** beat the S&P 500 by buying pullbacks into support that *hold*, in stocks still in an uptrend.
**Backtest window:** Jan 2013 – Nov 2026 (13.9 years), 229 US + Canadian large/mid caps, USD terms, 15 bps/side execution costs plus 25 bps/side on each CAD conversion, signals at close executed at **next day's open** (no lookahead). *Regenerated 2026-11-19 on prices through Nov 19.*

| | CAGR | Vol | Sharpe | MaxDD |
|---|---|---|---|---|
| SPY buy & hold | 14.2% | 16.5% | 0.86 | −33.7% |
| Equal-weight B&H of universe (bias floor) | 20.1% | 16.4% | 1.16 | −38.5% |
| **Support bounce (locked config)** | **32.5%** | 24.5% | **1.23** | −33.7% |
| Breakout strategy (for comparison) | 20.7% | 22.1% | 0.93 | −29.6% |
| 50/50 support + breakout | 27.0% | 21.3% | 1.19 | −28.5% |

677 trades, **55% win rate**, median hold 57 days, worst trade −43.5%. It beat SPY in roughly four of five rolling 1-year and 3-year windows. In 2022 it returned +9.1% vs SPY −18.2%; at 30 bps/side it made 30.7%.

> Note: the locked headline above was regenerated after correcting the chandelier-stop ratchet, and again on 2026-11-19 through Nov 2026. Detailed historical comparison tables below should be regenerated before relying on their old point estimates.

## Why this beats the breakout version

Same pivot levels, opposite trigger. The breakout system buys strength *after* a level breaks, so it pays up and its stop sits far below the entry. This one buys the pullback *into* the level while the trend is intact, so the entry is near the level that defines the risk — a tighter, better-located stop. Then it uses the same "let winners run" exit. The result is a higher win rate (55% vs 43%) and a better Sharpe (1.23 vs 0.93). The two are only 0.67 correlated day-to-day, so a 50/50 blend has a better drawdown than either.

---

## ⚠️ Honest expectation — read before trading

The headline 33.9% is **not** what to expect. Four separate stress tests:

| Test | Result | Read |
|---|---|---|
| Equal-weight B&H of the same universe | **20.1%/yr** | survivorship-bias floor ≈ +6%/yr free |
| Delete the 12 biggest winners | **21.1%** | above SPY and the 20.1% bias floor |
| Fixed-2013 universe (no post-2013 IPOs) | **29.5%** vs 20.2% floor | +9.3%/yr of real edge |
| 25 random 60%-universe draws | median **25.4%**, min 20.0%, **25/25 beat SPY** | the edge is not one lucky name |
| ETF-only (bias-free) | **7.0%** vs SPY 14.2% | needs single-stock dispersion; does not work on indices |

The top 20 trades are 51% of the summed return, and 2024 alone was +137% on SMCI (+324%) and APP (+321%) — names the universe contains *because we know today they won*. The random-subsample test is the most trustworthy one, and it says ~25%, not 37%.

**Realistic forward expectation — superseded 2026-09-06/07.** The tests above discount against the *hand-picked* floor and leave the hindsight in. Re-run on a point-in-time S&P 500 panel with delisted names (`../fable_review/REPORT.md` §3.1) the same rules make **16.6%/yr @10 (Sharpe 0.76, −39% DD) vs SPY 14.9% (0.91)**, t < 1, beating SPY in 32% of rolling 3-year windows; @5 slots 15.7%. Deleting the 12 biggest point-in-time winners leaves 5.9%. **Expect roughly SPY ± 3%/yr at 1.45× the vol with a deeper drawdown, not SPY + 4–9%.** The book runs as tuition-sized exposure on that basis; do not add capital on backtest grounds. The strategy is essentially always fully invested (10/10 positions, 93% of days) with no market-timing filter, so it will take the full force of a bear market — 2022 was kind to it, and that may not repeat.

---

## Universe policy (decided 2026-09-07)

**Re-pick the list every January by a point-in-time rule; never hand-maintain it.** `../fable_review/UNIVERSE_REFRESH.md` ran the locked rules on the point-in-time panel under five universe policies. A list frozen in January 2013 makes **11.8%/yr @10 (Sharpe 0.58, −50% DD; @5: −62% DD)** against 16.6% refreshed — it tracks the refreshed list for about five years, then falls behind for good once it has missed a cycle of new leaders (TSLA, PLTR, SMCI, APP, CEG, VST were all added after 2019). Yearly refresh by index membership or top-N by 63-day median dollar volume lands at 13.7–19.2% (top-130 was the best cell at 19.2% / 0.94, but the size effect is non-monotonic and t = 1.41 — noise around 16%). The policy is second-order to the verdict above; freezing the list just adds a way to lose.

Practical rule: each January, rebuild `../breakout/universe.py` from current S&P 500 membership (or the top ~130–250 US names by dollar volume, plus the TSX names by the same rule), rerun `download.py`, and let existing positions run to their stops.

## Locked rules (v1 — do not tweak without a new backtest)

**Levels.** A pivot low is a bar whose low is the lowest of the 5 bars on each side (k=5), confirmed 5 bars later. **Support = the most recent confirmed pivot low** (the nearest one under price, not the lowest of the window). **Resistance = the highest pivot high of the last 120 trading days.**

**Entry (next open), max 10 positions, ~10% of equity each:**
1. The day's **low reaches into the support zone**: `low ≤ support + 1.0 × ATR(20)`,
2. but the **close finishes above support** — the level held,
3. and the close is **above the 200-day SMA** (buy dips in uptrends, not falling knives),
4. more candidates than slots → take the highest **6-month momentum** first.
5. **No green-bar confirmation** and **no market-regime filter** — both were tested and both *hurt* (waiting for an up-day costs 7%/yr; parking in IEF when SPY is below its 200SMA costs 12%/yr).

**Exit (next open) — the two-phase stop:**
- **Phase 1 (initial):** fixed stop at `support − 3 × ATR(20)`, measured at entry and never moved. Wide on purpose: a 2×ATR stop drops the CAGR to 29%.
- **Phase 2 (trailing):** once the close reaches the **resistance target**, the stop switches to a 3×ATR chandelier trail off the highest close, and the position is allowed to run.

Edge case: if the stock is already trading above its 120-day pivot resistance (a pullback inside an ongoing breakout), the target is met on day one and the trade trails from the start. That is 3% of trades (21 of 677) and clearly worse than average — 33% win rate vs 55% — so it is left in rather than special-cased.

That hybrid is where the return comes from: selling at the resistance target gives 23.6%, trailing from the start gives 17.8%, and the hybrid gives 32.5%. Stops averaged −14.4% (210 trades); trailing exits averaged +17.0% (467 trades).

**Sizing:** equity ÷ 10 per position, no pyramiding, no leverage. CAD names trade in CAD; the backtest is in USD. Live sizing is set in the local book config — the earlier 5-slot live config was worse on every metric on the honest universe.

### Tested and rejected: a sector cap

`run_support(..., sector_cap=N)` limits concurrent positions per sector (13 buckets in `../breakout/sectors.py`, with Semis/Software/Hardware and Gold/Materials split out). **It is off in the locked config because it costs money and does not reduce risk:**

| | CAGR | Sharpe | MaxDD | 2022 |
|---|---|---|---|---|
| No cap (locked) | **32.5%** | 1.23 | −33.6% | **+9.1%** |
| Cap 3 per sector | 27.7% | 1.11 | −35.8% | +9.2% |
| Cap 2 per sector | 31.3% | 1.28 | −35.5% | −6.5% |
| Cap 1 per sector | 26.0% | 1.24 | −31.8% | −11.3% |

A 3-cap costs **~4.8%/yr** and a 1-cap ~6.5%/yr; the 2-cap now costs only ~1.2%/yr and even posts a slightly better Sharpe — but it still gives back the 2022 edge below. The 2022 column shows why concentration was load-bearing: capping forced the capital into weaker sectors and turned a positive year negative.

The lesson: **sector concentration here is the output of the momentum ranking, not a bug.** When the book piles into one sector it is because those are the highest-momentum names holding support — the system has found the sector that works. Capping it caps the signal. Note also that the book is concentrated most of the time (81% of month-ends have >2 names in one sector), so a 2-cap binds constantly rather than acting as an occasional safety valve.

### Tested and rejected: exiting on a 200SMA break

`run_support(..., trend_exit=True)` closes a position when the close drops below its 200-day average — i.e. when the uptrend that qualified the entry is gone. Intuitive, and wrong:

| | CAGR | Sharpe | MaxDD | Win rate | 2022 |
|---|---|---|---|---|---|
| Locked (stop only) | **32.5%** | **1.23** | −33.6% | **55%** | **+9.1%** |
| + exit on 200SMA break | 28.4% | 1.11 | −34.9% | 49% | −3.4% |

It costs **~4.1%/yr** and *worsens* drawdown and win rate — the 200SMA-break exits are mostly shakeouts. The wide `support − 3×ATR` stop is deliberately placed to sit through exactly this: a broken level and a lost 200SMA are *not* exit conditions, only the stop is.

The honest caveat on the other side: 2013–2026 contains no episode where a concentrated book met a sector collapse. The cap is insurance against a tail that did not occur in-sample — but 4.6%/yr is a steep premium, and the per-position 3×ATR stop with 10% sizing already bounds the damage.

**Parameter plateau:** k=3/7, lookback 60/180, band 0.75/1.25, trail 2.5/4, 8/12 positions all land between 29% and 40%. Tuned only on 2013–2019, then checked on 2020–2026 — all 24 combinations in the grid beat SPY in *both* halves.

## Daily workflow

Web dashboard (recommended — covers both strategies):
```bash
cd strategy/dashboard && ../../.venv/bin/python app.py
```

Or the CLI:
```bash
cd strategy/support
../../.venv/bin/python signals.py            # candidates + exit checks + watchlist
../../.venv/bin/python signals.py enter CSCO --price 111.68 --shares 50
../../.venv/bin/python signals.py positions
../../.venv/bin/python signals.py exit CSCO --price 125.00
```

Run the scan **after the close**; enter and exit **at the next open**. `positions.json` holds the live state (entry price, initial stop and target are stored at entry so the phase logic is reproducible); closed trades append to `trades_log.csv`.

## Files

- `engine.py` — data loading, FX, pivot levels, RSI/ATR, metrics (reads the OHLCV cache in `../breakout/`)
- `backtest.py` — `run_support()` event-driven backtest + variant grid
- `robustness.py` — bias floor, sensitivity grid, cost stress, rolling windows, ETF-only, winner deletion
- `signals.py` — daily tool, exposes `scan()` for the dashboard
- `trade_report.py` — lists backtest trades over a recent window with entry/exit reasons (`--months 12`); also viewable in the dashboard's Backtest trades panel
- `compare_curves.pkl` — SPY / breakout / support / combined equity curves used by the dashboard chart

Universe and price data are shared with the breakout strategy: `../breakout/universe.py` and `../breakout/download.py` (rerun that to refresh prices for both).

## Relation to the other strategies

`../breakout/` (S/R breakout, 21.1% on the hand-picked list, **9.0% point-in-time — retired 2026-09-07**) and `../backtest/` (dual momentum, 30.0%) run on the same survivorship-heavy universe, so **they are not independent confirmation of each other**. If running more than one live, use separate sleeves and separate position files.
