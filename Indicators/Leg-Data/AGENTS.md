# Market Structure Analysis

`Leg-Swing-Data.pine` is a standalone Pine Script v6 indicator that combines
private market-structure events with session-aware ATR normalization to study
confirmed swing behavior in two modes:

- `Leg Analysis` measures completed swing-to-swing legs.
- `Revisit Analysis` tracks what happens after a confirmed swing level is
  touched or broken.

This indicator owns inputs, runtime orchestration, CSV logging, and table
rendering. `MarketStructureLibrary` owns pivot confirmation, structure state,
and event generation. `ATRxSession` owns session ATR math. `InputLibrary`
provides shared enums and formatting helpers.

---

## Dependencies

| Library | Alias | Role |
|---------|-------|------|
| `OneCleverGuy/InputLibrary/4` | `iLib` | Shared timezone and text-size enums. |
| `OneCleverGuy/MarketStructureLibrary/30` | `MS` | Confirmed pivots, structure state, external direction, and pivot events. |
| `OneCleverGuy/ATRxSession/4` | `ATRxS` | Session-only ATR calculation. |

---

## Input Groups

| Group | Purpose |
|-------|---------|
| `Analysis` | Switches between `Leg Analysis` and `Revisit Analysis`. |
| `Session & Timezone` | Session window, timezone, and outside-start session classification rule. |
| `ATR Settings` | Number of completed sessions used for session ATR averaging. |
| `Market Structure` | Pivot lookback/lookahead plus optional ATR-based pivot confirmation. |
| `Table` | Table anchor position and text size. |
| `CSV Export` | Toggles Pine Log CSV emission. |
| `Debug` | Enables diagnostic labels for newly completed records. |

---

## Local UDTs

| Type | Fields | Purpose |
|------|--------|---------|
| `PivotSnapshot` | `price`, `bar`, `timeUnix`, `isHigh` | Active confirmed pivot level stored while waiting for a revisit/break. |
| `ZigzagPoint` | `price`, `bar`, `timeUnix`, `isHigh`, `inSession`, `dirAtCreation`, `extHighPrice`, `extLowPrice` | Reduced two-point zigzag state used to build completed legs. |
| `LegData` | `isBullish`, `startPrice`, `endPrice`, `startBar`, `endBar`, `startTime`, `endTime`, `priceDistance`, `atrMultiple`, `isWithTrend`, `isCounterTrend`, `brokeStructure`, `startInSession`, `endInSession`, `dirAtStart` | Final stored record for one completed leg. |
| `LegCategoryStats` | `count`, `atrCount`, `totalPrice`, `totalAtr`, `minPrice`, `maxPrice`, `minAtr`, `maxAtr` | Aggregated leg metrics for one table category. |
| `RevisitData` | `pivotIsHigh`, `pivotPrice`, `pivotTime`, `pivotBar`, `breakPrice`, `breakTime`, `breakBar`, `breakInSession`, `ageMs`, `firstNextIsHigh`, `hasNextHigh`, `nextHighPrice`, `nextHighTime`, `nextHighPriceDist`, `nextHighTimeDist`, `hasNextLow`, `nextLowPrice`, `nextLowTime`, `nextLowPriceDist`, `nextLowTimeDist`, `atrAtCompletion` | One revisit record from pivot confirmation through post-break swing completion. |
| `RevisitCategoryStats` | `count`, `totalAgeMs`, `maxAgeMs`, `nextHighCount`, `atrValidCount`, `totalPH`, `minPH`, `maxPH`, `totalAtrH`, `totalTH`, `minTH`, `maxTH`, `totalPL`, `minPL`, `maxPL`, `totalAtrL`, `totalTL`, `minTL`, `maxTL` | Aggregated revisit metrics for one table category. |

---

## Runtime State

| State | Purpose |
|-------|---------|
| `pivots`, `msState`, `msDir` | Live market-structure engine state returned by `MS.tick(...)`. |
| `persistentDir` | Last known non-ranging external direction; used to classify legs even during `Unknown` / `Ranging` phases. |
| `zigzag` | Two-point rolling zigzag cache used to collapse same-side pivots and confirm legs. |
| `legs` | Persistent completed leg records for `Leg Analysis`. |
| `activePivots` | Confirmed pivot levels still waiting to be touched or broken in `Revisit Analysis`. |
| `pendingRevs` | Revisit records that have broken but have not yet seen both next confirmed high and low. |
| `confirmedRevs` | Completed revisit records used by the table and CSV export. |
| `csvHdrLogged` | One-shot guard so the active mode logs a legend line and CSV header once per script run. |
| `tbl` | Single persistent table reused on the last bar with mode-specific layouts. |

---

## Phases

1. Shared Runtime Context
   Resolve session membership, session ATR, outside-start rule, and active mode.
2. Market Structure Engine
   Tick `MS.State`, confirm pivots, update events, and snapshot current
   external high/low plus persistent direction.
3. Break Detection
   In revisit mode, compare the current bar against every active pivot level
   and open a `RevisitData` record the moment price reaches that level.
4. Confirmed Pivot Processing
   For each new confirmed pivot event:
   - update zigzag state and maybe append a `LegData` record
   - register the new pivot for revisit tracking
   - advance pending revisit records with the new confirmed swing
5. CSV Export
   Emit a mode-specific legend line, a CSV header line, then newly completed
   rows into Pine Logs.
6. Table Display
   On the last bar, aggregate either leg stats or revisit stats and render the
   matching table layout.
7. Debug Labels
   When enabled, annotate only newly completed leg or revisit records.

---

## Function Map

### Shared Helper Functions

| Function | Meaning |
|----------|---------|
| `getExternalPivotPrice(...)` | Safe accessor for external pivot prices from the `MS.Pivot[]` store. |
| `isLegInSession(...)` | Resolves final in/out session classification using the outside-start rule. |
| `formatDuration(...)` | Converts milliseconds into `Xd Yh Zm` text. |
| `formatPriceAndAtr(...)` | Formats price distance plus optional ATR multiple for table cells. |

### Leg Building

| Function | Meaning |
|----------|---------|
| `getTrendRelationship(...)` | Converts bullish/bearish leg direction plus outer direction into `WithTrend` / `CounterTrend` flags. |
| `isStructureBreak(...)` | Tests whether the leg endpoint exceeded the external level that existed at the leg start. |
| `processConfirmedPivot(...)` | Collapses same-side swings, confirms completed legs, and appends `LegData` records. |

### Revisit Tracking

| Function | Meaning |
|----------|---------|
| `updatePendingRevisits(...)` | Fills pending revisit records with the next confirmed high and low after the break. |

### Statistics And CSV

| Function | Meaning |
|----------|---------|
| `addLegStats(...)`, `addRevisitStats(...)` | Mutate category stat maps in place. |
| `computeLegStats(...)` | Aggregates `legs` into category keys used by the leg table. |
| `computeRevisitStats(...)` | Aggregates `confirmedRevs` into category keys used by the revisit table. |
| `buildLegCsvRow(...)` | Serializes one `LegData` record into a CSV row string. |
| `buildRevisitCsvRow(...)` | Serializes one `RevisitData` record into a CSV row string. |

### Table Rendering

| Function | Meaning |
|----------|---------|
| `getLegSubcategoryLabels(...)`, `getLegSubcategorySuffixes(...)` | Provide the ordered row labels and category keys for leg table breakdowns. |
| `formatLegStats(...)` | Formats a `LegCategoryStats` record into count/avg/min/max text. |
| `formatRevisitDistance(...)`, `formatRevisitTime(...)`, `formatRevisitStats(...)` | Format revisit metrics for multi-line table cells. |
| `setTableTitleRows(...)` | Builds the shared title block for either table mode. |
| `renderLegTable(...)` | Renders the `Leg Analysis` table. |
| `renderRevisitTable(...)` | Renders the `Revisit Analysis` table. |

---

## Function Hierarchy

```text
Market Structure Analysis
|
+-- Shared runtime context
|   +-- ATRxS.getSessionAtr(...)
|   +-- time(..., i_session, tz)
|
+-- Market structure engine
|   +-- MS.initState(...)
|   +-- MS.tick(...)
|   +-- MS.updateExternalDirection(...)
|   +-- getExternalPivotPrice(...)
|
+-- Revisit break detection
|   +-- activePivots scan
|   +-- array.push(pendingRevs, RevisitData.new(...))
|
+-- Confirmed pivot processing
|   +-- processConfirmedPivot(...)
|   |   +-- getTrendRelationship(...)
|   |   +-- isStructureBreak(...)
|   +-- updatePendingRevisits(...)
|
+-- CSV export
|   +-- buildLegCsvRow(...)
|   +-- buildRevisitCsvRow(...)
|
+-- Table display
|   +-- computeLegStats(...)
|   |   +-- addLegStats(...)
|   +-- computeRevisitStats(...)
|   |   +-- addRevisitStats(...)
|   +-- renderLegTable(...)
|   |   +-- formatLegStats(...)
|   |   +-- setTableTitleRows(...)
|   +-- renderRevisitTable(...)
|       +-- formatRevisitStats(...)
|       +-- formatRevisitDistance(...)
|       +-- formatRevisitTime(...)
|       +-- setTableTitleRows(...)
|
+-- Debug
    +-- label.new(...)
    +-- formatDuration(...)
```

---

## Standard Integration Pattern

This is a standalone indicator, so users add it directly to the chart. The
internal runtime pattern is:

```pine
import OneCleverGuy/InputLibrary/4 as iLib
import OneCleverGuy/MarketStructureLibrary/30 as MS
import OneCleverGuy/ATRxSession/4 as ATRxS

var MS.Pivot[] pivots                 = array.new<MS.Pivot>()
var MS.State msState                  = MS.State.new()
var array<ZigzagPoint> zigzag         = array.new<ZigzagPoint>()
var array<LegData> legs               = array.new<LegData>()
var array<PivotSnapshot> activePivots = array.new<PivotSnapshot>()
var array<RevisitData> pendingRevs    = array.new<RevisitData>()
var array<RevisitData> confirmedRevs  = array.new<RevisitData>()

float sessATR = ATRxS.getSessionAtr(tz, i_session, i_atrCount)
MS.Event[] events = MS.tick(
    pivots, msState, true,
    i_lookBack, i_lookAhead, TOUCH_DETECT_BARS,
    i_useATR, sessATR, i_atrMultiple, float(na))

if MS.hasNewConfirmed(events)
    for evt in events
        // Build legs from alternating confirmed pivots.
        newLegs += processConfirmedPivot(...)

        // In revisit mode, register the pivot and complete any pending revisits.
        array.push(activePivots, PivotSnapshot.new(...))
        newRevs += updatePendingRevisits(...)

if barstate.islast
    renderLegTable(...) or renderRevisitTable(...)
```

---

## Execution Ownership

| Ownership area | Indicator owns | Imported libraries own |
|----------------|----------------|------------------------|
| Inputs | All `input.*` declarations, defaults, grouping, and tooltips | None |
| Structure events | Which `MS` outputs are consumed and how they are persisted | Pivot detection, event emission, structure state, external direction |
| Session normalization | How ATR is applied to tables and CSV output | Session ATR calculation itself |
| Runtime arrays | `zigzag`, `legs`, `activePivots`, `pendingRevs`, `confirmedRevs`, and header guards | None |
| Logging | CSV legend strings, CSV row ordering, and debug labels | None |
| Rendering | Table layout, row categories, colors, and debug labels | None |

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Keep `persistentDir` sticky | Leg trend classification intentionally uses the last non-`Unknown` / non-`Ranging` direction so every later leg has stable direction context. |
| `processConfirmedPivot(...)` assumes a two-point zigzag | The function intentionally trims the zigzag back to two points after a direction change. Expanding that store changes leg semantics. |
| Same-side pivot collapse is deliberate | Consecutive highs or lows replace the previous same-side point only when they are more extreme. Do not count both unless you want a different leg model. |
| Revisit mode measures confirmed structure after the break | A revisit record is not complete until it has both the next confirmed high and the next confirmed low after the break bar. |
| `activePivots` only matters in revisit mode | Leg mode does not populate or consume pivot revisit state. |
| `breakPrice` is the stored level, not the live wick price | Revisit distances are measured from the pivot level that was broken/touched, not from the exact intrabar excursion. |
| Signed revisit distances are direction-aware | High breaks treat upward continuation as positive; low breaks treat downward continuation as positive. |
| Session classification is asymmetric by design | Starts inside are always `In`. Starts outside only become `In` when the leg ends inside and `Starting Outside →` is set to `In Session`. |
| CSV logging is mode-specific and one-shot | The legend/header pair logs once per script run for the active mode after the first completed record appears. |
| Table row keys are string contracts | `renderLegTable(...)` and `renderRevisitTable(...)` assume the exact category keys built by `computeLegStats(...)` and `computeRevisitStats(...)`. |
| The indicator depends on private libraries | This file is repo-internal. Publishing it publicly requires the imported libraries to be available in the target TradingView account/version path. |
