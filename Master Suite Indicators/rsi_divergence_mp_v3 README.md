# RSI Divergence Indicator — MP v3

Pine Script v6 indicator that detects regular and hidden RSI divergences against price. v3 adds **early signals**: a faster, provisional divergence that prints a few bars before the standard one, with each early signal tracked until it either holds or fails.

Everything from v2 is unchanged and still prints exactly as before. This document covers what v3 adds. For the base logic, inputs, filters and known limitations, see [rsi_divergence_mp_v2 README.md](rsi_divergence_mp_v2%20README.md). v3-specific changes are tagged `// [MP v3]` in the source.

**File:** `rsi_divergence_mp_v3.pine`

---

## Install

1. TradingView → Pine Editor → new indicator
2. Paste the contents of `rsi_divergence_mp_v3.pine`
3. Save, then "Add to chart"

It can sit alongside v2 on the same chart. Titles differ (`RSI Div MP v3`), so alerts don't collide.

---

## Why early signals

A standard pivot needs `Pivot Lookback Right` bars after it before it can be confirmed. At the default of 5, a 15m signal prints 75 minutes after the pivot it marks.

Early mode finds the **newest** pivot with a much shorter right lookback (default 2) and compares it against the last **fully confirmed** pivot on the same side. The reference pivot keeps full quality; only the newest one is provisional. The same filters apply: min/max range, min RSI difference, the zone filter (regular only) and the true-swing-extreme option.

## Outcome tracking

Every early signal is watched until the full lookback has passed:

| Marker | Meaning |
|---|---|
| ✓ (held) | The full-lookback detector confirmed a pivot on the same bar. The early pivot became a normal pivot. |
| ✕ (failed) | RSI went beyond the pivot value first, or the full lookback expired without confirmation. |

A hit-rate table (top right) counts held vs failed for bull and bear sides separately, headed with the early lookback in use (e.g. `Early (R=2)`).

"Held" means the **pivot** survived. It doesn't mean the divergence was confirmed by the standard detector or that a trade would have worked. Use the table to judge whether the bars you gain are worth the failures.

---

## Inputs added in v3

### Early signals (v3)
| Input | Default | Notes |
|---|---|---|
| Show early divergence signals | on | Master switch for everything below |
| Early pivot lookback right | 2 | Capped at `Pivot Lookback Right − 1`. 1 = fastest and noisiest. Early mode is off entirely when `Pivot Lookback Right` is 1 |
| Mark outcome (✓ held / ✕ failed) | on | |
| Show early-signal hit-rate table | on | |

Early signals follow the same **Plot Bullish / Hidden Bullish / Bearish / Hidden Bearish** toggles and the **Confirm on bar close** gate as the standard ones.

### Display
| Signal | Label | Color |
|---|---|---|
| Early Regular Bullish | `E Bull` | cyan |
| Early Hidden Bullish | `E H Bull` | faded cyan |
| Early Regular Bearish | `E Bear` | orange |
| Early Hidden Bearish | `E H Bear` | faded orange |

With **Draw divergence lines on price chart** on, early signals get thin dotted lines so they read as provisional.

---

## Alerts

v2's alerts are all still there. v3 adds:

**Per-type `alertcondition()`**
- `EARLY Regular Bullish Divergence`, `EARLY Hidden Bullish Divergence`
- `EARLY Regular Bearish Divergence`, `EARLY Hidden Bearish Divergence`
- `Early Bullish Divergence FAILED`, `Early Bearish Divergence FAILED`

**Any alert() function call** — the dynamic webhook alert also carries early and failed messages:

```
EARLY Regular Bullish Divergence (unconfirmed) | BTCUSDT.P 15 | RSI 31.42 @ 62150.5
Early bullish divergence FAILED | BTCUSDT.P 15 | RSI 28.10 @ 61890.0
```

If you act on early alerts, also subscribe to the FAILED ones.

---

## Known limitations

Everything in the v2 list still applies, plus:

- **One pending early signal per side.** A new early bull signal replaces a pending one before it resolves, and regular and hidden early signals share the same slot.
- **Outcome counts only cover plotted signals.** If a type's Plot toggle is off, its early signals aren't tracked, though their alerts still fire.
- **The table counts from the start of loaded history**, so the numbers change with chart timeframe and how much history is loaded.

---

## Changelog

**v3**
- Added early divergence signals with a shorter right lookback, compared against the last fully confirmed pivot
- Added held / failed outcome tracking with ✓ / ✕ markers
- Added early-signal hit-rate table
- Added early and failed alerts (`alertcondition()` and dynamic `alert()`)
- Added dotted price-chart lines for early signals

**v2** — see [rsi_divergence_mp_v2 README.md](rsi_divergence_mp_v2%20README.md)
