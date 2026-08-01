# EMA & PVSR Master

Combines two previously separate indicators, EMA & PVSR Indicators and PVSR Volume Panel, into one script. Moving averages and candle coloring stay on the price chart while volume gets its own pane below, and both read from the same PVSR (volume classification) calculation so the candle colors and the volume bars never fall out of sync.

## Moving averages

- 6 EMAs (default lengths 10, 21, 50, 100, 200, 800), each with its own length, color, and on/off toggle
- 3 SMAs (default lengths 50, 100, 200), same per-line controls
- VWAP with adjustable color and line width
- Golden Cross / Death Cross labels for both the SMA pair (default 50/200) and the EMA pair (default 9/21), plotted independently with their own fast/slow lengths

## PVSR volume classification

Every bar is classified once, using volume relative to its recent average and the volume-to-range spread over a lookback period:

- Climax (green/red): volume at or above the climax multiplier, or the highest volume-spread bar in the lookback window
- Above-average (blue/violet): volume at or above the above-average multiplier
- Regular (light/dark gray): everything else, including bars where volume data isn't available

That classification drives the candle colors on the chart, the volume column colors in the indicator's pane, and an optional set of supply/demand zones (see below).

## Volume pane

Below the price chart, the indicator's own pane shows volume columns colored by PVSR classification, a volume moving average, and threshold lines marking the above-average and climax levels, so you can see at a glance how a bar's volume compares to its recent history.

## Vector zones (optional)

Climax and above-average candles can draw a box around their range: bullish vectors create demand zones below price, bearish vectors create supply zones above. Zones extend right until price closes through them (or a wick pierces them, depending on your mitigation setting) and are capped per side so old zones roll off automatically.

## Settings

Master toggles at the top turn each subsystem on or off (EMAs, EMA cross labels, SMAs, SMA cross labels, VWAP, PVSR candles, volume panel, vector zones) without touching the individual settings underneath. Each moving average, the PVSR thresholds, the vector zone appearance, and the volume panel colors all have their own input groups.

## Alerts

EMA Golden/Death Cross, SMA Golden/Death Cross, price crossing VWAP, and bullish or bearish vector candle (climax or above-average volume).

## Notes

The indicator is declared with `overlay=false` so the volume pane can exist, but every price-chart plot is tagged to force itself onto the main chart. If you don't see EMAs, SMAs, or candle coloring, check that the relevant master toggle is on.
