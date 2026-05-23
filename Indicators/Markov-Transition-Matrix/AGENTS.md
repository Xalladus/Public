# Markov Transition Matrix

## Overview

`Markov-Transitions.ps` is a standalone Pine Script v6 indicator that builds a
3x3 state-transition matrix from higher-timeframe returns and renders it as a
chart table.

- The script classifies each completed analysis-timeframe bar as `Bullish`,
  `Neutral`, or `Bearish`.
- Classification is based on the N-bar percent return, where N is
  `Lookback Window`.
- Every time a new higher-timeframe bar confirms, the script records one
  transition from the previous class to the current class.
- The matrix is cumulative across all available history inside the selected
  `request.security()` context.
- The display also derives a simple directional bias from the current row:
  `P(Bull) - P(Bear)`.

This file is indicator-only. There are no library imports and no local UDTs.

## Responsibilities

What this script owns:

- input declarations
- higher-timeframe classification and transition tallying
- table creation and rendering
- summary and prediction-bar formatting

What this script does not do:

- rolling-window transition analysis
- true multi-step Markov matrix multiplication
- strategy entries, alerts, or broker automation
- chart drawings beyond the single `table`

## Inputs

| Input | Purpose |
|-------|---------|
| `Threshold (%)` | Minimum absolute N-bar return needed to leave `Neutral` |
| `Timeframe` | Higher timeframe used for analysis via `request.security()` |
| `Lookback Window` | Number of candles used to compute each return |
| `Table Position` | Anchor position for the rendered table |

## Classification Model

For each confirmed bar in the selected analysis timeframe:

1. Compute percent return from `close[_lookback]` to `close`.
2. Compare that return to `+threshold` and `-threshold`.
3. Convert the result into one of three integer classes:
   - `2` = Bullish
   - `1` = Neutral
   - `0` = Bearish
4. If a previous class already exists, increment the transition bucket
   `lastCls * 3 + cls`.

Flat array bucket mapping:

| From \\ To | Bearish (`0`) | Neutral (`1`) | Bullish (`2`) |
|-----------|----------------|---------------|---------------|
| Bearish (`0`) | `0` | `1` | `2` |
| Neutral (`1`) | `3` | `4` | `5` |
| Bullish (`2`) | `6` | `7` | `8` |

The display reorders those classes visually as:

- rows: Bullish, Neutral, Bearish
- columns: Bullish, Neutral, Bearish

That reordered presentation is visual only. The internal tally storage stays in
the flat `from * 3 + to` format above.

## Runtime Phases

1. Input Resolution
   Convert the timeframe dropdown and table-position dropdown into Pine values.
2. Higher-Timeframe Compute
   Run `computeTally()` inside `request.security()` for the selected timeframe.
3. Data Gate
   Reject chart timeframes above the selected analysis timeframe.
4. Table Lifecycle
   Recreate the table on the last bar using the selected anchor position.
5. Matrix Render
   Compute row totals, percentages, `P^2`, `P^3`, and raw counts for each cell.
6. Summary Render
   Derive the current-row probabilities and directional bias from the most
   recent confirmed class.
7. Prediction Bar Render
   Fill a 9-segment bar from `-100` to `+100` using the bias value.

## Function Map

### Helper Functions

- `tfString(_label)`
  Maps the UI timeframe label to the Pine timeframe string passed into
  `request.security()`.
  - `"1 Day"` -> `"D"`
  - `"12 Hour"` -> `"720"`
  - `"8 Hour"` -> `"480"`
  - `"6 Hour"` -> `"360"`
  - `"4 Hour"` -> `"240"`
  - `"1 Hour"` -> `"60"`

- `classify(_pct, _thr)`
  Converts an N-bar percent return into the internal class code.
  - returns `2` when `_pct > _thr`
  - returns `0` when `_pct < -_thr`
  - otherwise returns `1`

- `tblPos(_label)`
  Maps the table-position dropdown into a Pine `position.*` enum.

- `clsName(_c)`
  Converts internal class integers back into display labels for the table and
  summary row.

- `fPct(_v)`
  Rounds a numeric percentage to one decimal place and appends `%`.

### Core Computation

- `computeTally(_lookback, _thr)`
  The only stateful computation function in the file.
  It runs inside the requested higher-timeframe context and returns a
  10-element float array:
  - `[0..8]` are cumulative transition counts
  - `[9]` is the most recent class code

  Internal state persisted with `var` inside the security context:
  - `tally`: the 9 transition buckets
  - `lastCls`: previous confirmed class
  - `prevT`: previous higher-timeframe bar timestamp
  - `barsSeen`: number of unique higher-timeframe bars processed

  Important behavior:
  - updates only when `time != prevT`, so the tally advances once per confirmed
    analysis-timeframe bar
  - ignores bars until there are more than `_lookback` bars available
  - skips classification if the reference close is `na` or zero
  - returns a fresh snapshot array each call rather than exposing the live
    `tally` array directly

## Execution Flow

### Higher-Timeframe Data

- `tfSel` is derived by `tfString(i_timeframe)`.
- `tfOk` enforces the script rule that the chart timeframe must be less than or
  equal to the chosen analysis timeframe.
- `data` is the `request.security()` result for `computeTally(...)` using
  `lookahead = barmerge.lookahead_off`.

### Table Render

The render block runs only on `barstate.islast`.

- `tbl` is deleted and recreated each render pass so the anchor position stays
  consistent with the current input.
- The script first checks:
  - invalid chart timeframe vs analysis timeframe
  - missing or malformed returned data
- If data is valid, the script renders:
  - title row
  - column headers
  - three matrix rows with paired detail rows
  - summary row
  - prediction-bar row

## Matrix Semantics

Each matrix cell shows four values across two visual rows:

- main row: transition probability from the current source state to the target
  state
- detail row left: `P^2`
- detail row middle: `P^3`
- detail row right: raw transition count

`P^2` and `P^3` here are simple powers of the single-step probability, not
second-step or third-step Markov forecasts derived from matrix multiplication.
Treat them as persistence/intensity transforms, not full chained-transition
projections.

## Summary And Bias

The summary logic uses only the row that matches the most recent confirmed
class:

- `pBull` = probability of next state being Bullish
- `pNeut` = probability of next state being Neutral
- `pBear` = probability of next state being Bearish

Directional bias:

- `pred = pBull - pBear`
- positive values tint the summary and bar green
- negative values tint them red
- `0` is neutral

The 9-segment bar maps the range `[-100, +100]` across the row and fills only
the portion covered by the current bias.

## Critical Invariants

- `computeTally()` must remain inside `request.security()`; moving its results
  to the chart scope first would silently break the higher-timeframe analysis.
- The flat transition array must stay aligned with `fromCls * 3 + toCls`.
- `lastCls` uses `-1` as the pre-seeded "no prior state" sentinel.
- `tfOk` must gate rendering when the chart timeframe exceeds the analysis
  timeframe.
- The returned snapshot array must stay length `10`, because the render block
  assumes index `9` is the last class.

## Safe Edit Points

- Input options or defaults: `Inputs`
- New timeframe labels: `tfString()`
- State thresholds or classification policy: `classify()`
- Return-window logic or tally rules: `computeTally()`
- Table palette or wording: `Table Render`
- Bias presentation only: summary and prediction-bar subsection

## Pitfalls

- Do not replace `request.security(..., computeTally(...))` with a call that
  passes precomputed chart-scope values. That would sample the wrong timeframe.
- Do not interpret `P^2` and `P^3` as true 2-step and 3-step transition
  probabilities. The code does not perform matrix powers.
- Do not remove the `time != prevT` gate inside `computeTally()`. That gate
  prevents multiple increments on the same higher-timeframe bar.
- Do not reorder class integers unless you also update:
  - `classify()`
  - `clsName()`
  - the flat-array indexing logic
  - the display row/column maps
- The matrix is cumulative over all loaded history, so changing inputs resets
  the script and rebuilds the tally from scratch.
