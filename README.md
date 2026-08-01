# pine-scripts

Pine Script v6 indicators for TradingView.

## Scripts

### [EMA & PVSR Master](ema_pvsr_master.pine)

EMAs, SMAs, and VWAP on the price chart, plus a PVSR (volume classification) system that colors candles by climax or above-average volume and mirrors that coloring in its own volume pane. Includes optional supply/demand vector zones and crossover alerts.

Full writeup: [ema_pvsr_master.description.md](ema_pvsr_master.description.md)

### [Pivots + Key Levels](pivots_and_key_levels_master_.pine)

Fibonacci or Camarilla pivot points (P, R1-R5, S1-S5) merged with a key-levels system covering the Monday range and daily/weekly/monthly/yearly highs, lows, midpoints, and opens. Nearby levels merge onto a single line instead of stacking on top of each other.

Full writeup: [pivots_and_key_levels_master.description.md](pivots_and_key_levels_master.description.md)

### [PVSRA × Camarilla Regime Strategy](pvsra_camarilla_strategy.pine)

A backtestable `strategy()` built from the two indicators above: Camarilla levels decide where, PVSRA vector candles decide when, and position relative to the break level decides direction. Currently Phase 1, which ships the exit model, cost assumptions, diagnostics, and a random-entry control arm. The real signal lands in Phase 2.

This is a research artifact, not a signal source. Neither PVSRA nor Camarilla has a published evidence base, so the file is built so a negative result is unambiguous, and the random-entry control runs before any parameter tuning. A backtest is not a prediction.

## License

Both scripts are released under the [Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/), per the license header in each file.
