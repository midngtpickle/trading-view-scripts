# EMA & PVSR Master

[![Pine Script v6](https://img.shields.io/badge/Pine_Script-v6-blue.svg)](https://www.tradingview.com/pine-script-docs/)
[![Source Code](https://img.shields.io/badge/Source_Code-ema__pvsr__master.pine-purple.svg)](ema_pvsr_master.pine)

Combines two previously separate indicators—**EMA & PVSR Indicators** and **PVSR Volume Panel**—into a single, cohesive script. Moving averages and candle coloring stay on the main price chart, while volume displays in its dedicated sub-pane below. Both subsystems read directly from the exact same PVSR (Price Volume Spread Analysis) calculation engine, guaranteeing that candle colors and volume bars never drift out of sync.

---

## 📈 Moving Averages & Trend Tools

- **6 Exponential Moving Averages (EMAs)**: Default lengths `10, 21, 50, 100, 200, 800`, each with its own toggle, length, and color. Line width is fixed at 1.
- **3 Simple Moving Averages (SMAs)**: Default lengths `50, 100, 200`, each with its own toggle, length, and color. Line width is fixed at 2.
- **VWAP**: TradingView's session-anchored `ta.vwap(close)`, with adjustable color and width. There is no custom anchor setting.
- **Crossover Detection & Visual Labels**:
  - **SMA Golden / Death Cross**: Default `50 / 200` period detection.
  - **EMA Golden / Death Cross**: Default `9 / 21` period detection.
  - Independent toggles and customizable fast/slow lengths for both pairs.

---

## 📊 PVSR Volume Classification

Every bar is evaluated once based on volume relative to its recent moving average and the volume-to-range spread over a lookback window (default `10` bars):

| Classification | Candle / Bar Color | Trigger Condition |
| :--- | :--- | :--- |
| **Climax** | 🟢 Green / 🔴 Red | Volume $\ge$ Climax Multiplier (default `2.0×`) × average, **OR** highest Volume-Spread bar in lookback window |
| **Above-Average** | 🔵 Blue / 🟣 Violet | Volume $\ge$ Above-Average Multiplier (default `1.5×`) × average |
| **Regular** | ⚪ Light Gray / 🔘 Dark Gray | Standard market volume / missing volume data |

This single classification drives:
1. **Candle Body & Wick Colors** on the price chart.
2. **Volume Column Colors** in the sub-pane.
3. **Dynamic Vector Supply / Demand Zones** (optional).

### Key Calculation Controls

- **Exclude current bar from lookback** *(Default: ON)*: Measures average volume and volume-spread highs over the previous $N$ bars (excluding the bar being tested). This matches the canonical PVSRA specification. When disabled, a massive volume bar sits inside its own comparison window and elevates the threshold.
- **Use volume × spread climax leg** *(Default: ON)*: Controls the spread-multiplied climax test. Fires when a bar marks a new lookback high in `Volume × (High - Low)`. Disabling this lets you inspect classifications driven strictly by pure volume multiples.

---

## 📉 Dedicated Volume Sub-Pane

Below the main price chart, the indicator's pane provides:
- **PVSR-Colored Volume Columns**: Direct visual correspondence to price action.
- **Volume Moving Average Line**: Simple MA of volume (default `20` bars, independent of the PVSR lookback).
- **Threshold Lines**: Plots the Above-Average and Climax thresholds (the PVSR average × each multiplier), so you can see exactly where a bar changes class.

---

## 📦 Dynamic Vector Zones (Supply & Demand)

Off by default (both the master toggle and the section toggle start disabled). When enabled, Climax and Above-Average vector candles project supply and demand order-flow zones:
- **Bullish Vector Candles**: Project **Demand Zones** below price.
- **Bearish Vector Candles**: Project **Supply Zones** above price.
- **Zone Extent**: `Body with wicks` (full high/low, default) or `Body only`.
- **Mitigation Handling**: Zones extend rightward until mitigated by price: `Close` (a bar closes through the far edge, default) or `Wick` (any wick pierces it).
- **Auto-Pruning**: Active zones are capped per side (default `20`, max `50`); the oldest is deleted first.
- **Styling**: Zone transparency, border width, and an optional single override color.

---

## ⚙️ Settings & Organization

Master toggles allow enabling or disabling entire subsystems with one click without resetting underlying configurations:
- **Master Toggles**: EMAs, EMA Cross Labels, SMAs, SMA Cross Labels, VWAP, PVSR Candles, Volume Panel, Vector Zones.
- **Customizable Thresholds**: Fine-tune volume lookback length, above-average multiplier, and climax multiplier.
- **Appearance**: Per-element colors, VWAP width, zone transparency and border width.

---

## 🔔 Alerts

Configured for standard TradingView alert integration:
- **EMA Golden Cross / Death Cross**
- **SMA Golden Cross / Death Cross**
- **Price Crossing VWAP**
- **Bullish / Bearish Vector Candle** (Triggered on Climax or Above-Average volume; gated by the **Show Vector Candle Alerts** setting)

> [!TIP]
> Each part of this script is also available as a standalone indicator in [`individual_scripts/`](../individual_scripts/) (`ema_sma_`, `pvsr_candles_`, `pvsr_volume_`, `pvsr_vector_zones_`). The standalone PVSR scripts each carry their own copy of the classification inputs, so they only agree if you set them identically. This master script has no such drift risk.

---

## 💡 Notes & Troubleshooting

> [!NOTE]
> **Pane Setup & `force_overlay`**:
> The indicator is declared with `overlay=false` so the volume pane can render beneath price, but price plots (EMAs, SMAs, VWAP, candle coloring) use `force_overlay=true` to project onto the main price chart.

> [!TIP]
> **If Volume Bars Appear Compressed on the Main Chart**:
> If the indicator is added directly to the main chart, volume numbers (e.g. thousands/millions) and price numbers will share one scale, compressing the volume bars. Right-click the indicator name in your chart legend $\rightarrow$ **Move to** $\rightarrow$ **New pane below**.

---

## 📝 Changelog

- **v3 (2 August 2026)**
  - Added **Exclude current bar from lookback** (default: on). Average volume and volume-spread highs are measured over prior $N$ bars matching authentic PVSRA rules.
  - Added **Use volume × spread climax leg** toggle (default: on).
  - Fixed compile warning: removed invalid `force_overlay` parameter from `barcolor()`.
- **v2**
  - Unified EMA & PVSR Indicators with PVSR Volume Panel into a single script to ensure synchronized state and prevent calculation drift.
