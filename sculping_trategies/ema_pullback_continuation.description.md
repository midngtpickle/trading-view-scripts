# 🌙🥒 EMA 9/21 Pullback Continuation — Strategy Guide

*A Midnight Pickle 🌙🥒 Production*

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

The entry logic operates on a 3-state machine visible via chart background shading:

| State | Condition | Visual Indicator |
|---|---|---|
| **ARM** | Trend intact & price touches 9 EMA pullback zone | Strong background tint |
| **FIRE** | Resumption candle closes back through 9 EMA | Entry triangle marker |
| **DISARM** | Price breaches 21 EMA (reversal) or `armExpiry` bars pass | Background shading clears |

### Entry Conditions (All must be true)
- **EMA Trend**: 9 EMA separated from 21 EMA, 21 EMA sloping in trend direction.
- **HTF Alignment**: Price aligned with 50 HTF EMA (60m on 5m chart, 240m on 15m chart).
- **Regime Filters**: ADX > `adxMin` (trending market), 9/21 EMA separation > `separAtr` (rejects chop).
- **Resumption**: Setup is armed by a pullback touch, then triggered by a resumption close.

### Exit Mechanics
- **TP1**: 50% scale-out at 1.0 R-multiple.
- **Runner Exits**: Chandelier trailing stop (default), Slow EMA trail, or fixed target (2.5R).
- **Stop Loss**: Beyond the pullback swing high/low + ATR buffer.
- **Trend Flip**: Full exit if 9/21 EMAs cross in reverse direction.

---

## 5. Performance Analytics Table

The on-chart stats table tracks performance without win-rate inflation from scale-outs:

- **N**: Total full trade positions.
- **Win%**: Percentage of positions closing net positive.
- **Avg R**: Mean R-multiple per trade (**primary metric**).
- **Net R**: Total cumulative R-multiple generated.
- **Best Trade**: Displays largest single trade R-multiple and its percentage of net profit to audit outlier reliance.

### Breakdown Categories
- **Regime (ADX at Entry)**: Verifies that high ADX (>30) outperforms weak ADX.
- **Session (UTC)**: Slices performance by Asia (00:00–07:00 UTC), London (07:00–13:00 UTC), and NY (13:00–21:00 UTC).
- **Direction**: Audits Long vs Short performance asymmetry.

---

## 6. Best Practices & Troubleshooting

- **Runner Exit Importance**: Trend strategies depend on outlier trades. Avoid fixed TP caps that choke large multi-R runners.
- **Walk-Forward Testing**: Validate on out-of-sample data splits to verify edge stability.
- **Non-Repainting Security**: Built with `request.security(..., expr[1], lookahead = barmerge.lookahead_on)` using `[1]` offset for historical integrity.
- **Zero Trades?**: Check if HTF filter is too strict or decrease `adxMin`.
