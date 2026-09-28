# TradingView Pine Script Indicators & Strategies (v6)

[![Pine Script v6](https://img.shields.io/badge/Pine_Script-v6-blue.svg)](https://www.tradingview.com/pine-script-docs/)
[![License: MPL 2.0](https://img.shields.io/badge/License-MPL_2.0-brightgreen.svg)](https://opensource.org/licenses/MPL-2.0)
[![TradingView](https://img.shields.io/badge/TradingView-Compatible-0084ff.svg)](https://www.tradingview.com/)

![TradingView Pine Script Suite](lead-image.png)

A curated collection of modular, production-ready Pine Script v6 indicators and strategies for TradingView. Designed for active traders, swing traders, and technical analysts seeking clean charts, synchronized volume-spread classification, automated multi-timeframe key levels, RSI divergence detection, and regime-hedged scalping strategies without cluttering the screen.

---

## 📑 Table of Contents

- [Featured Indicators & Strategies](#-featured-indicators--strategies)
  - [1. EMA & PVSR Master](#1-ema--pvsr-master)
  - [2. Pivots + Key Levels Master](#2-pivots--key-levels-master)
  - [3. RSI Divergence Indicator MP (v2 & v3)](#3-rsi-divergence-indicator-mp-v2--v3)
  - [4. Individual Scripts](#4-individual-scripts)
  - [5. BTC 5M & 15M Scalping Strategy Pair](#5-btc-5m--15m-scalping-strategy-pair)
  - [6. BTC Asia Range Sweep & Reclaim](#6-btc-asia-range-sweep--reclaim)
- [Installation Guide](#-installation-guide)
- [TradingView Limits & Best Practices](#-tradingview-limits--best-practices)
- [Repository Structure](#-repository-structure)
- [License & Disclaimer](#-license--disclaimer)

---

## 🚀 Featured Indicators & Strategies

### 1. [EMA & PVSR Master](Master%20Suite%20Indicators/ema_pvsr_master.pine)
> **Combines multi-period moving averages, VWAP, and synchronized PVSRA volume classification into a single unified script.**

- **Synchronized Volume Analysis**: Computes PVSRA (Price Volume Spread Analysis) once to drive both the price-chart candle coloring and the lower volume pane bars, so they can't drift apart.
- **Volume Classification Modes**:
  - **Climax (Green/Red)**: Volume ≥ 2.0× average, or the highest volume × spread bar in the lookback.
  - **Above-Average (Blue/Violet)**: Volume ≥ 1.5× average.
  - **Regular (Gray)**: Normal trading activity.
- **Dynamic Vector Zones** (off by default): Order-flow supply (bearish vector) and demand (bullish vector) boxes with close or wick mitigation and a per-side zone cap.
- **Moving Average Suite**: 6 configurable EMAs (10, 21, 50, 100, 200, 800), 3 SMAs (50, 100, 200), session VWAP, and independent EMA (9/21) and SMA (50/200) golden / death cross labels.
- **Integrated Alerts**: EMA and SMA crosses, price crossing VWAP, and bullish / bearish vector candles.

📖 **Detailed Documentation**: [ema_pvsr_master.description.md](Master%20Suite%20Indicators/ema_pvsr_master.description.md)

---

### 2. [Pivots + Key Levels Master](Master%20Suite%20Indicators/pivots_and_key_levels_master_.pine)
> **Multi-timeframe Fibonacci and Camarilla pivot points combined with session and higher-timeframe (HTF) key levels and smart label merging.**

- **Advanced Pivot System**:
  - Selectable calculations: **Fibonacci** or **Camarilla** (default). Camarilla uses a shifted mapping (R3 = H4, the breakout level), so it won't line up with TradingView's built-in.
  - Multi-timeframe support: Auto, Hourly, Daily, Weekly, Monthly, Quarterly, Yearly, Biyearly, Triyearly, Quinquennial.
  - Historical lookback control to respect TradingView drawing budgets, and per-level close-cross or touch alerts.
- **Key Levels**:
  - **Asia Session**: High, Low and 50% of a configurable session (default 16:00–18:00 New York, the "Asia Box"), tracked live or as the last completed session.
  - **Monday Range**: Live Monday High, Low, and Open.
  - **Daily**: Today's Open, Previous Day High/Low/50% (PDH/PDL).
  - **Weekly**: Current Week High/Low/50% (live), Previous Week High/Low/50% (PWH/PWL).
  - **Monthly & Yearly**: Live developing and previous period extremes and midpoints, plus the current year open.
- **Smart Level Merging**: Levels within a configurable tick tolerance collapse into a single line with concatenated labels (e.g., `PDH  |  CW High`) to prevent overlapping line clutter.
- **Budget-Optimized**: Redraws on the last bar to stay inside TradingView's 500-line / 500-label object limits.

📖 **Detailed Documentation**: [pivots_and_key_levels_master.description.md](Master%20Suite%20Indicators/pivots_and_key_levels_master.description.md)

---

### 3. RSI Divergence Indicator MP (v2 & v3)
> **A hardened fork of TradingView's built-in Divergence Indicator, detecting regular and hidden RSI divergences with filtering and confirmation controls the built-in lacks.**

- **[v2](Master%20Suite%20Indicators/rsi_divergence_mp_v2.pine)**: Bar-close confirmation, optional true-swing-extreme price comparison, minimum RSI delta filter, RSI zone filter for regular divergences, optional divergence lines on the price chart, and a dynamic webhook-ready `alert()` alongside per-type alert conditions.
  - 📖 **Documentation**: [rsi_divergence_mp_v2 README.md](Master%20Suite%20Indicators/rsi_divergence_mp_v2%20README.md)
- **[v3](Master%20Suite%20Indicators/rsi_divergence_mp_v3.pine)**: Everything in v2, plus **early signals** that print after a shorter right lookback (default 2 bars instead of 5), ✓ held / ✕ failed outcome markers, a hit-rate table, and early / failed alerts.
  - 📖 **Documentation**: [rsi_divergence_mp_v3 README.md](Master%20Suite%20Indicators/rsi_divergence_mp_v3%20README.md)

---

### 4. [Individual Scripts](individual_scripts/)
> **The two master scripts split into standalone indicators, for when you only want one piece on the chart or need the full object budget for it.**

| Script | Split out of | What it does |
|---|---|---|
| [ema_sma_.pine](individual_scripts/ema_sma_.pine) | EMA & PVSR Master | 6 EMAs, 3 SMAs, VWAP, golden/death cross labels |
| [pvsr_candles_.pine](individual_scripts/pvsr_candles_.pine) | EMA & PVSR Master | PVSRA candle coloring + vector candle alerts |
| [pvsr_volume_.pine](individual_scripts/pvsr_volume_.pine) | EMA & PVSR Master | PVSRA-colored volume columns, volume MA, threshold lines (own pane) |
| [pvsr_vector_zones_.pine](individual_scripts/pvsr_vector_zones_.pine) | EMA & PVSR Master | Supply/demand boxes from vector candles |
| [pivots_.pine](individual_scripts/pivots_.pine) | Pivots + Key Levels Master | Fibonacci / Camarilla pivots with alerts |
| [key_levels_.pine](individual_scripts/key_levels_.pine) | Pivots + Key Levels Master | Asia session, Monday range, D/W/M/Y levels with smart merging |

> [!IMPORTANT]
> The three standalone PVSR scripts each carry their own copy of the classification inputs. They only agree with each other if those inputs are set identically in all three. The master script doesn't have this problem.

The master scripts' documentation applies to their split-out versions.

---

### 5. [BTC 5M & 15M Scalping Strategy Pair](scalping_strategies/)
> **A regime-hedged pair of Pine Script v6 strategies engineered for 5m and 15m Bitcoin trading, complete with built-in R-multiple performance analytics tables.**

- **[Camarilla Range Bounce Scalper](scalping_strategies/camarilla_range_bounce.pine)**:
  - **Mean-Reversion Thesis**: Fades touches of Camarilla S3 (long) and R3 (short) during low-volatility/ranging market conditions, targeting the central pivot (PP) and opposite boundaries with stops beyond S4/R4.
  - **Textbook Camarilla**: Unlike the Pivots + Key Levels indicator, this strategy uses standard levels (R3 = H3). The indicator's R3 is this strategy's R4.
  - **Regime Filters**: ADX max threshold (< 22), price containment within S3/R3 over lookback window, optional BB width percentile filter, and RSI extreme confirmation.
  - **Scale-Out Mechanics**: Takes 50% TP1 at central pivot (PP), moves stop to breakeven, and lets TP2 run to opposite R3/S3 level.
  - 📖 **Detailed Strategy Guide**: [camarilla_range_bounce.description.md](scalping_strategies/camarilla_range_bounce.description.md)

- **[EMA 9/21 Pullback Continuation](scalping_strategies/ema_pullback_continuation.pine)**:
  - **Trend-Following Thesis**: Arms on a pullback to the 9 EMA inside an established 9/21 trend, then enters on the resumption candle.
  - **Regime Filters**: High ADX requirement (> 22), 9/21 EMA separation spacing, and higher-timeframe EMA direction alignment (1H for 5m chart / 4H for 15m chart).
  - **Runner Exits**: Chandelier trailing stop (default), slow EMA trail, or fixed R-multiple target to preserve multi-R trend outliers.
  - 📖 **Detailed Strategy Guide**: [ema_pullback_continuation.description.md](scalping_strategies/ema_pullback_continuation.description.md)

- **Honest R-Multiple Analytics**: Both strategies show on-chart tables tracking net R, win rate, and trade counts per position (scale-outs aren't double-counted), sliced by ADX regime, UTC session (Asia / London / NY / Late), and direction.

---

### 6. [BTC Asia Range Sweep & Reclaim](scalping_strategies/btc_asia_sweep/)
> **Liquidity sweep and mean-reversion strategy specifically designed for Bitcoin perpetual contracts during the post-US close / early Asian session.**

- **[Asia Range Sweep & Reclaim v1](scalping_strategies/btc_asia_sweep/midnight_pickle_asia_sweep_v1.pine)**:
  - **Core Thesis**: Exploits liquidity sweeps that occur in the thin-orderbook vacuum following the US cash close. Identifies the "Asia Box" (16:00–18:00 ET), detects false breakouts / stop sweeps during the "Power Window" (18:00–20:00 ET), and enters when price reclaims back inside the range.
  - **Timezone-Safe Anchoring**: Built with America/New_York (US DST) and Asia/Tokyo anchoring to prevent DST seasonal drift.
  - **Precision Filters**: Minimum/maximum ATR sweep penetration thresholds (0.15–1.50 ATR), zero-lag box-anchored divergence engine (six oscillators), Tokyo lunch filter (11:30–12:30 JST), optional volume / prior-day level / DXY filters, and risk-bounded position sizing.
  - 📖 **Detailed Strategy Guide**: [asia_sweep_strategy_guide.md](scalping_strategies/btc_asia_sweep/asia_sweep_strategy_guide.md)
  - 📊 **Interactive Visual Playbook**: [btc_asia_sweep_playbook.html](scalping_strategies/btc_asia_sweep/btc_asia_sweep_playbook.html)
  - 📝 **Manual NZ-schedule Playbook**: [Asia Range Sweep v3.md](scalping_strategies/btc_asia_sweep/Asia%20Range%20Sweep%20v3.md). A discretionary, UTC-anchored checklist for trading the setup by hand from New Zealand. Its times are separate from the script's defaults.

> [!TIP]
> The **Asia Session** key level in the Pivots + Key Levels indicator defaults to the same 16:00–18:00 ET window, so you can see the Asia Box on any chart without running the strategy.

---

## 🛠 Installation Guide

Follow these steps to add any indicator or strategy from this repository to your TradingView chart:

1. Open **[TradingView](https://www.tradingview.com/)** and navigate to your chart.
2. At the bottom of the screen, open the **Pine Editor** tab.
3. Click **Open** -> **New Indicator** (or **New Strategy** for strategy scripts).
4. Copy the entire raw code from the desired `.pine` file:
   - [ema_pvsr_master.pine](Master%20Suite%20Indicators/ema_pvsr_master.pine)
   - [pivots_and_key_levels_master_.pine](Master%20Suite%20Indicators/pivots_and_key_levels_master_.pine)
   - [rsi_divergence_mp_v2.pine](Master%20Suite%20Indicators/rsi_divergence_mp_v2.pine) or [rsi_divergence_mp_v3.pine](Master%20Suite%20Indicators/rsi_divergence_mp_v3.pine)
   - Any script in [individual_scripts/](individual_scripts/)
   - [camarilla_range_bounce.pine](scalping_strategies/camarilla_range_bounce.pine) *(strategy)*
   - [ema_pullback_continuation.pine](scalping_strategies/ema_pullback_continuation.pine) *(strategy)*
   - [midnight_pickle_asia_sweep_v1.pine](scalping_strategies/btc_asia_sweep/midnight_pickle_asia_sweep_v1.pine) *(strategy)*
5. Paste the code into the Pine Editor.
6. Click **Save** and give the script a name.
7. Click **Add to Chart**.

> [!TIP]
> **Volume Pane Placement (EMA & PVSR Master)**:
> The EMA & PVSR script is configured with `overlay=false` so the volume histogram renders in its own pane below the chart while moving averages and candle coloring are forced onto the main price scale via `force_overlay=true`. If the volume pane lands directly on the price chart upon adding, right-click the indicator name in the legend and select **Move to -> New pane below**.

---

## ⚡ TradingView Limits & Best Practices

- **Object Budget (500 Lines / 500 Labels / 500 Boxes)**:
  TradingView caps each script at 500 lines, 500 labels, and 500 boxes. The scripts here stay well within these limits at default settings. In Pivots + Key Levels, the pivots and key levels share one budget; with a large pivot lookback, the oldest drawings roll off. Use the split-out scripts if you need the full budget for each.
- **Bar Replay Behavior**:
  The *current* week, month, and year levels in Pivots + Key Levels (and `key_levels_.pine`) are only correct on a live realtime bar. In Bar Replay they show the **previous** period's extremes. Asia Session, Monday Range, and all "previous period" levels are unaffected.
- **Strategy Settings**:
  Commission (0.045%) and slippage (2 ticks) are set in each strategy's code. Check they survived in **Properties** before judging a backtest, and leave slippage non-zero.
- **Execution Engine**:
  All scripts are written in **Pine Script v6**.

---

## 📂 Repository Structure

```text
├── README.md                                  # Repository overview and quick start guide
├── AGENTS.md                                  # Instructions for AI coding agents (source of truth)
├── CLAUDE.md                                  # Pointer to AGENTS.md for Claude Code
├── LICENSE                                    # Mozilla Public License 2.0
├── .gitignore                                 # Standard Git ignore configuration
├── lead-image.png                             # Main repository header preview image
├── Master Suite Indicators/                   # Core indicator suite
│   ├── ema_pvsr_master.pine                   # EMA & PVSR Master indicator
│   ├── ema_pvsr_master.description.md         # EMA & PVSR Master documentation
│   ├── pivots_and_key_levels_master_.pine     # Pivots + Key Levels Master indicator
│   ├── pivots_and_key_levels_master.description.md # Pivots + Key Levels documentation
│   ├── rsi_divergence_mp_v2.pine              # RSI Divergence MP v2 indicator
│   ├── rsi_divergence_mp_v2 README.md         # RSI Divergence MP v2 documentation
│   ├── rsi_divergence_mp_v3.pine              # RSI Divergence MP v3 (early signals)
│   └── rsi_divergence_mp_v3 README.md         # RSI Divergence MP v3 documentation
├── individual_scripts/                        # Master scripts split into standalone indicators
│   ├── ema_sma_.pine                          # EMAs, SMAs, VWAP, cross labels
│   ├── pvsr_candles_.pine                     # PVSRA candle coloring
│   ├── pvsr_volume_.pine                      # PVSRA volume pane
│   ├── pvsr_vector_zones_.pine                # PVSRA supply/demand zones
│   ├── pivots_.pine                           # Fibonacci / Camarilla pivots
│   └── key_levels_.pine                       # Asia, Monday, D/W/M/Y key levels
└── scalping_strategies/                        # BTC scalping strategies
    ├── camarilla_range_bounce.pine            # Camarilla Range Bounce mean-reversion strategy
    ├── camarilla_range_bounce.description.md  # Camarilla Range Bounce strategy guide
    ├── ema_pullback_continuation.pine         # EMA 9/21 Pullback Continuation trend strategy
    ├── ema_pullback_continuation.description.md # EMA 9/21 Pullback Continuation strategy guide
    └── btc_asia_sweep/                        # BTC Asia session range sweep & reclaim
        ├── midnight_pickle_asia_sweep_v1.pine # Asia Range Sweep & Reclaim v1 strategy
        ├── asia_sweep_strategy_guide.md       # Strategy and timezone guide for the script
        ├── btc_asia_sweep_playbook.html       # Interactive visual strategy playbook
        └── Asia Range Sweep v3.md             # Manual NZ-schedule trading playbook
```

---

## 📜 License & Disclaimer

### License
This project is licensed under the [Mozilla Public License 2.0 (MPL-2.0)](LICENSE). You are free to use, modify, and distribute this software in accordance with the terms of the MPL-2.0 license.

### Disclaimer
*These scripts are provided for educational and informational purposes only. Nothing contained herein constitutes investment, financial, or trading advice. Trading financial markets involves substantial risk of loss. Always conduct your own research and risk management before executing trades.*
