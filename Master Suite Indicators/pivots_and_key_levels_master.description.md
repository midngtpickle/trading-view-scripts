# Pivots + Key Levels Master

[![Pine Script v6](https://img.shields.io/badge/Pine_Script-v6-blue.svg)](https://www.tradingview.com/pine-script-docs/)
[![Source Code](https://img.shields.io/badge/Source_Code-pivots__and__key__levels__master__.pine-purple.svg)](pivots_and_key_levels_master_.pine)

Unifies a multi-timeframe pivot point calculation engine (**Fibonacci** and **Camarilla**) with an automated key level framework (**Asia Session**, **Monday Range**, **Daily**, **Weekly**, **Monthly**, **Yearly**). Designed to provide comprehensive market structure context on a single chart while actively managing TradingView's 500-line / 500-label drawing object limits.

---

## 🎯 Pivot Points System

- **Calculation Methods** (default **Camarilla**):
  - **Fibonacci Pivots** (0.382 / 0.618 / 1.0 / 1.272 / 1.618 × prior range)
  - **Camarilla Pivots** (Custom institutional variant, see below)
- **Timeframe Resolution** (default **Daily**): Auto, Hourly, Daily, Weekly, Monthly, Quarterly, Yearly, Biyearly, Triyearly, Quinquennial. `Auto` uses Daily on charts ≤15m, Weekly on higher intraday charts, Monthly on daily charts, and Yearly on weekly/monthly charts.
- **Use Daily-based Values** (default on): Build the pivot period from requested OHLC data rather than from the chart's own bars.
- **Historical Lookback** (default `1`): How many past pivot periods stay drawn. Raise it with care; it shares the object budget with the key levels.
- **Visuals & Labels**: Toggle level pairs (P, R1/S1 … R5/S5), set a color per pair, and place labels at the left or right edge of each segment. Pivot line style and width are not configurable.
- **Integrated Level Alerts**: One alert condition per level (P, R1–R5, S1–S5). **Use Close Price for Alerts** on (default) fires when the close crosses the level; off fires when the bar's range touches it.

### Camarilla Level Shift & Architecture

The Camarilla calculation in this script uses a deliberate custom institutional mapping where levels are shifted by one step compared to textbook equations (H1 and L1 are omitted):

| Plotted Line | Calculation Source | Interpretation |
| :--- | :--- | :--- |
| **R1 / S1** | $H_2 / L_2$ | Inner range support / resistance |
| **R2 / S2** | $H_3 / L_3$ | Mean reversion reversal boundaries |
| **R3 / S3** | $H_4 / L_4$ | **Breakout / Trend acceleration boundary** *(Note: not an extreme)* |
| **R4 / S4** | $H_5 / L_5$ | Strong momentum extension |
| **R5 / S5** | $H_6 / L_6$ | Macro extreme target $(High / Low) \times Close$ |

> [!NOTE]
> **Camarilla Notation Context**:
> Because $H_4$ maps to **R3**, R3 functions as the primary range-breakout trigger rather than a rare outlier extreme. This intentionally differs from TradingView's standard built-in Camarilla indicator.

> [!WARNING]
> **Not the same R3 as the Camarilla Range Bounce strategy.** [`camarilla_range_bounce.pine`](../scalping_strategies/camarilla_range_bounce.pine) uses the textbook mapping (R3 = $H_3$, R4 = $H_4$). That strategy's **R4** is this indicator's **R3**. Don't read one script's levels off the other's chart.

---

## 📏 Key Levels

Automatically draws and tracks market reference levels across six groups:

- **Asia Session**: High, Low, and 50% of a configurable session window. Default is `1600-1800` in `America/New_York`, the post-US-close "Asia Box" used by the [BTC Asia Sweep strategy](../scalping_strategies/btc_asia_sweep/). With **Track Session Live** on (default), levels follow the session while it is open; otherwise only the last completed session is shown. Timezone choices: New York, UTC, Tokyo, Singapore, Hong Kong, London, Auckland. Needs an intraday chart of 1H or lower; on 4H+ nothing is drawn.
- **Monday Range**: Live Monday High, Low, and Open (`Mon H` / `Mon L` / `Mon O`), reset each new week.
- **Daily**: Today's Open (`D Open`), Previous Day High (`PDH`), Previous Day Low (`PDL`), Previous Day 50% (`PD 50%`).
- **Weekly**: Current Week High/Low/50% (live developing), Previous Week High (`PWH`), Low (`PWL`), and 50%.
- **Monthly**: Current Month High/Low/50% (live developing), Previous Month High (`PMH`), Low (`PML`), and 50%.
- **Yearly**: Current Year Open/High/Low/50% (live developing), Previous Year High (`PYH`), Low (`PYL`), and 50%.

Key levels are drawn as short lines on the latest bar (default `10` bars long) with the price in the label.

---

## 🔀 Smart Level Merging (De-Cluttering)

When multiple key levels sit within the **Merge Tolerance** (default `2` ticks), the indicator **merges them onto a single line**:

- Instead of 3 overlapping lines obscuring the chart, a single line is rendered.
- Labels are concatenated (e.g., `Mon H  |  PDH  |  CW High`).
- Levels are added shortest timeframe first (Asia → Monday → Daily → Weekly → Monthly → Yearly), and the first level at a price keeps its color, style, and width.

---

## ⚙️ Customization & Global Overrides

- **Global Style Overrides**: Force uniform color, line style (Solid, Dashed, Dotted), or line width across all key levels simultaneously with master override toggles.
- **Per-Group Customization**: Individually configure visibility, color, style, and width for each group (Asia, Monday, Daily, Current/Previous Week, Month, and Year) when master overrides are disabled.

---

## 💡 Notes & Technical Context

> [!IMPORTANT]
> **TradingView 500-Object Cap Management**:
> Pivot segments and key level lines redraw on the current bar rather than writing historical drawing objects bar-by-bar. This maintains high responsiveness and avoids hitting TradingView's platform ceiling of 500 lines and 500 labels.

> [!WARNING]
> **Bar Replay Behavior**:
> Current developing periods (Current Week, Month, Year) use `request.security` with `lookahead_off` and no offset. On a live chart the last bar is realtime, so they track the developing period correctly. In Bar Replay the last bar is historical, and on historical bars that call returns the **last closed** period, so the "current" week/month/year lines show the **previous** period's extremes. The Asia Session and Monday Range are built from chart bars and are correct in replay.
>
> For the same reason, don't reuse the current-period values in an alert condition or port them into a strategy.

> [!TIP]
> The two halves are also available separately as [`individual_scripts/pivots_.pine`](../individual_scripts/pivots_.pine) and [`individual_scripts/key_levels_.pine`](../individual_scripts/key_levels_.pine), each with the full 500-line / 500-label budget to itself.

---

## 📝 Changelog

- **v2 (2 August 2026)**
  - Fully documented the $R_1 \dots R_5$ Camarilla level mapping shift in code comments and UI tooltips on R3, R4, and R5 toggles.
  - Documented `request.security` behavior and Bar Replay characteristics for current developing HTF levels.
- **v1**
  - Combined Pivot Enhanced and Key Levels into a unified script sharing the 500-line and 500-label object budget.
