# RSI Divergence Indicator — MP

Pine Script v6 indicator that detects regular and hidden RSI divergences against price, with filtering and confirmation controls the TradingView built-in doesn't have.

Hardened fork of TradingView's built-in Divergence Indicator. All changes to the original logic are tagged `// [MP]` in the source.

**File:** `rsi_divergence_mp_v2.pine`

---

## Install

1. TradingView → Pine Editor → new indicator
2. Paste the contents of `rsi_divergence_mp_v2.pine`
3. Save, then "Add to chart"

Runs in its own pane. If **Draw divergence lines on price chart** is enabled it also draws onto the price pane via `force_overlay`.

---

## How it works

RSI pivots are found with `ta.pivotlow` / `ta.pivothigh` using a left and right lookback. When a new pivot forms, it's compared against the previous pivot on the same side — RSI value against RSI value, price against price. The four combinations:

| Type | RSI | Price | Reads as |
|---|---|---|---|
| Regular Bullish | Higher Low | Lower Low | Downtrend losing momentum |
| Hidden Bullish | Lower Low | Higher Low | Pullback in an uptrend |
| Regular Bearish | Lower High | Higher High | Uptrend losing momentum |
| Hidden Bearish | Higher High | Lower High | Pullback in a downtrend |

Labels are plotted back at the pivot bar via `offset=-lbR`.

### Timing — read this before trading it

A pivot needs `lbR` bars to its right before it can be confirmed. **Every signal appears `lbR` bars after the pivot bar it marks.** At the default of 5, a 1m signal is five minutes old when it prints. This is inherent to pivot-based divergence, not a flaw in this script, but it means the entry is never at the pivot.

The pivot check also reads the *live* bar. With **Confirm on bar close** off, a signal can appear and then vanish as the current bar moves. Leave it on unless you have a reason not to.

---

## Inputs

### RSI
| Input | Default | Notes |
|---|---|---|
| RSI Period | 14 | |
| RSI Source | close | |

### Pivots
| Input | Default | Notes |
|---|---|---|
| Pivot Lookback Right | 5 | Also sets signal lag — see above |
| Pivot Lookback Left | 5 | |
| Max of Lookback Range | 60 | Upper bound on bars between the two pivots |
| Min of Lookback Range | 5 | Lower bound. Stops adjacent pivots on the same swing pairing up |
| Use true swing extreme for price | off | See below |

**Use true swing extreme for price** — off, the script compares `low[lbR]` / `high[lbR]`, i.e. price on the bar where RSI pivoted. That bar isn't always the actual swing extreme, so on choppy swings the price comparison can disagree with what the chart looks like. On, it compares the genuine high/low across the `lbL + lbR + 1` bar window centred on the pivot. On is more faithful to the chart; off matches the original built-in.

Note: the min/max range is measured as pivot separation minus one, inherited from the original. Setting `5` filters at a true separation of 6.

### Filters
| Input | Default | Notes |
|---|---|---|
| Confirm on bar close | on | Gates all labels, lines and alerts to `barstate.isconfirmed` |
| Require RSI pivot inside zone | off | Regular divergences only |
| Bull: RSI pivot at or below | 40 | |
| Bear: RSI pivot at or above | 60 | |
| Min RSI difference between pivots | 0 | |

**Zone filter** requires the RSI pivot itself to sit in oversold/overbought territory before a regular divergence counts. It is deliberately *not* applied to hidden divergences — hidden div legitimately forms mid-range during pullbacks, and gating it at 40/60 would suppress nearly all of it.

**Min RSI difference** rejects near-flat divergences where the two pivots are almost the same RSI value. `0` accepts everything (original behaviour). `3–5` is a reasonable starting point on low timeframes.

### Display
| Input | Default |
|---|---|
| Plot Bullish | on |
| Plot Hidden Bullish | off |
| Plot Bearish | on |
| Plot Hidden Bearish | off |
| Draw divergence lines on price chart | off |

Price-chart lines are drawn with `line.new(..., force_overlay=true)`. They accumulate; `max_lines_count=500` means the oldest expire automatically once the cap is reached, so no manual cleanup is needed, but old lines will disappear from historical scrollback on long charts.

---

## Alerts

Two mechanisms, both gated by **Confirm on bar close**:

**Per-type** — four `alertcondition()` entries appear in the alert dialog dropdown (Regular Bullish, Hidden Bullish, Regular Bearish, Hidden Bearish). Static message text; use these when you want one alert per divergence type.

**Any alert() function call** — a single alert covering all four types, with a dynamic message:

```
Regular Bullish Divergence | BTCUSDT.P 5 | RSI 31.42 @ 62150.5
```

Fires `alert.freq_once_per_bar`. Use this one for webhooks.

---

## Known limitations

- **No higher-timeframe selector.** The `timeframe=` argument in `indicator()` cannot coexist with side-effect calls (`alert()`, `line.new()`) — Pine throws CE10080. The built-in's HTF selector was also unsafe here regardless: `offset=-lbR` shifts by chart bars, not HTF bars, so labels land in the wrong place. Change the chart timeframe instead.
- **Signals lag by `lbR` bars.** Not fixable within a pivot-based approach.
- **No cooldown.** Consecutive pivots on the same extended swing can each produce a signal.
- **No trend context.** A regular bearish divergence in a strong uptrend will still print. Filter externally, or gate against an EMA/HTF bias.
- RSI only. Swapping the oscillator means editing the `osc =` line.

---

## Suggested starting points

**BTC scalping, 1–5m** — Pivot lookbacks 5/5, Min RSI difference 4, zone filter on at 35/65, Confirm on bar close on, regular divergences only. Tight enough to cut most of the noise; expect few signals.

**Swing, 1H–4H** — Pivot lookbacks 5/5, Max range 60, Min RSI difference 0–3, zone filter off, hidden divergences on for continuation entries.

Both are starting points, not settings that have been walk-forward tested.

---

## Changelog

**v2** — forked from the TradingView built-in Divergence Indicator (v6)

- Added bar-close confirmation gating
- Added true-swing-extreme price comparison option
- Added minimum RSI delta filter
- Added RSI zone filter for regular divergences
- Added optional price-chart divergence lines
- Added dynamic `alert()` calls alongside the original `alertcondition()`s
- Added `minval` guards on pivot and range inputs
- Switched to `input.source` / `input.bool` / typed inputs, grouped in settings
- Removed the `timeframe=` selector (incompatible with side effects, and misaligned labels)
