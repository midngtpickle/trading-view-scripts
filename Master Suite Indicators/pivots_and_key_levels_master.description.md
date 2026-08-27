# Pivots + Key Levels Master

[![Pine Script v6](https://img.shields.io/badge/Pine_Script-v6-blue.svg)](https://www.tradingview.com/pine-script-docs/)
[![Source Code](https://img.shields.io/badge/Source_Code-pivots__and__key__levels__master__.pine-purple.svg)](pivots_and_key_levels_master_.pine)

Unifies a multi-timeframe pivot point calculation engine (**Fibonacci** and **Camarilla**) with an automated higher-timeframe (HTF) key level framework (**Monday Range**, **Daily**, **Weekly**, **Monthly**, **Yearly**). Designed to provide comprehensive market structure context on a single chart while actively managing TradingView's 500-line / 500-label drawing object limits.

---

## 🎯 Pivot Points System

- **Calculation Methods**:
  - **Fibonacci Pivots** (Classic ratios)
  - **Camarilla Pivots** (Custom institutional variant)
- **Timeframe Resolution**: Auto, Hourly, Daily, Weekly, Monthly, Quarterly, Yearly, Biyearly, Triyearly, Quinquennial.
- **Intraday Daily OHLC Basis**: Optional toggle to anchor pivot calculations to Daily OHLC data when viewing lower intraday timeframes.
- **Historical Lookback**: Adjustable history depth to keep drawing objects within the platform cap.
- **Visuals & Labels**: Toggle individual levels (P, R1–R5, S1–S5), customize line styles/colors, and position labels on the left or right edge of pivot segments.
- **Integrated Level Alerts**: Trigger alerts when price closes across any pivot level or on intra-bar touch.

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

---

## 📏 Higher-Timeframe (HTF) Key Levels

Automatically draws and tracks critical market reference levels across five key time horizons:

- **Monday Range**: Live Monday High, Low, and Midpoint (resets each new trading week).
- **Daily**: Today's Open, Previous Day High (PDH), Previous Day Low (PDL), Previous Day Midpoint (pMid).
- **Weekly**: Current Week High/Low/Midpoint (live developing), Previous Week High (PWH), Previous Week Low (PWL), Previous Week Midpoint.
- **Monthly**: Current Month High/Low/Midpoint (live developing), Previous Month High (PMH), Previous Month Low (PML), Previous Month Midpoint.
- **Yearly**: Current Year Open/High/Low/Midpoint (live developing), Previous Year High (PYH), Previous Year Low (PYL), Previous Year Midpoint.

---

## 🔀 Smart Level Merging (De-Cluttering)

When multiple higher-timeframe levels cluster near the same price within a user-defined tick tolerance, the indicator **automatically merges them onto a single line**:

- Instead of 3 overlapping lines obscuring the chart, a single line is rendered.
- Labels are concatenated cleanly (e.g., `PDH | CW High | Mon High`).
- Eliminates visual noise around confluence zones.

---

## ⚙️ Customization & Global Overrides

- **Global Style Overrides**: Force uniform color, line style (Solid, Dashed, Dotted), or line width across all key levels simultaneously with master override toggles.
- **Per-Group Customization**: Individually configure colors and visibility for Monday, Daily, Weekly, Monthly, and Yearly sets when master overrides are disabled.

---

## 💡 Notes & Technical Context

> [!IMPORTANT]
> **TradingView 500-Object Cap Management**:
> Pivot segments and key level lines redraw on the current bar rather than writing historical drawing objects bar-by-bar. This maintains high responsiveness and avoids hitting TradingView's platform ceiling of 500 lines and 500 labels.

> [!TIP]
> **Bar Replay Behavior**:
> Current developing periods (Current Week, Month, Year) compute using `request.security` with `lookahead_off` and no offset. On live charts, they track the developing candle in real time. In historical Bar Replay, developing levels reflect the latest historical bar reached by the replay playhead.

---

## 📝 Changelog

- **v2 (2 August 2026)**
  - Fully documented the $R_1 \dots R_5$ Camarilla level mapping shift in code comments and UI tooltips on R3, R4, and R5 toggles.
  - Documented `request.security` behavior and Bar Replay characteristics for current developing HTF levels.
- **v1**
  - Combined Pivot Enhanced and Key Levels into a unified script sharing the 500-line and 500-label object budget.
