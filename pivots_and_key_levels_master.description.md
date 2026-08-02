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

### Level naming

The Camarilla calculation here is a deliberate custom variant, not the textbook equation, and the internal values are mapped onto the plotted lines with a one-step shift. H1 and L1 are not plotted:

| plotted as | R1 | R2 | R3 | R4 | R5 |
|---|---|---|---|---|---|
| comes from | H2 | H3 | H4 | H5 | H6 |

Worth knowing if you cross-reference other material. This script's R3 carries the H4 value, which published Camarilla writing treats as the breakout level, so it is not the "far extreme, rarely reached" that R3 means in standard pivot notation. TradingView's built-in Pivot Points Standard set to Camarilla plots five levels and puts (previous high / previous low) × previous close at its R5; that same value sits at this script's R5, but nothing in between lines up, so the two are not directly comparable.

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

Because the key levels draw only on the last bar, the current week, month, and year lines show the developing period correctly on a live chart. In bar replay they will show the previous period's extreme instead, since replay makes a historical bar the last bar. Don't reuse those values in an alert or port them into a strategy without rebuilding them, for the same reason.

## Changelog

**v2 (2 August 2026)**

- Documented the R1 to R5 level mapping in the code and in tooltips on the R3, R4, and R5 toggles. No calculation changed. The mapping shifts by one step against published Camarilla notation, and R3 in particular carries the breakout level rather than a far extreme, which is easy to misread.
- Documented why the current-period week, month, and year levels use `request.security` with `lookahead_off` and no offset, what that means in bar replay, and why rebuilding them natively would break the monthly and yearly levels on intraday charts.

**v1**

- Merged Pivot Enhanced and Key Levels into one script sharing the 500-line and 500-label object budget.
