# AGENTS.md — Coding Agent Instructions for TradingView Scripts

This is the single source of instructions for coding agents working on the **Midnight Pickle (MP) TradingView Pine Script** repository. It follows the [AGENTS.md](https://agents.md) convention, so Codex, Cursor, Copilot, Gemini-based tools and others read it directly; `CLAUDE.md` imports it for Claude Code.

> [!NOTE]
> Edit **this** file. `CLAUDE.md` is a pointer to it, kept that way so the instructions
> cannot drift between tools.

This is a personal project with one user-developer. Git history is the record of changes; per-script changelogs live in each script's documentation file.

---

## What's here

Pine Script **v6** indicators and strategies, pasted into TradingView by hand. There is no build, package manager, or test suite.

| Folder | Contents |
|---|---|
| `Master Suite Indicators/` | Combined indicators (EMA & PVSR Master, Pivots + Key Levels Master) and RSI Divergence MP v2/v3 |
| `individual_scripts/` | The two master scripts split into standalone indicators |
| `scalping_strategies/` | BTC strategies: Camarilla Range Bounce, EMA 9/21 Pullback, and `btc_asia_sweep/` |

`README.md` is the public index. It lists every script, links its docs, and contains a repository structure tree.

Folder and file names are linked from `README.md` and the `.md` docs. If you rename or move anything, update every link in the same change.

## Verifying changes

- There is no local Pine compiler. Compile by pasting into the TradingView Pine Editor, or through the TradingView MCP tools (`pine_set_source` → `pine_smart_compile` → `pine_get_errors`) if they are connected.
- For strategies, check the Strategy Tester still runs and the on-chart stats table renders.
- Don't claim a script compiles unless you compiled it.

---

## Keeping docs in sync (the main source of drift)

Every script has a doc next to it. When you change inputs, defaults, levels, alerts, or behavior, update the doc **in the same change**:

| Script | Doc |
|---|---|
| `ema_pvsr_master.pine` | `ema_pvsr_master.description.md` |
| `pivots_and_key_levels_master_.pine` | `pivots_and_key_levels_master.description.md` |
| `rsi_divergence_mp_v2.pine` / `_v3.pine` | `rsi_divergence_mp_v2 README.md` / `rsi_divergence_mp_v3 README.md` |
| `camarilla_range_bounce.pine` | `camarilla_range_bounce.description.md` |
| `ema_pullback_continuation.pine` | `ema_pullback_continuation.description.md` |
| `btc_asia_sweep/midnight_pickle_asia_sweep_v1.pine` | `btc_asia_sweep/asia_sweep_strategy_guide.md` (and `btc_asia_sweep_playbook.html` if the rules change) |
| `individual_scripts/*.pine` | Covered by the matching master script's doc |

Also update `README.md` when you add, move, rename, or delete a script or folder: the feature section, the Installation Guide list, and the Repository Structure tree.

Write docs from the code, not from memory: state real defaults, real option names, and real label text.

`scalping_strategies/btc_asia_sweep/Asia Range Sweep v3.md` is a manual trading playbook, not a script doc. Its times are deliberately UTC/NZ-based and differ from the script's defaults.

---

## Conventions and constraints

### Master vs individual scripts
- `individual_scripts/` are splits of the masters with "logic unchanged". A fix in one should be mirrored in the other (e.g. `pivots_and_key_levels_master_.pine` ↔ `pivots_.pine` + `key_levels_.pine`; `ema_pvsr_master.pine` ↔ `ema_sma_`, `pvsr_candles_`, `pvsr_volume_`, `pvsr_vector_zones_`).
- The split versions are native overlays, so they drop the `force_overlay=true` arguments the master needs. Don't copy those back in.
- The three standalone PVSR scripts each hold their own copy of the classification inputs and maths. Keep them identical to each other and to the master.

### Camarilla (two different mappings, on purpose)
- **Pivots indicators** (`pivots_and_key_levels_master_.pine`, `pivots_.pine`) use a deliberate custom variant: plotted R1–R5 come from H2–H6, so R3 = H4. The code says "Do not 'correct' it against published formulas." Leave the math alone unless the user asks.
- **`camarilla_range_bounce.pine`** uses textbook levels (R3 = H3, R4 = H4). So the indicator's R3 is the strategy's R4. Keep the warning about this in both docs.

### Pine patterns used here
- **Previous-period data:** `request.security(..., expr[1], lookahead = barmerge.lookahead_on)`. The `[1]` is what makes lookahead safe. Never use `lookahead_on` without it.
- **Current developing HTF levels** use `lookahead_off` with no offset and are only drawn on `barstate.islast`. They are wrong on historical bars (and in Bar Replay). Don't feed them into alerts or strategies.
- **Object budget:** 500 lines / labels / boxes per script. Redraw on the last bar and delete old objects rather than drawing per bar.
- **Timezones:** anchor sessions to the exchange that sets the schedule (`America/New_York`, `Asia/Tokyo`, `UTC`), never to NZ local time. NZ and US DST flip about five weeks apart.
- **No `timeframe=` in `indicator()`** for scripts that call `alert()` or `line.new()` — Pine rejects it (CE10080).

### Strategies
- Keep `commission_value = 0.045` (percent), `slippage = 2`, `process_orders_on_close = true`, `calc_on_every_tick = false` unless asked.
- R-multiple stats are measured **per position** (snapshot `strategy.netprofit` at open and at flat, divide by initial cash risk to the original stop). Don't switch to per-exit counting, which double-counts scale-outs.
- These are untested hypotheses, not proven systems. Docs should keep that tone: no performance claims.

### Style
- The master and individual indicators start with the MPL-2.0 header and `// © midnight pickle`, then `//@version=6`. The strategies and RSI Divergence scripts open with `//@version=6` and a banner comment. Match the file you're in.
- Script titles, short titles, stats-table headers and alert messages carry the `MP` mark (e.g. `Key Levels MP`, `MP Camarilla Range Bounce`). Use `MP`, not emoji.
- Inputs are grouped with emoji-prefixed `group=` names and have tooltips explaining *why*.
- Comments explain *why*, often recording the bug or trap that motivated the code. Keep that style. In the RSI Divergence scripts, changes vs. the TradingView original are tagged `// [MP]` (and `// [MP v3]`).
- File names use snake_case. Several end in a trailing underscore (`pivots_and_key_levels_master_.pine`, `key_levels_.pine`); keep existing names so links don't break.

### Commits
Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`) with a body explaining the reason.
