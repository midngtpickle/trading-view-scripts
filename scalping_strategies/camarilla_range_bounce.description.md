# MP Camarilla Range Bounce Scalper — Strategy Guide

*A Midnight Pickle (MP) Production*

Pine Script v6 mean-reversion strategy designed for 5m/15m Bitcoin trading during low-volatility/ranging market conditions.

---

## 1. Thesis & Concept

In sideways/ranging conditions, Camarilla R3 and S3 levels act as strong price reversion boundaries rather than breakout triggers. The R4 and S4 levels mark the boundaries where "the range has broken and a trend is starting."

- **Long Entries**: Fade touches of S3 when price is contained within the range.
- **Short Entries**: Fade touches of R3 when price is contained within the range.
- **Target**: Central pivot (PP) for TP1 (50%), opposite R3/S3 boundary for TP2.
- **Stop Loss**: Placed beyond S4/R4 (or ATR-based).

> [!WARNING]
> **Textbook Camarilla levels.** This strategy uses the standard mapping: R3 = $C + 1.1 \times range / 4$ ($H_3$) and R4 = $C + 1.1 \times range / 2$ ($H_4$), from the previous `Pivot timeframe` period (default `D`). The [Pivots + Key Levels](../Master%20Suite%20Indicators/pivots_and_key_levels_master.description.md) indicator shifts everything one step, so **its R3 is this strategy's R4**. Use `Plot levels` on this script to see the levels it actually trades.

> **Regime Context**: This strategy relies on ADX being **below** ~22 (`adxMax`). Running it in strong trending markets will result in losses as price breaks through R3/S3 to R4/S4.

---

## 2. TradingView Setup & Execution

1. Open a BTC chart (`BINANCE:BTCUSDT.P` or `BINANCE:BTCUSDT` — perpetual futures provide realistic scalping conditions).
2. Set the timeframe to **5m** or **15m**.
3. Open the **Pine Editor** tab at the bottom panel.
4. Paste the full contents of [`camarilla_range_bounce.pine`](camarilla_range_bounce.pine).
5. Click **Save**, name it, then click **Add to chart**.

---

## 3. Configuration & Timeframe Calibration

Before evaluating backtest results in the Strategy Tester, ensure these parameters are calibrated:

- **Commission**: Default is `0.045%`. Set this to your actual exchange taker fee. At 5m scalping frequency, fee drag significantly impacts net profitability.
- **Slippage**: Default is `2` ticks. Do not set this to zero; slippage on market orders during volatile candles must be accounted for.
- **Order Fill Timing**: `process_orders_on_close = true` ensures entries fill at the close of the signal bar.

### Timeframe-Specific Settings

| Setting | 5m Chart | 15m Chart | Description |
|---|---|---|---|
| `containBars` | 20 | 12 | Price lookback containment within R3/S3 |
| `adxMax` | 22.0 | 25.0 | Max ADX threshold (above this, entries are blocked) |
| Time Stop | 30–60 bars | 0 (off) or 20 | Optional bar limit for stagnant range trades |

---

## 4. Strategy Rules & Logic

### Entry Conditions (All must be true)
- **Regime Filters**: ADX(14) < `adxMax` (default 22), no close outside R3/S3 for the last `containBars` (default 20), optional BB-width percentile below `bbwPct` (off by default).
- **Touch Tolerance**: Low comes within `touchAtr` (default 0.25 ATR) of S3, or high within that distance of R3. Price doesn't have to pierce the level.
- **Rejection Candle** (default on): For longs, the bar closes above S3 **and** closes green. For shorts, it closes below R3 and closes red.
- **RSI Filter** (default on): RSI(14) < 35 for longs / > 65 for shorts.
- **Invalidation**: The bar has not closed beyond S4 (long) or R4 (short).
- **Limits**: Max `4` entries per day (resets on the daily bar). Optional session filter (off by default; `0700-1600` Europe/London when on). Longs and shorts can be disabled separately.

### Exit Mechanics
- **TP1**: `tp1Pct` (default 50%) scale-out at the central pivot (PP).
- **TP2**: Remainder at the opposite R3/S3 level (or PP too if `Use TP2` is off).
- **Stop Loss**: `S4/R4` mode (default) uses S4, or one tick beyond the signal bar's wick if that is further. `ATR` mode uses 1.5 × ATR(14) from entry.
- **Breakeven**: Stop pulls to entry after TP1 is hit (default on).
- **Range Invalidation**: Immediate exit if price closes beyond S4/R4.
- **Time Stop**: Optional, off by default (`0`).

A light teal background marks bars where the full regime filter passes.

### Structural R:R Caveat
Because TP1 is at PP and stops are beyond S4/R4, the Risk:Reward ratio on TP1 is often below 1:1. This produces a high win rate, but performance must be evaluated on **Avg R**, not raw win rate.

### Swapping Custom Pivot Math
The Camarilla formula block is isolated in the script:
```pinescript
// PIVOT CALCULATION  ◀── SWAP YOUR CUSTOM MATH IN HERE, NOTHING ELSE CHANGES
```
You can replace the `R1`–`R4` / `S1`–`S4` / `PP` variable assignments with custom formulas without altering downstream logic.

---

## 5. Performance Analytics Table

The on-chart stats table measures each **position** once, from open to flat, so TP1 scale-outs aren't counted as separate wins. It snapshots `strategy.netprofit` at open and at flat and divides by the initial cash risk (entry to the original stop, before any breakeven move):

- **N**: Total full trade positions.
- **Win%**: Percentage of positions closing net positive.
- **Avg R**: Mean R-multiple per trade (**primary metric**).
- **Net R**: Total cumulative R-multiple generated.

### Breakdown Categories
- **Regime (ADX at Entry)**: Dead flat (<15), Quiet (15–20), Waking (20+). Confirms whether the edge really lives in the quietest conditions.
- **Session (UTC)**: Asia (00–07), London (07–13), NY (13–21), Late (21–24).
- **Direction**: Long (S3 bounce) vs Short (R3 fade), to spot directional bias or liquidation-cascade asymmetry.
- **Total**: All trades, plus trades per day and the number of days in the sample.

---

## 6. Best Practices & Troubleshooting

- **Walk-Forward Testing**: Fit parameters on the first 60% of data, freeze settings, and test on the remaining 40% to prevent curve-fitting.
- **Non-Repainting Security**: Uses `request.security(..., expr[1], lookahead = barmerge.lookahead_on)` with `[1]` offset to prevent future data leakage.
- **Zero Trades?**: Loosen `adxMax` or `touchAtr`.
- **Too Many Trades?**: Tighten `touchAtr` or lower `maxPerDay`.
