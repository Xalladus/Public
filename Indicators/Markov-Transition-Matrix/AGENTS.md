# Markov Transition Matrix

## Overview

`Markov-Transitions.ps` is a standalone Pine Script v6 analytic indicator. It
owns input declarations, higher-timeframe state classification, cumulative
transition and same-state continuation statistics, live chart-candle coloring,
and table rendering. It does not import any libraries, place trades, emit
alerts, or create chart objects other than its table and optional candle colors.

The indicator classifies each selected analysis-timeframe return as `Bullish`,
`Neutral`, or `Bearish`; tallies transitions across the configured candle gap;
and shows both a transition matrix and optional same-state average-move panel.

Dependency note: this indicator has no imported-library dependencies.

---

## Input Groups

| Group | Input | Purpose |
|-------|-------|---------|
| `Settings` | `Threshold (%)` | Absolute N-bar return threshold that separates bullish and bearish states from neutral. |
| `Settings` | `Lookback Window` | Number of analysis-timeframe candles between the reference close and current close for classification. |
| `Settings` | `Timeframe` | Analysis timeframe supplied to `request.security()`; it must be at least the chart timeframe. |
| `Settings` | `Candle Diff` | Number of classified analysis candles between the transition's from-state and to-state. `1` is the next-state model. |
| `Settings` | `Display` | Selects `Both`, `Transition Matrix`, or `Average Move`. |
| `Appearance` | `Color Candles` | Enables chart-timeframe candle colors based on the live higher-timeframe threshold boundaries. |
| `Appearance` | `Bull`, `Neut`, `Bear` | Colors used for the live-state candle coloring. |
| `Appearance` | `Table Position` | Chart anchor for the rendered table. |

---

## Phases

1. Input Resolution
   Map the timeframe and table-position labels, resolve panel visibility, and
   calculate the table row layout.
2. Higher-Timeframe Compute
   Call `computeTally(...)` in `request.security()` to update and return a
   cumulative 37-value snapshot in the selected timeframe context.
3. Live-State Color Compute
   Retrieve the higher-timeframe reference close, derive the bullish and
   bearish threshold lines, and color each chart candle from its current close.
4. Runtime Gates
   Require the chart timeframe to be less than or equal to the selected
   analysis timeframe and require a correctly sized snapshot before rendering.
5. Matrix Render
   Render cumulative state-transition probabilities, simple `P^2`/`P^3`
   powers, raw counts, and the current-row directional bias.
6. Average-Move Render
   When enabled, render close, high, low, and up/down statistics for each
   same-state continuation: Bull -> Bull, Neutral -> Neutral, and Bear -> Bear.

---

## Runtime State And Snapshot Contract

The indicator has no UDTs. `computeTally(...)` owns all persistent state inside
the `request.security()` expression, so it persists independently in the
selected analysis-timeframe context.

| State / value | Type | Purpose |
|---------------|------|---------|
| `tally` | `var float[]` (9) | Cumulative transition counts stored at `fromCls * 3 + toCls`. |
| `stats` | `var float[]` (27) | Nine continuation-stat accumulators for each source class. |
| `lastCls` | `var int` | Most recently classified analysis bar; `-1` means no valid class yet. |
| `prevT` | `var int` | Previous analysis-bar time; prevents duplicate processing within one higher-timeframe bar. |
| `barsSeen` | `var int` | Unique analysis-bar count used to gate classification until lookback history exists. |
| `clsWindow` | `var int[]` | Rolling class window; its oldest class becomes the from-state once it reaches `Candle Diff`. |
| `res` | fresh `float[]` (37) | Snapshot returned from the security call. It copies persistent tally and stat data without exposing the live arrays. |

### State Codes And Transition Buckets

| Class | Code | Meaning |
|-------|------|---------|
| Bearish | `0` | Return is below `-Threshold (%)`. |
| Neutral | `1` | Return is between the two thresholds, inclusive. |
| Bullish | `2` | Return is above `Threshold (%)`. |

| From \\ To | Bearish (`0`) | Neutral (`1`) | Bullish (`2`) |
|-----------|----------------|---------------|---------------|
| Bearish (`0`) | `0` | `1` | `2` |
| Neutral (`1`) | `3` | `4` | `5` |
| Bullish (`2`) | `6` | `7` | `8` |

The rendered rows and columns use Bullish, Neutral, Bearish order, but internal
storage always uses the numeric `fromCls * 3 + toCls` order above.

### `computeTally()` Return Layout

| Index range | Contents |
|-------------|----------|
| `[0..8]` | Transition counts using the flat bucket mapping above. |
| `[9]` | Latest class code as a float (`-1`, `0`, `1`, or `2`). |
| `[10..18]` | Bearish -> Bearish continuation stats. |
| `[19..27]` | Neutral -> Neutral continuation stats. |
| `[28..36]` | Bullish -> Bullish continuation stats. |

Each nine-value same-state stat block is: sample count, cumulative high percent,
high absolute move, low percent, low absolute move, close percent, close
absolute move, count of closes above the origin close, and count below it.

---

## Function Map

### Label, State, And Formatting Helpers

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `tfString(_label)` | `_label`: input timeframe label | `string` | Maps the UI label to the timeframe string passed to `request.security()`. Unknown labels use `"D"`. |
| `tfTitle(_label)` | `_label`: input timeframe label | `string` | Maps the UI label to the compact table-title label. |
| `classify(_pct, _thr)` | `_pct`: percent return; `_thr`: threshold percent | `int` | Returns `2` above `_thr`, `0` below `-_thr`, otherwise `1`. |
| `tblPos(_label)` | `_label`: table-position label | `position` | Maps the UI label to a Pine `position.*` value. |
| `clsName(_c)` | `_c`: class code | `string` | Converts a state code to its display name, or `"N/A"` for an unsupported value. |
| `fPct(_v)`, `fSignedPct(_v)` | `_v`: percent value | `string` | Formats an unsigned or signed one-decimal percentage. |
| `fSignedAbs(_v)` | `_v`: price move | `string` | Formats an absolute move using `format.mintick` and adds `+` when positive. |
| `fStatMain(_pct, _abs)` | `_pct`: close percent; `_abs`: close move | `string` | Builds the main average-close table-cell text. |
| `fStatDetail(_label, _pct, _abs)` | `_label`: metric name; `_pct`: percent; `_abs`: move | `string` | Builds high/low continuation detail text. |
| `fUpDnText(_upPct, _dnPct)` | `_upPct`: share closing above origin; `_dnPct`: share closing below origin | `string` | Builds the up/down continuation detail text. |

### Higher-Timeframe Computation

| Function | Arguments | Returns | Meaning / side effects |
|----------|-----------|---------|------------------------|
| `computeTally(_lookback, _step, _thr)` | `_lookback`: return window; `_step`: transition candle gap; `_thr`: classification threshold | fresh `float[]` of length `37` | Mutates persistent security-context arrays and state once per unique analysis bar, then returns their snapshot. Same-state continuation stats are recorded only when from-state equals to-state. |

`computeTally(...)` computes the N-bar state from `close[_lookback]`. Once
`clsWindow` contains `_step` prior classes, it records a transition from its
oldest class to the current class. For same-state transitions, it measures the
high, low, and close excursion from `close[_step]` across that interval.

---

## Function Hierarchy

```text
Markov Transition Matrix
|
|-- Input and display resolution
|   |-- tfString(...)
|   |-- tfTitle(...)
|   `-- tblPos(...)
|
|-- Higher-timeframe state engine (inside request.security)
|   |-- computeTally(...)
|   |   |-- classify(...)
|   |   |-- transition tally mutation
|   |   `-- same-state continuation-stat mutation
|   `-- returned 37-value snapshot
|
|-- Chart-context live state
|   |-- request.security(..., close[i_lookback])
|   `-- barcolor(...)
|
`-- Last-bar table render
    |-- transition matrix and bias bar
    |   |-- clsName(...)
    |   `-- fPct(...)
    `-- average-move panel
        |-- fStatMain(...)
        |-- fStatDetail(...)
        `-- fUpDnText(...)
```

---

## Standard Integration Pattern

This is a standalone indicator, so users add `Markov-Transitions.ps` directly
to a chart; there is no importable public API. The critical in-file pattern is
to keep the stateful function inside `request.security()` and consume the
returned snapshot only after its shape is validated:

```pine
string tfSel = tfString(i_timeframe)
bool tfOk = timeframe.in_seconds(timeframe.period) <= timeframe.in_seconds(tfSel)

float[] data = request.security(
    syminfo.tickerid,
    tfSel,
    computeTally(i_lookback, i_candleDiff, i_threshold),
    lookahead = barmerge.lookahead_off)

bool dataOk = not na(data) and array.size(data) == 37

if barstate.islast and tfOk and dataOk
    int lastCls = int(array.get(data, 9))
    // Read transition counts from [0..8] and continuation stats from [10..36].
```

For live chart-candle coloring, the script separately retrieves
`close[i_lookback]` from the same analysis timeframe and compares each chart
close to the derived live bullish/bearish boundaries. That coloring can change
before the analysis-timeframe bar closes; the cumulative tally does not.

---

## Execution Ownership

| Ownership area | Indicator owns | Not owned / not provided |
|----------------|----------------|--------------------------|
| Inputs | All `input.*` declarations, defaults, groups, tooltips, and appearance colors. | No external configuration library. |
| State classification | Return-window calculation and three-state classification in the selected security context. | No rolling-window or alternate state model. |
| Statistics | Cumulative transition counts and same-state high/low/close continuation aggregates. | No full multi-step Markov matrix multiplication. |
| Rendering | Table lifecycle, matrix, prediction bar, average-move panel, and candle colors. | No labels, lines, boxes, trades, alerts, or broker automation. |
| Runtime validation | Timeframe and snapshot-shape gates before table rendering. | No automatic conversion to a supported chart timeframe. |

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Keep `computeTally()` inside `request.security()` | Its `var` arrays and counters must persist in the selected higher-timeframe context. Computing values in chart scope first changes the analysis model. |
| Process each analysis bar once | Keep the `time != prevT` gate. Without it, repeated chart bars within one analysis bar can increment the cumulative statistics multiple times. |
| Preserve the 37-value snapshot contract | The render code assumes `[9]` is the latest class and `[10..36]` are three nine-value continuation-stat blocks. Update producer and consumer together if this changes. |
| Preserve state-code and bucket order | `0` = Bearish, `1` = Neutral, `2` = Bullish, and each count lives at `fromCls * 3 + toCls`. Display order is intentionally different. |
| `Candle Diff` changes the transition definition | A value greater than `1` compares the current class with the class that many analysis candles earlier; it is not repeated one-step matrix multiplication. |
| Continuation stats are same-state only | The average-move panel records only Bull -> Bull, Neutral -> Neutral, and Bear -> Bear events. It does not summarize all transitions. |
| `P^2` and `P^3` are not Markov-chain forecasts | They are powers of the displayed one-step cell probability, not probabilities from matrix multiplication. |
| Keep `tfOk` as a render and color gate | The chart timeframe must not exceed the selected analysis timeframe. Do not silently render results on an unsupported chart timeframe. |
| Live candle colors and tally timing differ | Candle colors use the current chart close against live higher-timeframe threshold boundaries; `computeTally()` advances only once per unique analysis bar. |
| The data is cumulative | Counts and continuation statistics cover all loaded history in the security context. Changing inputs recompiles the script and rebuilds state. |
| Table lifecycle is last-bar only | The table is deleted and recreated in the `barstate.islast` block so the selected anchor and mode-specific row count stay correct. |
