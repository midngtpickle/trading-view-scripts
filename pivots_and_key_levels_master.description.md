# Pivots + Key Levels

Merges two indicators into one: a pivot point system (Fibonacci or Camarilla, R1 through R5 and S1 through S5) and a key levels system (Monday range, daily/weekly/monthly/yearly highs, lows, midpoints, and opens). Both draw on the same chart and share the same 500-line / 500-label object budget that TradingView imposes on a single indicator.

## Pivot points

- Choice of calculation method: Fibonacci or Camarilla
- Timeframe options: Auto, Hourly, Daily, Weekly, Monthly, Quarterly, Yearly, Biyearly, Triyearly, Quinquennial
- Lookback period controls how many past pivot segments stay drawn on the chart
- Optional "use daily-based values" so intraday charts still calculate pivots from daily OHLC
- Each level (P, R1/S1 through R5/S5) can be shown or hidden and has its own color
- Labels can sit on the left or right edge of each pivot segment
- Alerts for every level, either on close crossing the level or on the bar range simply touching it

A large lookback combined with every level enabled can approach the 500-object cap on its own; if levels stop drawing, that's usually why.

## Key levels

- Monday range: high, low, and open, tracked live as Monday's session develops and reset each new week
- Daily: today's open, previous day's high/low, previous day's midpoint
- Weekly: current week's high/low/midpoint (live), previous week's high/low/midpoint
- Monthly: current month's high/low/midpoint (live), previous month's high/low/midpoint
- Yearly: current year's high/low/midpoint/open (live), previous year's high/low/midpoint

"Current period" levels update as the period develops. "Previous period" levels are locked in from the prior, completed period.

Levels that land within a configurable tick tolerance of each other get merged onto a single line, with their labels combined (for example "PDH | CW High") instead of drawing overlapping lines on top of each other. Line length, merge tolerance, and per-group color/style/width are all adjustable, and master overrides let you force one color, one line style, or one width across every key level at once.

## Notes

Pivot and key-level drawings redraw on the last bar rather than persisting historical objects bar by bar, which keeps the chart responsive even with several years of history loaded. Camarilla level math is left as originally sourced.
