# MP EMA 9/21 Pullback Continuation — Strategy Guide

*A Midnight Pickle (MP) Production*

Pine Script v6 trend-following strategy designed for 5m/15m Bitcoin trading during high-volatility/trending market conditions.

---

## 1. Thesis & Concept

In established trends, pullbacks to the fast EMA offer low-risk entries in the direction of the macro movement. This strategy trades the **resumption of the trend**, rather than blindly catching a falling or rising pullback bar.

- **Long Entries**: Uptrend regime + pullback to 9 EMA + resumption candle close through 9 EMA.
- **Short Entries**: Downtrend regime + pullback to 9 EMA + resumption candle close through 9 EMA.
- **Target**: 50% TP1 at 1R, trailing runner (Chandelier stop default) for the remaining position.
- **Stop Loss**: Placed beyond the pullback swing extreme (+ ATR buffer).

> **Regime Context**: This strategy relies on ADX being **above** ~22 (`adxMin`). It is the structural counterpart to the Camarilla Range Bounce strategy.

---

## 2. TradingView Setup & Execution

1. Open a BTC chart (`BINANCE:BTCUSDT.P` or `BINANCE:BTCUSDT` — perpetual futures provide realistic scalping conditions).
2. Set the timeframe to **5m** or **15m**.
3. Open the **Pine Editor** tab at the bottom panel.
4. Paste the full contents of [`ema_pullback_continuation.pine`](ema_pullback_continuation.pine).
5. Click **Save**, name it, then click **Add to chart**.

---

## 3. Configuration & Timeframe Calibration

Before evaluating backtest results in the Strategy Tester, ensure these parameters are calibrated:

- **Commission**: Default is `0.045%`. Set this to your actual exchange taker fee tier.
- **Slippage**: Default is `2` ticks. Do not set this to zero.
- **Order Fill Timing**: `process_orders_on_close = true` ensures conservative entry timing at bar close.

### Timeframe-Specific Settings

| Setting | 5m Chart | 15m Chart | Description |
|---|---|---|---|
| `htf` | `60` (1 Hour) | `240` (4 Hour) | Higher timeframe trend filter |
| `armExpiry` | 12 bars | 8 bars | Bars before un-triggered setup disarms |
| `adxMin` | 22.0 | 25.0 | Minimum ADX trend strength threshold |

---

## 4. Strategy Rules & State Machine

The entry logic operates on a 3-state machine. A faint teal/red background marks the trend regime; a stronger tint marks an armed setup.

| State | Condition | Visual Indicator |
|---|---|---|
| **ARM** | Trend intact & low (long) / high (short) comes within `touchAtr` (default 0.20 ATR) of the 9 EMA | Strong background tint |
| **FIRE** | Candle closes back beyond the 9 EMA in the trend direction, with a green (long) / red (short) body | Entry triangle marker |
| **DISARM** | Close goes more than `deepStop` (default 0.35 ATR) past the 21 EMA, the trend conditions stop holding, or `armExpiry` (default 12) bars pass | Strong tint clears |

### Entry Conditions (All must be true)
- **EMA Trend**: 9 EMA above (long) / below (short) the 21 EMA, and the 21 EMA rising / falling over the last `slopeLen` (default 3) bars.
- **HTF Alignment** (default on): Close above (long) / below (short) the 50 EMA of the previous completed `htf` bar (default `60`; use `240` on 15m).
- **Regime Filters**: ADX(14) > `adxMin` (default 22), and the 9/21 gap ≥ `separAtr` (default 0.15 ATR).
- **Resumption**: Setup is armed by a pullback touch, then triggered by a resumption close. Optional stricter trigger: close beyond the prior bar's high/low (`needEngulf`, off by default).
- **Limits**: Max `5` entries per day. Optional session filter (off by default; `0700-1600` Europe/London when on). Longs and shorts can be disabled separately.

### Exit Mechanics
- **TP1**: `tp1Pct` (default 50%) scale-out at `tp1R` (default 1.0R).
- **Runner Exits**: Chandelier (default: highest high / lowest low of 10 bars ∓ 2.0 ATR), Slow EMA (21 EMA ∓ the swing buffer), or `Fixed R` target at `tp2R` (default 2.5R). Trailing stops only ever tighten.
- **Stop Loss**: `Swing` mode (default) sits beyond the pullback's extreme plus a 0.15 ATR buffer. `ATR` mode uses 1.5 × ATR(14) from entry.
- **Breakeven**: Stop pulls to entry after TP1 (default on).
- **Trend Flip**: Full exit if 9/21 EMAs cross in reverse direction.
- **Time Stop**: Optional, off by default (`0`).

---

## 5. Performance Analytics Table

The on-chart stats table measures each **position** once, from open to flat, so TP1 scale-outs aren't counted as separate wins. R is the net P&L divided by the initial cash risk (entry to original stop, before any breakeven or trail move):

- **N**: Total full trade positions.
- **Win%**: Percentage of positions closing net positive.
- **Avg R**: Mean R-multiple per trade (**primary metric**).
- **Net R**: Total cumulative R-multiple generated.
- **Best Trade**: Displays largest single trade R-multiple and its percentage of net profit to audit outlier reliance.

### Breakdown Categories
- **Regime (ADX at Entry)**: Weak (<20), Trending (20–30), Strong (30+). Checks whether strong trends actually outperform.
- **Session (UTC)**: Asia (00–07), London (07–13), NY (13–21), Late (21–24).
- **Direction**: Long (bull pullback) vs Short (bear rally) asymmetry.
- **Total**: All trades, plus trades per day and the number of days in the sample.

---

## 6. Best Practices & Troubleshooting

- **Runner Exit Importance**: Trend strategies depend on outlier trades. Avoid fixed TP caps that choke large multi-R runners.
- **Walk-Forward Testing**: Validate on out-of-sample data splits to verify edge stability.
- **Non-Repainting Security**: Built with `request.security(..., expr[1], lookahead = barmerge.lookahead_on)` using `[1]` offset for historical integrity.
- **Zero Trades?**: Check if HTF filter is too strict or decrease `adxMin`.
