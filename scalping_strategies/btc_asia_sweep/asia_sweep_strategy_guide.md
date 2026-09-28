# MP BTC Asia Range Sweep & Reclaim — Strategy Guide v1

Companion doc for `midnight_pickle_asia_sweep_v1.pine`.

**Scope: BTC perps only.** See §9 — the parameter defaults, fee arithmetic and ATR thresholds are calibrated to BTC and don't transfer.

This supersedes the NZ-local-time version of the guide. The changes are listed in §1 so you can see exactly what moved and why.

---

## 1. What changed from your draft

| # | Issue in the original | Change |
|---|---|---|
| 1 | NZDT column assumed NZ and US DST move together | **All timing re-anchored to New York.** NZ shifts on 27 Sep; the US doesn't shift until 1 Nov. For roughly five months of the year the original NZDT column was an hour early — putting the first hour of the "Asia Box" inside live US session. |
| 2 | Tokyo lunch converted into NZ time | Anchored to `Asia/Tokyo` directly. Japan has no DST, so a Tokyo anchor never drifts; an NZ or NY anchor does. |
| 3 | "Early Asian market buildup" during the Power Window | Relabelled. 18:00–20:00 ET is the **post-NY vacuum** — Tokyo cash doesn't open until 00:00 UTC. The premise still works (thin books sweep easily) but the label was wrong, and it hid a testable variant: shift the window +1h/+2h to catch the actual Tokyo open. |
| 4 | Flat 0.2% stop | Now `max(0.2%, 0.35×ATR)` by default, mode-switchable. See §4 — this is the single change most likely to decide whether the strategy is viable. |
| 5 | Risk was unbounded (stop = wick extreme + 0.2%, entry = reclaim close) | Added `maxRiskPct` skip rule and risk-based position sizing. Skipped trades are counted so you can see how often it bites. |
| 6 | "Aggressive wick" / "fails to hold" undefined | Now `minPenAtr` / `maxPenAtr` — a sweep must be deep enough to have taken stops, and shallow enough to still be a sweep rather than a break. |
| 7 | "Long Trap" / "Short Trap" naming | Renamed **Sweep-Low-Long** / **Sweep-High-Short**. A "long trap" conventionally means a trap *for* longs, i.e. a short setup — the original names meant the opposite of how they read. |
| 8 | Missing ticker tokens in §1 and §5 | Restored. |
| 9 | §3 headed "BTC-Specific" but half about SOL | Removed. **This is a BTC-only strategy.** All parameter defaults, ATR ratios and fee assumptions below are calibrated to BTC perps and should not be assumed to transfer. |
| 10 | Day-of-week not specified | Default mask is **Sun–Thu ET**, which is Mon–Fri mornings NZ. The NZ Monday session starts Sunday afternoon in New York. |

---

## 2. The clock (New York anchored)

| ET | UTC (US winter) | UTC (US summer) | Phase | Action |
|---|---|---|---|---|
| 16:00 | 21:00 | 20:00 | US settlement | Institutional volume exits. BTC enters low-volume consolidation. |
| 16:15 | 21:15 | 20:15 | Heatmap check | Coinglass/Hyblock. Mark high-density liquidation clusters outside the forming range. |
| 16:00–18:00 | — | — | **Asia Box** | Mark high and low of this 2h window on the 15M chart. |
| 18:00–20:00 | — | — | **Power Window** | Hunt the sweep and reclaim. |
| 11:30–12:30 JST | 02:30 | 02:30 | **Tokyo lunch** | Volatility drops. No new entries. Manage only. |

Convert to NZ at your end — but never store NZ times in the script. The market follows US DST.

**Day mask:** `12345` in Pine = Sun, Mon, Tue, Wed, Thu (1 = Sunday). Those five ET afternoons are your five NZ weekday mornings.

---

## 3. The setup

**Step 1 — Asia Box.** High and low of 16:00–18:00 ET. Mid-line is the 50% equilibrium.

**Step 2 — Sweep.** During the Power Window, price trades outside the box:
- **Sweep-Low-Long:** price breaks the Asia Low, shorts pile in, the break fails.
- **Sweep-High-Short:** price breaks the Asia High, FOMO longs trigger, the break fails.

The sweep must be **≥ 0.15 ATR** deep (shallower = nobody's stops were taken) and **≤ 1.50 ATR** deep (deeper = the range is genuinely broken; the script disarms rather than fading it).

**Step 3 — Reclaim.** A 5M candle closes back inside the box. Same-bar reclaims count — a bar that wicks below and closes inside is the classic version.

**Step 4 — Confirmation.** Divergence, in one of two modes (§5).

---

## 4. Risk — read this before the first backtest

### The fee arithmetic

Round-trip taker on BTC perps is roughly **0.09–0.10% of notional**. If your risk is 0.2% of notional:

```
fee drag = 0.095 / 0.2 = ~0.48R per trade
```

Nearly half a unit of risk gone before the trade has an opinion. At 1:1 you'd need a ~60% win rate just to break even on costs. That is not impossible for a mean-reversion setup, but it means the strategy has almost no room for error.

Three ways out, in order of preference:

1. **Widen the stop** (default: `max(0.2%, 0.35×ATR)`). Halves the fee drag in R terms. Costs you position size, not expectancy.
2. **Enter maker-side.** Not modelled here — Pine can't reliably simulate limit-fill probability at the reclaim level.
3. **Accept it and demand a bigger edge.** Fine, as long as it's a decision rather than an oversight.

The `Percent` stop mode is left in so you can measure the difference yourself. Run the same window under `Percent` and `Max of both` and compare net R. If `Percent` wins, ignore everything above.

### Risk cap

Entry is the reclaim close; the stop sits beyond the sweep wick. A deep wick means a wide stop, and the distance is not bounded by anything. `maxRiskPct` (default 1.2% of price) skips those trades. The footer row shows the skip count — if it's a large fraction of signals, your `maxPenAtr` is too permissive.

### Targets

- **TP1** — box mid. Takes 50% off, moves the stop to breakeven **+0.05%** (true breakeven is a loss after fees).
- **TP2** — opposite box edge.

---

## 5. Divergence — what got finished

Ported from `midnight_pickle_divergence_v1.pine`. Three items that were open in v1 are now closed:

- **Multi-occurrence pivot scan.** v1 compared only `ta.valuewhen(..., 1)` — the immediately previous pivot. Real divergences frequently skip a pivot. Now scans occurrences 1–3, with an explicit bar-distance gate replacing the old `_inRange` helper (which only ever measured the gap to occurrence 1 anyway).
- **Price pivot verification.** v1 sampled `low[lbR]` at the *oscillator's* pivot bar without checking price had a swing there. Now requires a confirmed price pivot on the same bar.
- **Freshness gate.** A 40-bar-old divergence and a 3-bar-old one are not the same trade. `divFresh` caps the age at entry.

### The lag problem, and the new mode

Pivot divergence confirms `lbR` bars **after** the pivot. On 5M with `lbR = 5` that's 25 minutes. The Power Window is 120 minutes. The reclaim does not wait.

So there's a third mode — **Box-anchored**, the default:

> At the sweep extreme, price made a lower low relative to the box low. Did the oscillator make a lower low relative to *its own minimum during the box window*? If not, that's a divergence — and it's known the instant the sweep prints.

Zero lag, genuinely a divergence, and anchored to the same structure the strategy is already trading. Mode options:

| Mode | Lag | Use when |
|---|---|---|
| `Off` | — | Baseline run. Start here. |
| `Box-anchored` | none | Default. Live-tradeable. |
| `Pivot (strict)` | `lbR` bars | Research — measuring how much the classic method actually adds once you pay the lag. |

The full six-oscillator selector (RSI / Stoch / MFI / CCI / MACD Hist / OBV) and the adaptive percentile zones for the unbounded ones came across intact. Note the location gate here offers `Off / Current pivot / Both pivots` — the standalone scanner's `Either pivot` mode was dropped as it doesn't map cleanly onto the box-anchored logic.

---

## 6. Liquidity proxies — the honest caveat

**Pine has no liquidation heatmap feed.** Coinglass and Hyblock are not available to `request.security` at any price. This matters because the heatmap read is arguably the core of your discretionary edge.

Three proxies stand in, all off by default:

| Proxy | Setting | What it approximates |
|---|---|---|
| Sweep depth in ATR | `minPenAtr` / `maxPenAtr` | Whether a meaningful stop cluster was reached |
| Volume spike on the sweep bar | `useVolSpk` | Liquidation cascade signature |
| Proximity to prior-day H/L | `usePdLevel` | Known resting-stop magnets |

They are correlated with liquidation density. They are not a measurement of it.

**The implication is the important part:** if the mechanical backtest is flat but your discretionary results aren't, the difference is the heatmap read — meaning your edge is a skill you can't validate statistically and can't automate. That's a legitimate position, but you should hold it knowingly rather than mistake a flat backtest for a broken strategy.

---

## 7. Setup and settings that invalidate results

1. **Chart timeframe: 5M** (1M/3M also fine). The script warns above 15M — on higher timeframes the sweep and reclaim collapse into one bar and the backtest is fiction.
2. **Properties → Commission:** `0.045%` per side, type Percent. Pre-set in code, but verify it survived.
3. **Properties → Slippage:** `2` ticks. Also pre-set.
4. **Properties → Recalculate:** leave "on every tick" **off**. `process_orders_on_close = true` is doing deliberate work.
5. **Symbol:** `BINANCE:BTCUSDT.P` or `BYBIT:BTCUSDT.P`. BTC perps only — spot pairs have a different liquidation structure and won't reflect the thesis, and the parameter defaults are BTC-calibrated (§9). Pick one venue and stay on it; funding and liquidation profiles differ enough between exchanges to move results.

---

## 8. Running it — layer order

Do not turn everything on and admire the result. Each filter cuts trades, and a smaller sample looks better by accident.

**Run 1 — mechanical baseline.** `divMode = Off`, `useVolSpk = off`, `usePdLevel = off`, `useDxy = off`. This tells you whether sweep-and-reclaim in this window has an edge net of fees. Nothing else matters until you know this.

**Run 2 — add divergence only.** Record Δ trade count and Δ net R.

**Run 3 — add volume spike only** (divergence back off). Same measurement.

**Run 4 — add prior-day levels only.**

**Run 5 — DXY only.**

Then combine only what actually paid. A filter that improves avg R while cutting trade count by more than half is usually selecting a lucky subsample, not filtering noise.

### Variants worth testing separately

- **Power window +1h and +2h** — captures the real Tokyo open rather than the NY vacuum. Genuinely different setups; test them as separate configurations, not as a tuned parameter.
- **Longs vs shorts unlocked separately.** BTC downside is faster (liquidation cascades). Expect asymmetry even where the logic is symmetric.
- **Box window length** — 2h is your draft's choice, not a law. 90min and 3h are both defensible.

---

## 9. Scope — BTC only

This is a **BTC-only strategy**. Everything below is calibrated to BTC perps and none of it should be assumed to transfer:

- **Fee arithmetic (§4)** assumes ~0.09–0.10% round-trip taker. Different venue, different maths, different verdict on the stop.
- **ATR-relative sweep depths** (`minPenAtr` 0.15 / `maxPenAtr` 1.50) are BTC's distribution. A higher-beta asset sweeps deeper in ATR terms for the same structural reason, so these thresholds would need re-derivation, not just tweaking.
- **Directional asymmetry.** BTC downside is faster than upside — liquidation cascades are not symmetric. Expect long and short buckets to diverge even though the logic is mirrored. Don't average them; read them separately.
- **The 16:00 ET anchor** is a BTC-specific premise. It works because BTC is 24/7 and US institutional flow exits at the equity close, leaving a genuine liquidity vacuum. An asset that doesn't trade around the clock has no such vacuum to fade.

### One exception: a second symbol as a validation tool

Running identical settings on a correlated asset is still worth doing **as a robustness test**, not as a trading decision. A genuine structural edge — post-NY vacuum, thin book, failed break — usually survives the transfer at least partially. Total collapse on a second symbol means you fitted BTC's specific window rather than the structure. That's a diagnostic you run once and act on; it isn't a second market to trade.

### CME gap alignment

Not implemented. It's Monday-only (~1/5 of your sample), and it needs `CME:BTC1!` data that's absent on many chart configurations. Deferred to v2 — flag it if you want it and I'll add a gap-proximity gate.

---

## 10. Reading the stats table

Buckets are tailored to this strategy rather than the generic session/regime split from the Camarilla and EMA scripts — this is a single-session strategy, so session bucketing tells you nothing.

**DIRECTION** — long vs short. Watch for asymmetry; if one side carries everything, consider trading only that side rather than averaging a winner with a loser.

**SWEEP DEPTH** — shallow (<0.4 ATR) / mid (0.4–1.0) / deep (>1.0). This is your heatmap proxy in action. If deep sweeps materially outperform, the "wait for the liquidation cluster to be hit" instinct is validated by something measurable. If they don't, that's worth knowing too.

**BOX WIDTH** — narrow (<1.5 ATR) / normal / wide (>3 ATR). A wide box means the "consolidation" wasn't consolidating. If wide-box trades are the losers, set `maxBoxAtr` and stop taking them.

**Footer** — total n, win%, net R, best single trade and what % of net it represents, and the risk-skip count.

> If one trade is 40%+ of net R, you don't have a strategy, you have a lucky sample.

### R accounting

R uses the **original** stop distance and the **original** position size, snapshotted at entry. Breakeven shifts and TP1 scale-outs can't distort the denominator. This matters: the naive implementation recalculates risk after the BE move, which makes every runner look infinitely profitable in R terms.

---

## 11. Evaluation checklist

Work in order. Stop at the first failure.

- [ ] **Sample size.** Under ~30 trades per bucket is noise. One session per day means ~250 trades a year at most — a 6-month test is a small sample by construction.
- [ ] **Average trade net of commission** above roughly 2× round-trip cost.
- [ ] **Profit factor** above ~1.2 in backtest (below that usually means below 1.0 live).
- [ ] **Max consecutive losses** — can you sit through that streak at 4pm ET without changing the rules?
- [ ] **Equity curve shape** — grinding, or three outliers and a flat line? Check the best-trade footer.
- [ ] **Max drawdown vs your tolerance.** Yours, not the backtest's.

## 12. Walk-forward

1. Set the chart date range to the **first 60%** of your data.
2. Tune `minPenAtr`, `maxPenAtr`, `armExpiry`, `stopAtr`.
3. **Freeze. Change nothing.**
4. Move the range to the **last 40%**.
5. Compare.

Out-of-sample collapse is the normal outcome, not a failure of process. It's the process working.

Also test: different epochs (2021 bull / 2022 bear / 2023 chop), a second symbol with identical settings **as a one-off robustness check only** (§9), and deliberately worse fee assumptions to find where the edge breaks.

Because this is single-session and BTC-only, your sample ceiling is roughly 250 trades a year. Epoch testing matters more here than it would for a multi-session strategy — you cannot compensate for a thin sample by adding markets, so the only axis you have left is time.

---

## 13. Known limitations

- **No liquidation heatmap.** §6. The largest gap.
- **No economic calendar.** `newsDates` is a manual blackout list. You'll need to paste CPI/FOMC/NFP dates (in ET) yourself.
- **No funding rate modelling.** Positions held across a funding interval carry an unmodelled cost. Your holds are short, so this is minor, but it's non-zero.
- **Intrabar ambiguity.** If a 5M bar touches both stop and TP1, Pine guesses the order. On sweep setups — where the stop sits just beyond a wick that already printed — this is a real optimism bias. Test on 1M to bound it.
- **Limit fills assumed.** TP1/TP2 assume you got filled at the level. In a fast reclaim you may not have.
- **DXY is not 24/7.** Handled with `ignore_invalid_symbol` and `na` guards, but during forex-closed hours the filter simply passes rather than blocking. That's deliberate — silently blocking every weekend trade would be worse.

---

## 14. Troubleshooting

| Symptom | Cause |
|---|---|
| No trades at all | Day mask excludes your box days; or `boxValid` failing on `minBoxAtr`/`maxBoxAtr`; or chart history shorter than the session windows. |
| Trades outside the window | Session string typo. Format is `HHMM-HHMM`, no colon. |
| "Deep" bucket empty | `maxPenAtr` disarming before those sweeps reclaim. Raise it and watch the risk-skip count. |
| Every trade skipped on risk | `maxRiskPct` too tight for the current volatility regime, or `maxPenAtr` letting through wicks that produce huge stops. |
| Box drawn an hour off | You changed the session to a local NZ time. Don't. Anchor to ET. |
| MFI/OBV flat | Symbol provides no volume. Switch oscillator. |
| Compile error on `force_overlay` | Only `plot`, `plotshape`, `label.new` and `box.new` accept it. `barcolor` and `bgcolor` don't. |

---

*Not financial advice. This is a jar of brine with strong opinions about pivot geometry.*
