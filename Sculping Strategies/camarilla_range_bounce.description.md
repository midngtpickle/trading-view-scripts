# 🌙🥒 Camarilla Range Bounce Scalper — Strategy Guide

*A Midnight Pickle 🌙🥒 Production*

Pine Script v6 mean-reversion strategy designed for 5m/15m Bitcoin trading during low-volatility/ranging market conditions.

---

## 1. Thesis & Concept

In sideways/ranging conditions, Camarilla R3 and S3 levels act as strong price reversion boundaries rather than breakout triggers. The R4 and S4 levels mark the boundaries where "the range has broken and a trend is starting."

- **Long Entries**: Fade touches of S3 when price is contained within the range.
- **Short Entries**: Fade touches of R3 when price is contained within the range.
- **Target**: Central pivot (PP) for TP1 (50%), opposite R3/S3 boundary for TP2.
- **Stop Loss**: Placed beyond S4/R4 (or ATR-based).

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
- **Regime Filters**: ADX < `adxMax`, price contained within R3/S3 for `containBars`, optional BB-width compression.
- **Touch Tolerance**: Price touches S3/R3 within `touchAtr` distance.
- **Rejection Candle**: Wick pierces the level, but bar closes back inside the boundary.
- **RSI Filter**: RSI at an extreme (< 35 for long / > 65 for short).
- **Invalidation**: Price has not already closed beyond S4 (for long) or R4 (for short).

### Exit Mechanics
- **TP1**: 50% scale-out at the central pivot (PP).
- **TP2**: Remaining 50% at the opposite R3/S3 level.
- **Stop Loss**: Beyond S4/R4 level or ATR multiplier.
- **Breakeven**: Stop loss pulls to entry price automatically after TP1 hits.
- **Range Invalidation**: Immediate exit if price closes beyond S4/R4.

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

The on-chart stats table tracks performance without win-rate inflation from scale-outs by snapshotting account equity per full trade cycle:

- **N**: Total full trade positions.
- **Win%**: Percentage of trades closing net positive.
- **Avg R**: Mean R-multiple per trade (**primary metric**).
- **Net R**: Total cumulative R-multiple generated.

### Breakdown Categories
- **Regime (ADX at Entry)**: Compares performance across low (<15), waking (15-20), and high ADX buckets to confirm ranging edge.
- **Session (UTC)**: Evaluates performance across Asia (00:00–07:00 UTC), London (07:00–13:00 UTC), and NY (13:00–21:00 UTC).
- **Direction**: Compares Long vs Short metrics to identify directional bias or liquidation cascade asymmetric risk.

---

## 6. Best Practices & Troubleshooting

- **Walk-Forward Testing**: Fit parameters on the first 60% of data, freeze settings, and test on the remaining 40% to prevent curve-fitting.
- **Non-Repainting Security**: Uses `request.security(..., expr[1], lookahead = barmerge.lookahead_on)` with `[1]` offset to prevent future data leakage.
- **Zero Trades?**: Loosen `adxMax` or `touchAtr`.
- **Too Many Trades?**: Tighten `touchAtr` or lower `maxPerDay`.
