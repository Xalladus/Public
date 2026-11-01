# Session High/Low Average and Range

[`Session-Average.ps`](Session-Average.ps) is a Pine Script v6 overlay. It owns session detection, historical high/low distances from the session open, projected average and custom levels, chart drawings, labels, and distance text. This indicator uses chart bars; it does not request intrabars, place trades, or emit alerts.

## Current release

- Script: `Session-Average.ps`.
- Published release identifier and visibility: Unconfirmed.
- Confirmation source/date: no verified publication information supplied as of 2026-10-31.
- Local input-alignment edits do not establish a new published release.

## Dependencies

The executable source imports `OneCleverGuy/InputLibrary/<version>` as `iLib` for timezone, line style/width, text size, and alignment enums and conversion methods. The import retains its existing numeric pin. This legacy library has no owning source or release record in the repository; do not silently migrate it to UtilityLibrary1CG.

Follow the shared [style guide](../../docs/style-guide.md) and [AGENTS.md standard](../../docs/agents-file-standard.md).

## Files and navigation

Use this `AGENTS.md` as the folder entry point and maintainer reference. Keep navigation here rather than adding a separate README.

| File | Purpose |
| --- | --- |
| [Session-Average.ps](Session-Average.ps) | Indicator source. |
| [Session-Average-Guide.md](Session-Average-Guide.md) | Source-traced calculation and drawing pipeline. |
| [publication-document.txt](publication-document.txt) | Current TradingView description and layout update notes. |

## Input Groups

| Group | Controls |
| --- | --- |
| Session | Timezone appears first, followed by Session hours on a separate line. Session Period controls the calculation sample; Sessions Displayed controls drawing retention. |
| Lines | Open, Avg H/L, 2x Avg H/L, 3x Avg H/L, and Custom 1/2 rows each contain visibility, color, width, style, and label toggle. |
| Lines | Avg Fill, 2x Fill, and 3x Fill contain visibility then fill color. Factor/Mode contains the custom multiplier then Average/Range base. Labels contains global visibility, side, and size. |
| Display Distances | Distances toggles distance text. Custom Text contains the custom-color toggle, color, and size. Align H/V contains horizontal then vertical alignment. |

## Phases

1. Resolve the selected timezone and parse session hours with `setSessionInfo`.
2. At session start, create a session record and high/low drawing records. When the existing sample gate permits, use the preceding record's statistics to project average multiples and custom levels from the new open.
3. During the session, update its high and low from chart bars.
4. After session close, append high/open and open/low distances, store averages and extrema, and prune sample and drawing history with the existing retention rules.
5. On the last bar, show or clear the insufficient-session warning table.

## UDTs and Runtime State

| Type or store | Responsibility |
| --- | --- |
| `sessionInfo` | Parsed hours, scheduled duration, and session OHLC state. |
| `lineInputs` | Resolved line, label, and fill appearance. |
| `lineData` | Average and extreme distances plus the lines, labels, and boxes owned by a session side. |
| `sessionArr` | Retained session records. |
| `highDistanceArr`, `lowDistanceArr` | Historical positive distances from the open used for statistics. |
| `highLineDataArr`, `lowLineDataArr` | High/low statistics and chart-object handles retained per session. |

### Local field contracts

| Type | Fields and types | Meaning |
| --- | --- | --- |
| `sessionInfo` | `hours`, `startTime`, `endTime`: string | Original session specification and parsed HHMM endpoints. |
| `sessionInfo` | `openTimeSinceMidnight`, `closeTimeSinceMidnight`, `duration`: int | Parsed timing offsets and elapsed session duration. |
| `sessionInfo` | `openHour`, `openMinute`, `closeHour`, `closeMinute`: int | Clock components. |
| `sessionInfo` | `highPrice`, `lowPrice`, `openPrice`: float, default `na` | Observed session prices. |
| `lineInputs` | `showLine`, `showLabel`, `showBox`: bool; `lineColor`, `boxColor`: color | Visibility and colors for one level family. |
| `lineInputs` | `lineWidth`: int; `lineStyle`, `labelTextSize`, `labelText`: string | Converted drawing constants and label content. |
| `lineData` | `rangeHigh`, `rangeLow`, `averageRange`: float, default `na` | Maximum, minimum, and mean distances for one side. |
| `lineData` | `openLine`, `averageLine`, `doubleLine`, `tripleLine`, `customLine1`, `customLine2`: line | Optional retained line handles, initially `na`. |
| `lineData` | `openLabel`, `averageLabel`, `doubleLabel`, `tripleLabel`, `customLabel1`, `customLabel2`: label | Optional retained label handles, initially `na`. |
| `lineData` | `averageBox`, `doubleBox`, `tripleBox`: box; `lineStart`, `lineEnd`: int | Average band handles, initially `na`, and drawing timestamps. |

## Function Map

| Function | Arguments and result |
| --- | --- |
| `setSessionInfo` | `string this`, `string _userTimezone`; returns parsed `sessionInfo` including overnight duration. |
| `createAllShapes` | `int _x1`, `int _x2`, `float _price`, `float _boxPrice`, `float _openPrice`, `lineInputs _lineInput`, `string _labelSide`, `bool _isHigh`, `bool _showAllLabels`; returns `[line, label, box]` and creates allowed drawings. |
| `deleteElements` | Optional `line[] _lines = na`, `label[] _labels = na`, `box[] _boxes = na`; deletes supplied objects. |

## Function Hierarchy

```text
Input resolution
  +-- i_timeZone.timezoneToString()
  +-- enum conversion methods -> persistent lineInputs
Execution
  +-- i_sessionTime.setSessionInfo(TIMEZONE)
  +-- Session start -> previous statistics -> createAllShapes()
  +-- Active session -> update high/low
  +-- Session end -> distance statistics -> prune -> deleteElements()
  +-- Last bar -> warning table
```

## Standard Integration Pattern

Add the source directly as an indicator; it has no exported API. Preserve the existing timezone conversion, persistent appearance objects, session arrays, and start/update/end ordering. Average mode projects mean distances; Range mode projects maximum historical distances from the open, separately for highs and lows.

## Rules And Pitfalls

| Rule | Detail |
| --- | --- |
| Inline titles | Each `group` and `inline` row has a short, clear first title and exactly `""` for every later title. Do not use Unicode padding, spaces, or separator labels. Tooltips explain the controls in order. |
| Input preservation | Preserve defaults, enum options, bounds, display settings, input order, and inline keys during layout edits. Each row's `group` arguments align, with one blank line after the block. |
| Fill order | Avg Fill, 2x Fill, and 3x Fill start with their visibility checkbox and short title, followed by the unlabeled color picker. Keep visibility first; trailing checkboxes on the separate line-appearance rows control labels. |
| Session setup order | Timezone is the first input and Session is second, on separate lines without `inline`. Keep this order so the timezone is selected before entering session hours. |
| Labels | Labels require the line, its label toggle, and the global Labels switch. Fill visibility is independent of line visibility. |
| Distances | Custom Text disables only the custom-color override; text size still applies. Low-side vertical alignment mirrors the selected high-side alignment. |
| Coordinates | Drawings use timestamps with `xloc.bar_time`; preserve this model. |
| Existing sample behavior | Input bounds, prior-record indexing, and statistics/pruning order are existing runtime behavior. Alignment edits do not validate or repair small sample settings or insufficient history. |
| Averaging order | Statistics are stored before excess samples are removed. The stored average can include the extra occurrence at that update; do not promise an exact N-session statistic without addressing and validating this behavior. |
| Documentation ownership | Keep the guide, publication copy, and navigation consistent with source. Use supported TradingView markup in publication text, preserve existing copy when adding release notes, and keep test plans/results under `prep/`. |
| Scope | Do not migrate the legacy library, change session calculations, rename variables, or reflow unrelated runtime code during an input-layout update. |

## Verification

TradingView compilation, visual alignment, and chart behavior require validation in TradingView. No local Pine compiler is provided.

- 2026-10-31: local source comparison confirmed all 53 input defaults, types, order, bounds, options, display settings, groups, and inline keys are unchanged. All 15 inline rows have a non-empty first title, empty later titles, aligned `group` arguments, and defined tooltip references. Input titles contain no Unicode padding.
- The prologue and all code after Inputs match the previous source. `git diff --check` passed. These checks establish local source consistency, not Pine compilation or chart behavior.

Pine compilation, small-sample indexing, averaging at the pruning boundary, session timing, and publication preview remain pending. No separate validation plan or results files are currently present; do not treat source review as a compilation, chart, or publication result.
