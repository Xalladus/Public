# Session Volume Averages

[`Session-Volume.ps`](Session-Volume.ps) is a standalone Pine Script v6 separate-pane indicator. It owns all inputs, timezone conversion, three independent session trackers, completed volume history, two baselines per session, global plots, and optional diagnostics. It has no imported dependencies, lower-timeframe requests, built-in alerts, or order execution.

## Current release

- Script: `Session-Volume.ps`.
- Published release identifier and visibility: Unconfirmed.
- Confirmation source/date: no verified publication information supplied as of 2026-10-31.
- Local input-layout and documentation edits do not establish a new published release.

## Dependencies

No imported libraries. Follow the shared [style guide](../../docs/style-guide.md), [AGENTS.md standard](../../docs/agents-file-standard.md), [guide standard](../../docs/guide-standards.md), and [publication markup reference](../../docs/tradingview-publication-markup.md).

## Files and navigation

Use this `AGENTS.md` as the folder entry point and maintainer reference. Keep navigation here rather than adding a separate README.

| File | Purpose |
| --- | --- |
| [Session-Volume.ps](Session-Volume.ps) | Indicator source. |
| [Session-Volume-Guide.md](Session-Volume-Guide.md) | Formulas, lifecycle, and output behavior. |
| [publication-document.txt](publication-document.txt) | Current TradingView description and input-help update notes. |

## Input Groups

| Group | Inputs and defaults |
| --- | --- |
| Timezone | One shared timezone, New York by default. |
| Sessions | Three compact rows labeled Session 1, Session 2, and Session 3. Each contains the enable checkbox, unlabeled hours, then unlabeled color. All start enabled. |
| Sessions | Session 1 defaults to 08:00-17:00/purple; Session 2 to 03:00-12:00/blue; Session 3 to 19:00-04:00/amber. |
| Sample Size | 20 completed sessions; allowed range 5-100. |
| Appearance | Up/Down contains the `close >= open` color followed by the `close < open` color. |
| Debug | Disabled by default; last-bar averages, tracked position, active/idle state, and sample target. |

## Phases

1. Convert the timezone and append `:23456` to each window's session string.
2. Initialize three persistent trackers from inputs.
3. Call `processBar` for all three trackers: archive on an observed exit, reset on entry, append volume while active.
4. Call `getOutputs` after processing to obtain historical baselines for the current bar position.
5. Gate the histogram by any active enabled window; gate each baseline independently.
6. Plot global outputs and optionally render Debug on the last bar.

## Local Enums and UDTs

`Timezones` includes `utc`, `exch`, `ny`, `chi`, `lax`, `tor`, `lon`, `ber`, `tyo`, `hkg`, `sgp`, and `syd`. `exch` resolves to `syminfo.timezone`; the others convert to their displayed timezone string.

| Type | Fields and types | Meaning |
| --- | --- | --- |
| `SessionRecord` | `barVolumes`: `array<float>` | Copied volume samples in ordinal chart-bar order. |
| `SessionRecord` | `totalVolume`: float; `barCount`: int | Completed occurrence's volume total and observed bar count. |
| `SessionTracker` | `enabled`: bool; `sessionStr`: string; `sessionColor`: color | Settings captured by the persistent tracker constructor. |
| `SessionTracker` | `wasInSession`: bool | Previous observed membership. |
| `SessionTracker` | `currentBarIndex`: int; `currentTotalVol`: float | Current count and total, starting at zero. |
| `SessionTracker` | `currentBarVols`: `array<float>` | Live observed samples; replaced on reset. |
| `SessionTracker` | `history`: `array<SessionRecord>` | Oldest-to-newest completed records, capped by sample size. |

`tracker1`, `tracker2`, and `tracker3` use `var`. Each owns independent mutable arrays. Input changes restart the script and reconstruct history.

## Function Map

| Helper | Arguments | Result and responsibility |
| --- | --- | --- |
| `timezoneToString` | `Timezones _this` receiver | `string`; returns exchange timezone or enum display text. |
| `resetLive` | `SessionTracker _this` receiver | Mutates count/total to zero and replaces the live volume array. |
| `archiveSession` | `SessionTracker _this`, `int _maxHistory` | Copies a positive-length live record to history and shifts an excess oldest record. |
| `getSessionAvgPerBar` | `SessionTracker _this` receiver | `float`; mean of each retained session's total/count, or zero. |
| `getBarPositionAvg` | `SessionTracker _this`, `int _pos` | `float`; mean at zero-based position, excluding records too short to contribute, or zero. |
| `isInSession` | `string _sessStr`, `string _tz` | `bool`; chart-bar membership through `time`. |
| `processBar` | `SessionTracker _tracker`, `string _tz`, `float _vol`, `int _sampleSize` | Updates session lifecycle, sample arrays, totals, and membership in place. |
| `getOutputs` | `SessionTracker _tracker`, `string _tz` | `[float sessAvg, float barPosAvg, bool active]`; reads history at the current ordinal position. |

## Function Hierarchy

```text
Input resolution -> timezoneToString -> three SessionTracker constructors
Execution
  +-- processBar(tracker1/2/3)
  |     +-- isInSession
  |     +-- archiveSession -> copy live record and trim history
  |     +-- resetLive
  |     +-- append volume and increment count
  +-- getOutputs(tracker1/2/3)
  |     +-- isInSession
  |     +-- getSessionAvgPerBar
  |     +-- getBarPositionAvg(currentBarIndex - 1)
  +-- histogram and six global baseline plots
Debug -> optional last-bar table
```

## Standard Integration Pattern

Add this source as an indicator; there is no exported API. Within the file, preserve processing before reads. For an existing tracker and resolved inputs, the normal handoff is:

```pine
processBar(tracker1, tzString, volume, i_sampleSize)
[s1SessAvg, s1BarAvg, s1Active] = getOutputs(tracker1, tzString)
```

Keep `plot` calls at global scope and preserve the existing active/enable gates.

## Rules And Pitfalls

| Rule | Detail |
| --- | --- |
| Shared timezone | All windows use the same timezone; do not describe them as separately timezone-aware market presets. |
| Weekday mask | Source appends `:23456`. Verify overnight/weekday transitions in TradingView rather than inferring a complete market calendar. |
| Session-average weighting | Average the per-session means with equal session weight; do not replace this with pooled total volume divided by pooled bars. |
| Positional baseline | Positions are ordinal chart bars. Skip records shorter than the requested position; do not imply clock-time alignment. |
| Archive trigger | An observed inside-to-outside membership transition archives the record. No separate date-rollover or continuous-session splitter exists. |
| Warmup | Partial completed history is used immediately; empty history returns zero. The first loaded session may be partial. |
| Missing volume | No `nz(volume)` replacement is applied. Do not promise validated output for unavailable volume feeds. |
| Output gates | Enabled inactive baselines are zero; disabled baselines are `na`. Histogram volume is shown once when any enabled window is active. |
| Input layout | One short, clear first title per `group`/`inline` row; all later titles are exactly `""`. Align `group` columns and use tooltips for order. Preserve defaults, activity/display arguments, and ordering. |
| Compact session rows | Keep all three numbered rows under the shared Sessions group. The enable checkbox carries the Session 1/2/3 title and shares its row's inline key with hours and color; do not restore separate session headings or redundant Hours labels. |
| Runtime preservation | This documentation/layout pass does not change accumulation, archival, formulas, plot gates, or Debug table allocation. Validate realtime behavior separately. |
| Publication | Maintain supported TradingView markup in publication copy, preserve earlier descriptions when adding update notes, and keep test plans/results in `prep/`. |

## Verification

The documentation/layout pass locally checked 14 preserved inputs, four valid inline rows, unchanged runtime code, publication markup, and the guide's arithmetic example. Pine compilation, formula checks in TradingView, session boundaries, realtime updates, and publication preview remain pending. No separate validation plan or results files are currently present. Local documentation checks do not establish runtime or published-release status.
