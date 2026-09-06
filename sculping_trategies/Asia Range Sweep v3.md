# Asia Range Sweep — BTC Playbook (NZ Edition, v2)

High-probability scalping framework for the Asia session, built for a New Zealand schedule.
Status: **untested hypothesis** — see Section 10 before risking capital.

---

## 1. Reading the Clock Correctly

Do **not** memorise fixed local times. Anchor everything to two invariants:

- **Japan never moves**: Tokyo open is always **00:00 UTC** (12:00 PM NZST / 1:00 PM NZDT).
- **New York moves twice a year**: US cash close is **20:00 UTC** during US daylight saving (mid-March to early November) and **21:00 UTC** otherwise.

This produces four calendar regimes:

| Period | US Close (UTC) | US Close (NZ local) | Notes |
|---|---|---|---|
| Apr – early Nov | 20:00 | 8:00 AM NZST | Current setup |
| Early Nov – mid-Mar | 21:00 | **10:00 AM NZDT** | Most guides get this wrong |
| Mid-Mar – early Apr | 20:00 | 8:00 AM NZST | Both clocks in flux — skip these weeks |
| Late Sep – early Nov | 20:00 | 9:00 AM NZDT | NZDT active, US still EDT |

**Rule: recheck the schedule at every DST flip (NZ: late Sep / early Apr; US: mid-Mar / early Nov). Do not trade on transition weeks — your box windows will be misaligned while you recalibrate.**

---

## 2. Session Schedule (anchored to UTC)

| Phase | UTC | NZST | NZDT | Action |
|---|---|---|---|---|
| US Settlement | 20:00* | 8:00 AM | 9:00 AM | Volume exits; consolidation begins |
| Prep Check | +15 min | 8:15 AM | 9:15 AM | Heatmap, DXY, news calendar (Section 3) |
| **Box Build** | US close → +2h | 8:00–10:00 AM | 9:00–11:00 AM | Mark Asia High / Low / Mid on 15M |
| **Power Window** | 23:00–01:00 | 11:00 AM–1:00 PM | 12:00–2:00 PM | Liquidity ramp into/through Tokyo open. Hunt sweeps |
| Dead Zone | 04:30–05:30 | 2:30–3:30 PM | 3:30–4:30 PM | Tokyo lunch halt (11:30–12:30 JST). No new entries |

\* Use 21:00 UTC during northern winter (see Section 1).

Note the correction vs. v1: the real Asian liquidity ramp sits **around the Tokyo equity open (midnight UTC)**, not two hours before it. The 10:00–12:00 window in the old version caught the tail end of nothing.

---

## 3. Pre-Session Prep (15 minutes)

1. **Liquidation heatmap** (Coinglass/Hyblock): mark clusters sitting within **0.5% of either box edge**. No cluster near an edge → no magnet → skip today. Remember heatmaps show *estimated* leverage, not orders. Context, not gospel.
2. **News calendar**: any red-folder US event (CPI, FOMC, NFP) within 4 hours → stand down for the day.
3. **DXY trend check**: strong DXY breakout day → longs off the table. Sharp DXY dump → shorts off the table. Correlation is regime-dependent; treat as a veto, not a signal.
4. **Mondays only**: identify the weekend CME gap (Friday 4:00 PM CT close vs Sunday 5:00 PM CT reopen). If the gap zone sits near an Asia Box edge, that edge becomes the preferred sweep target. If it doesn't, ignore gaps entirely.
5. **Prior day check**: if the US session ended in a large directional trend candle (>3% body), expect consolidation mid-trend, not a range sweep. Skip.

---

## 4. Constructing the Asia Box

On the 15M chart, box the full price range from US close to +2 hours.

- Top line = Asia High, bottom = Asia Low, midpoint = equilibrium.
- **Box quality filter**: height must be roughly **0.4%–2.0%** of price.
  - Under 0.4%: range is inside noise + fees. Skip.
  - Over 2.0%: it isn't consolidating, it's trending sideways violently. Skip.
- Once drawn, the box does not move. No redrawing mid-session.

---

## 5. Entry Trigger: Sweep & Reclaim

**Sweep qualification (all must be true):**
1. Wick penetrates the box edge by at least **0.1–0.15%** (less is noise).
2. Penetration reaches (or clearly heads toward) a marked liquidation cluster.
3. Sweep occurs inside the Power Window.
4. Reclaim: a **5M candle closes back inside the box** within **3 candles** of the extreme wick. Longer than that and it's a breakout, not a sweep — stand down.

Entry on the close of that reclaim candle.

- **Primary confirmation**: sweep volume expands, reclaim candle closes with the rejection wick visible.
- **Secondary (optional)**: RSI divergence on the 5M. Treat as a tiebreaker only — divergence alone confirms nothing.

**Directional bias**: take the trade the sweep implies — sweep of the Low → long; sweep of the High → short. One attempt per side per day maximum. If stopped out on the first, do **not** re-enter: a second push through the same level usually means genuine breakout.

---

## 6. Stops, Targets, Time Stop

**Stop loss** = beyond the absolute sweep wick by **max(0.15%, 0.5 x ATR-14 on the 15M)**.

This replaces the old flat 0.2%: fixed-percentage stops ignore volatility regime. In quiet conditions 0.2% may suffice; in volatile conditions it donates fees to faster traders. The ATR term adapts automatically; the floor prevents absurdly tight stops in dead markets.

- If the resulting stop distance exceeds **1.0%** from entry → skip (sweep too deep, trend risk too high).

**Targets:**
- TP1: box midpoint. Take 50% off, move stop to breakeven.
- TP2: opposite box edge. Close remaining.

**Expectancy gate (before entry):**
Distance to TP2 minus round-trip fees/slippage must be >= **1.5x** the stop distance. If not, pass. No exceptions — this single rule kills most losing scalping systems.

**Time stop:** if TP1 hasn't been reached within **90 minutes** of entry, exit at market. Asia ranges revert fast; if it hasn't worked quickly, the thesis is stale.

---

## 7. Position Sizing (this was missing entirely from v1/v2 drafts)

Risk per trade: **0.5% of equity** once proven, **0.25%** while validating.

```
Position size = (Equity x Risk%) / Stop distance %
```

Worked example ($100k account, 0.5% risk = $500):
- Box: High 118,800 / Low 117,400 / Mid 118,100
- Sweep of low to 117,400; 15M ATR = 0.40% → buffer = 0.20%
- SL = 117,400 x (1 − 0.002) = **117,165**
- Reclaim entry ≈ 117,700 → stop distance ≈ 0.45%
- Size = $500 / 0.0045 ≈ **$111k notional** (≈0.94 BTC equivalent)
- TP2 = 118,800 → +0.94% ≈ **2.1R** → passes the 1.5R gate → take it

Also enforce: max **one open position**, max **two trades per session**, hard daily loss limit of **1% equity** then platform closed.

---

## 8. No-Trade Rules (sit on hands when...)

- [ ] No liquidation cluster within 0.5% of either box edge
- [ ] Red-folder US news within 4 hours
- [ ] Prior US session closed with a >3% trend candle
- [ ] DXY in violent disagreement with trade direction
- [ ] Box height outside 0.4–2.0%
- [ ] DST transition week (Section 1)
- [ ] Weekends by default (thin books distort sweep behaviour; add back only with data proving weekend edge)
- [ ] Already taken two attempts today, or down 1%

---

## 9. Execution Checklist

1. Correct regime's clock loaded? (Table in Section 1)
2. Inside the Power Window?
3. Box valid (height 0.4–2.0%), drawn once, untouched?
4. Liquidation cluster confirmed near the swept edge?
5. Wick >= 0.1% beyond edge AND reclaimed (5M close back inside within 3 candles)?
6. DXY not vetoing?
7. News clear for 4 hours?
8. Stop placed per formula (not per feeling)?
9. Expectancy gate passes (>= 1.5R to TP2 after fees)?
10. Size calculated from risk %, not vibes?

Five-plus failures = no trade. There is always another session.

---

## 10. Validation Protocol — DO NOT SKIP

This strategy currently has **zero evidence behind it**. The concepts (liquidity sweeps, session volatility clustering, failed breakouts) are well-documented phenomena; whether *these exact rules* carry positive expectancy is unknown until measured.

**Protocol:**
1. Paper trade or backtest **minimum 50 qualified setups** (expect ~2 months of live sessions, or faster via historical replay on TradingView).
2. Log every setup that meets qualification — including ones you skip — with: date, box dimensions, sweep depth, stop distance, R-multiple outcome, screenshots.
3. Go live only after: sample win rate x average win > (sample loss rate x average loss) + fees, across 50+ trades.
4. Re-validate monthly. Edges in crypto microstructure decay as they become crowded.

**Journal template:**

| Date | Regime | Box H/L | Box height % | Cluster hit? | Entry | SL dist % | Result (R) | Notes |
|---|---|---|---|---|---|---|---|---|

---

## 11. Honest Limitations

- Liquidation heatmaps are estimates derived from assumed leverage tiers — accuracy varies by venue.
- DXY correlation flips sign across macro regimes; the veto helps most in trending dollar environments and adds little elsewhere.
- Sweep-and-reclaim is a known pattern. Its profitability compresses as more participants trade it. The validation loop in Section 10 is not optional bureaucracy — it's the difference between a system and a story.
- Fees matter enormously at this frequency. Maker entries where possible; know your venue's taker fee and slippage profile cold.
