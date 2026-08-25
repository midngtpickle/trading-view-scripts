# 🌙🥒 BTC Scalping Strategy Pair — Setup & Evaluation Guide

*A Midnight Pickle 🌙🥒 Production*

Two Pine Script v6 strategies designed as a **regime pair** for 5m/15m Bitcoin:

| File | Strategy | Wants | Logic type |
|---|---|---|---|
| `camarilla_range_bounce.pine` | Camarilla Range Bounce | ADX **below** ~22 | Mean reversion |
| `ema_pullback_continuation.pine` | EMA 9/21 Pullback | ADX **above** ~22 | Trend following |

They are structural opposites on purpose. Each will look broken during the other's best conditions — that is the design, not a bug. If both ever show green regime shading simultaneously, a filter is misconfigured.

---

## 1. Loading them into TradingView

1. Open a BTC chart (`BINANCE:BTCUSDT.P` or `BINANCE:BTCUSDT` — perps give you more realistic scalping conditions).
2. Set the timeframe to **5m** or **15m**.
3. Open **Pine Editor** (bottom panel).
4. `Open` → `New blank strategy`, delete the boilerplate.
5. Paste the full contents of one `.pine` file.
6. Click **Save**, name it, then **Add to chart**.
7. Repeat in a second Pine Editor tab for the other strategy.

Results appear in the **Strategy Tester** tab. The custom stats table renders top-right on the chart itself.

> **Run them one at a time.** Two strategies on one chart share the same equity model in the Strategy Tester and the readings become meaningless.

---

## 2. Configure these BEFORE reading any results

Three settings will silently invalidate everything if left wrong:

### Commission
Default is `0.045%`. Set this to **your actual taker fee**. At 5m scalping frequency, fee drag is frequently the entire difference between a green and red equity curve.

### Slippage
Default is `2` ticks. **Do not set this to zero.** Zero-slippage backtests on 5m crypto are fantasy. If anything, raise it — BTC perps during volatile sessions will give you worse than 2 ticks on market orders.

### Order fill timing
Both scripts use `process_orders_on_close = true`, meaning entries fill at the close of the signal bar. This is the conservative, honest setting. Leave it.

### Timeframe-specific adjustments

| Setting | 5m chart | 15m chart |
|---|---|---|
| **Range Bounce** — `containBars` | 20 | 12 |
| **Range Bounce** — `adxMax` | 22 | 25 |
| **Range Bounce** — time stop | 30–60 | 0 (off) or 20 |
| **EMA Pullback** — `htf` | `60` | `240` |
| **EMA Pullback** — `armExpiry` | 12 | 8 |

Leaving the EMA Pullback HTF filter at 60 on a 15m chart makes it nearly meaningless — the HTF is barely higher than the chart timeframe.

---

## 3. Strategy 1 — Camarilla Range Bounce

### The thesis
In ranging conditions, Camarilla R3/S3 behave as reversion boundaries rather than breakout triggers. R4/S4 mark "the range just died." So: fade S3 for longs, fade R3 for shorts, target the central pivot, stop beyond S4/R4.

### Entry conditions (all must be true)
- Regime filters pass: ADX < `adxMax`, price contained within R3/S3 for `containBars`, optional BB-width compression
- Price touches the level within `touchAtr` tolerance
- Rejection candle: wick through the level, close back inside
- RSI at an extreme (< 35 long / > 65 short)
- Price hasn't already closed beyond S4/R4

### Exits
- **TP1** — 50% at the central pivot (PP)
- **TP2** — remainder at the opposite R3/S3
- **Stop** — beyond S4/R4 (or ATR-based)
- **Breakeven** — stop pulls to entry after TP1
- **Range invalidation** — full close if price closes beyond S4/R4

### The structural caveat you must account for
With TP1 at PP and stops out at S4/R4, **your R:R on the first target is often below 1:1**. This produces a flattering high win rate that does not mean the strategy is profitable. Judge it on **Avg R**, never on win rate.

### Swapping in your own pivot math
The Camarilla calculation is isolated in a clearly-marked block:

```
// PIVOT CALCULATION  ◀── SWAP YOUR CUSTOM MATH IN HERE, NOTHING ELSE CHANGES
```

Replace the `R1`–`R4` / `S1`–`S4` / `PP` assignments with your own formulas. Nothing downstream needs touching, as long as your variables keep the same names.

---

## 4. Strategy 2 — EMA 9/21 Pullback Continuation

### The thesis
In established trends, pullbacks to the fast EMA offer entries with defined risk. Trade the resumption, not the pullback itself.

### The state machine
This is not a single-bar condition — it has three states, and both are visible as chart shading:

| State | What happens | Chart |
|---|---|---|
| **ARM** | Trend intact + price pulls back and touches the 9 EMA | Strong tint |
| **FIRE** | Price resumes with a confirming close back through the 9 | Triangle marker |
| **DISARM** | Pullback closes too far past the 21 EMA (it's a reversal now), or expires after `armExpiry` bars | Shading clears |

Faint background tint = trend regime active. Watching these two shading layers on the chart is the fastest way to sanity-check the logic before trusting any numbers.

### Entry conditions
- 9 EMA above/below 21 EMA, 21 EMA sloping in the trend direction
- Price on the correct side of the HTF EMA
- ADX > `adxMin` (trending)
- EMAs sufficiently separated (`separAtr`) — rejects tangled/chopping EMAs
- Setup armed, then a resumption candle fires it

### Exits
- **TP1** — 50% at 1R
- **Runner** — Chandelier trail by default (also: Slow EMA trail, or fixed 2.5R)
- **Stop** — beyond the pullback swing extreme, plus buffer
- **Trend flip** — full close if the 9/21 cross reverses

### Why the runner exit default matters more than anything else
Trend strategies make their money on a small number of outliers. Capping exits at a fixed 2.5R cuts off exactly the trades that pay for all the losers. That's why `trailMode` defaults to **Chandelier**, and why the stats table has a dedicated **"Best trade"** row.

---

## 5. Reading the stats table

Both scripts render the same table. Columns:

| Column | Meaning |
|---|---|
| **N** | Number of positions (not exit orders) |
| **Win%** | Percentage of positions closing net positive |
| **Avg R** | Mean R-multiple — **this is the column that matters** |
| **Net R** | Total R contributed by that bucket |

### Why the R accounting is custom
Pine treats every scale-out as its own "closed trade." Reading win rate straight off the built-in Strategy Tester would count each TP1 fill as a separate win and inflate the number badly.

Instead, both scripts snapshot `strategy.netprofit` when a position opens and again when it goes flat, then divide by the **initial** cash risk — entry to the original stop, *before* any breakeven or trail adjustment. A trade that takes TP1 and then stops out at breakeven shows as one small win, which is what actually happened to your account.

### Bucket groups

**REGIME (ADX at entry)** — the honesty check for the core thesis.
- *Range Bounce:* if "Dead flat <15" clearly beats "Waking 20+", the thesis holds — tighten `adxMax`. If they're indistinguishable, ADX isn't doing what the strategy assumes and the filter is just shrinking your sample.
- *EMA Pullback:* expect the reverse. "Strong 30+" should outperform. If it doesn't, the trend premise is weaker than assumed on this symbol/timeframe.

**SESSION (UTC)** — where the edge actually lives.
- Asia 00–07 typically chops and punishes trend strategies
- London 07–13 and NY 13–21 carry most real liquidity and volatility
- If one session is carrying the entire net, consider hard-restricting to it via `useSess`

**DIRECTION** — watch for asymmetry.
BTC downside moves are faster than upside (liquidation cascades). Expect shorts to behave differently from longs even where the logic is symmetric. Range Bounce R3 fades in particular often get stopped more violently than S3 bounces.

**Best trade (EMA Pullback only)** — largest single R and what % of net it represents.
If one trade is 40%+ of your net, you don't have a strategy, you have a lucky sample.

---

## 6. Evaluation checklist

Work through these in order. Stop at the first failure — there's no point tuning past a broken fundamental.

- [ ] **Sample size.** Under ~30 trades in a bucket is noise. With 3 regime × 4 session × 2 direction buckets you're slicing twelve ways — you will find a spuriously excellent-looking cell almost guaranteed.
- [ ] **Average trade net of commission.** If it's under roughly 2× your round-trip cost, the "edge" is inside the noise band of your fee model.
- [ ] **Profit factor.** Below ~1.2 on a backtest usually means below 1.0 live.
- [ ] **Max consecutive losses.** Can you actually sit through that streak without changing the rules? If not, the strategy is untradeable regardless of its expectancy.
- [ ] **Equity curve shape.** Smooth and grinding, or three outlier trades and a flat line? Check the "Best trade" row.
- [ ] **Max drawdown vs. your tolerance.** Not the backtest's tolerance. Yours.

---

## 7. Avoiding self-deception

The single fastest way to produce a beautiful, worthless backtest is to tune parameters until the totals look good. You've then fit the strategy to that specific window.

**Walk-forward split:**
1. Set your chart date range to the **first 60%** of your data.
2. Tune `adxMax` / `containBars` / `armExpiry` / `touchAtr` until you're happy.
3. **Freeze the settings. Change nothing.**
4. Move the range to the **last 40%**.
5. Compare the two tables.

If out-of-sample performance collapses, you curve-fitted. This is the normal outcome, not a failure of process — it's the process working.

**Also worth testing:**
- Different market epochs (2021 bull, 2022 bear, 2023 chop) rather than one continuous run
- A second symbol (ETH) with identical settings — genuine edges usually generalize at least partially
- Deliberately *worse* fee and slippage assumptions, to find where the edge breaks

---

## 8. Known limitations

- **Backtest fills are optimistic by nature.** Limit-order TP fills assume you got filled at the level; in reality you may not have.
- **No funding rate modelling.** On perps held across funding intervals this is a real, unmodelled cost.
- **Pine's intrabar resolution is limited.** On a 5m chart, if a bar hits both your stop and your target, Pine has to guess the order. Use `calc_on_order_fills` or bar-magnifier if you want stricter fill assumptions.
- **Neither strategy detects regime *transitions*.** They detect regime *state*. The bars where a range breaks into a trend are the most expensive bars for the Range Bounce strategy, and it will take those losses.
- **Both are unvalidated starting points.** They encode common, widely-documented setups — not proven edges. The backtest is how you find out whether any edge exists on your symbol, your timeframe, and your fee tier.

---

## 9. Troubleshooting

| Symptom | Likely cause |
|---|---|
| Zero trades | Regime filters too strict — loosen `adxMax` / `adxMin` first, then `touchAtr` |
| Far too many trades | `touchAtr` too generous, or `maxPerDay` too high |
| Wildly good results | Check commission and slippage aren't zeroed; check you didn't remove the `[1]` offset from `request.security` |
| Levels not plotting | `pivotTF` set below chart timeframe, or insufficient history loaded |
| Table shows all dashes | No closed positions yet in the visible range |
| Results change on reload | Repainting — verify the `[1]` offset is intact in the HTF/pivot requests |

---

## 10. On the non-repainting pattern

Both scripts use this form:

```pinescript
request.security(syminfo.tickerid, tf, expr[1], lookahead = barmerge.lookahead_on)
```

The `[1]` offset is what makes `lookahead_on` safe. It requests the **previous completed** period, so lookahead cannot leak unformed future data. Remove the offset and the backtest will look spectacular and the live results will not. If you modify these calls, keep the offset.

---

*Not financial advice. These are research tools for measuring whether an idea has an edge — not a recommendation to trade one.*
