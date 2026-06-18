# Candle POI Direction

`Candle-POI-Direction.ps` is the public Pine Script v6 indicator-side
orchestration layer for the candle-pattern plus point-of-interest direction
stack. It owns inputs, public pivot adaptation, runtime arrays, chart drawings,
score composition, and calls into `CandlePOIDirectionLibrary`.

The imported library owns exported scoring UDTs, OB/FVG resolution, state
transitions, raw score math, normalization, and optional tables. This public
indicator must not import `MarketStructureLibrary`.

---

## Dependencies

| Library | Alias | Role |
|---------|-------|------|
| `OneCleverGuy/CandlePOIDirectionLibrary/1` | `DIR` | Swing/zone UDTs, OB/FVG resolution, lifecycle updates, scoring helpers, tables |

The indicator uses a local public pivot adapter based on `ta.pivothigh()` and
`ta.pivotlow()`. It does not depend on private market-structure libraries.

---

## Input Groups

| Group | Purpose |
|-------|---------|
| `Shared` | Drawing extension horizon for active levels and zones. |
| `ATR` | ATR length plus candle equality/body tolerances. |
| `Structure` | Pivot confirmation, equality grouping, swing visibility, swing styling. |
| `Order Blocks` | OB visibility and bull/bear fill colors. |
| `Fair Value Gaps` | FVG visibility, minimum ATR gap, colors, and FVG proximity mode. |
| `Scoring Weights` | Category weights for structure, OBs, FVGs, and candle columns. |
| `Volume Confluence` | Volume SMA length and min/max volume multipliers. |
| `Display` | Score table and pattern legend toggles. |
| `Debug` | Pine Log diagnostics. |

---

## Local UDTs

| Type | Fields | Purpose |
|------|--------|---------|
| `SwingVisual` | `swingLine`, `swingLabel`, `obBox`, `obLabel`, `pivotIdx`, `isHigh`, `touched` | Indicator-owned chart objects for one swing and its co-located OB. |
| `FVGVisual` | `fvgBox`, `fvgLabel` | Indicator-owned chart objects for one FVG. |
| `PublicPivot` | `isHigh`, `price`, `timeUnix`, `relClass` | Confirmed public pivot adapter record using `DIR.dir_RelClass`. |
| `PublicEvent` | `code`, `pivotIndex` | Minimal event wrapper for newly confirmed pivots. |

---

## Runtime State

| State | Purpose |
|-------|---------|
| `pivots` | Confirmed public pivots used to classify new HH/HL/LH/LL/EH/EL points. |
| `msDir` | Current derived leg direction represented as `DIR.dir_Direction`. |
| `cacheOpen`, `cacheClose`, `cacheVolRatio`, `cachePrevTime`, `cacheNextTime` | Time-keyed bar caches used to reconstruct delayed pivot-neighbor data. |
| `swingVisuals`, `swings`, `obZones` | Parallel arrays; index `i` always describes the same structural pivot. |
| `fvgVisuals`, `fvgZones` | Parallel arrays; index `j` always describes the same FVG. |

---

## Phases

1. Cache Update: compute ATR and volume ratio, then update time-keyed OHLC maps.
2. Structure Engine: confirm public pivots, classify `DIR.dir_RelClass`, derive
   `legStartTime`, and update `DIR.dir_Direction`.
3. Pivot Materialization: create swing records, swing drawings, OB records, and
   OB drawings for new confirmed pivots.
4. Swing Tick: detect sweeps/breaks, update swing state, and credit nearest
   opposing swings, OBs, and FVGs.
5. Zone Tick: advance OB/FVG zone lifecycle with `DIR.dir_tickZone(...)` and
   restyle local drawings.
6. Active POI Extension: extend the newest unbroken swing high/low, bull/bear
   OB, and bull/bear FVG.
7. FVG Scan: detect confirmed 3-candle FVG windows, create zone records, and
   draw visuals.
8. Leg Pruning: delete objects and shift arrays whose `anchorTime` is before
   the current `legStartTime`.
9. Score Assembly: compute and normalize structure/OB/FVG scores, apply candle
   conviction and volume, then compute weighted bull/bear totals.
10. Display: optionally call the library score and legend tables.

---

## Function Map

### Cache And Structure Helpers

| Function | Meaning |
|----------|---------|
| `lookupBarData(...)` | Reads pivot candle, previous candle, and next candle OHLC from time-keyed maps. |
| `classifyRelClass(...)` | Converts a new pivot into `DIR.dir_RelClass` against the newest same-side pivot. |
| `findNewestPivotIdx(...)` | Finds the newest high or low pivot in the public pivot store. |
| `passesAtrConfirmation(...)` | Optional ATR-distance gate against the newest opposite-side pivot. |
| `directionName(...)` | Converts `DIR.dir_Direction` to debug text. |
| `computeExtendedTime(...)` | Converts bar-extension count into a future bar-time coordinate. |

### Swing Styling

| Function | Meaning |
|----------|---------|
| `swingStateToLineStyle(...)`, `swingStateToLineColor(...)`, `swingLineWidth(...)` | Derive line presentation from swing state and opposing-credit flag. |
| `swingStateText(...)`, `swingStateToLabelColor(...)` | Derive compact label text/color for sweeps, breaks, and credit. |

### Zone Styling

| Function | Meaning |
|----------|---------|
| `zoneStateToBorderStyle(...)`, `zoneStateToBorderColor(...)`, `zoneStateToBorderWidth(...)` | Derive box border presentation from zone lifecycle. |
| `zoneStateToFillColor(...)` | Derive fill opacity from zone lifecycle. |
| `zoneStateText(...)`, `zoneStateTextColor(...)` | Derive compact OB/FVG label text/color. |

---

## Call Hierarchy

```text
Candle POI Direction
|
+-- Bar cache update
|   +-- map.put(...) OHLC, volume ratio, prev/next times
|
+-- Structure engine
|   +-- ta.pivothigh(...) / ta.pivotlow(...)
|   +-- passesAtrConfirmation(...)
|   +-- classifyRelClass(...)
|   +-- findNewestPivotIdx(...)
|
+-- Pivot events
|   +-- DIR.dir_newSwing(...)
|   +-- lookupBarData(...)
|   +-- DIR.dir_resolveOBHigh(...) / DIR.dir_resolveOBLow(...)
|   +-- DIR.dir_newZone(...)
|   +-- line.new(...) / box.new(...) / label.new(...)
|
+-- Runtime ticking
|   +-- DIR.dir_applySwingEvent(...)
|   +-- DIR.dir_findCreditSwingIdx(...)
|   +-- DIR.dir_findCreditZoneIdx(...)
|   +-- DIR.dir_tickZone(...)
|
+-- FVG scan
|   +-- DIR.dir_detectFVG(...)
|   +-- DIR.dir_newZone(...)
|
+-- Scoring
|   +-- DIR.computeCandleScores(...)
|   +-- DIR.computeStructureScore(...)
|   +-- DIR.computeZoneScore(...)
|   +-- DIR.structureMaxRaw(...) / DIR.zoneMaxRaw(...)
|   +-- DIR.normalizeScore(...)
|
+-- Display
    +-- DIR.dir_drawScoreTable(...)
    +-- DIR.dir_drawLegendTable(...)
```

---

## Standard Integration Pattern

This is a standalone indicator, so consumers add it directly to a chart. The
implementation pattern inside the file is:

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR

var PublicPivot[] pivots = array.new<PublicPivot>()
var SwingVisual[] swingVisuals = array.new<SwingVisual>()
var DIR.dir_Swing[] swings = array.new<DIR.dir_Swing>()
var DIR.dir_Zone[] obZones = array.new<DIR.dir_Zone>()
var FVGVisual[] fvgVisuals = array.new<FVGVisual>()
var DIR.dir_Zone[] fvgZones = array.new<DIR.dir_Zone>()

// Confirm pivot -> create visual/record triplet.
DIR.dir_Swing swingRec = DIR.dir_newSwing(relClass, isHigh, price, pivotTime, volRatio)
array.push(swings, swingRec)
array.push(obZones, obZoneRec)
array.push(swingVisuals, SwingVisual.new(swingLn, swingLbl, obBx, obLbl, pivotIndex, isHigh, false))

// Tick records, then score the active leg.
[bullStRaw, bearStRaw] = DIR.computeStructureScore(swings, legStartTime, msDir, i_volMinMult, i_volMaxMult)
[bullObRaw, bearObRaw] = DIR.computeZoneScore(obZones, legStartTime, msDir, i_fvgMode, i_volMinMult, i_volMaxMult)
```

---

## Execution Ownership

| Ownership area | Indicator owns | Library owns |
|----------------|----------------|---------------|
| Inputs | All `input.*` declarations and user-facing defaults | None |
| Structure | Public pivot confirmation, rel-class assignment, leg start derivation | `DIR.dir_RelClass`, `DIR.dir_Direction`, scoring multipliers |
| Drawings | Lines, boxes, labels, extension, deletion, and styling | Optional score/legend tables |
| State arrays | Parallel visual/record arrays and pruning | UDT definitions and lifecycle transition helpers |
| Scores | Category weighting and final bull/bear percentages | Raw category scores and normalization helpers |

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Do not add private imports | The public indicator must remain dependent only on `CandlePOIDirectionLibrary`. |
| Keep parallel arrays aligned | Every push, set, or shift of `swingVisuals`, `swings`, and `obZones` must happen in matching code paths. The same applies to `fvgVisuals` and `fvgZones`. |
| Use `xloc.bar_time` consistently | Drawings are keyed by Unix bar time, not `bar_index`. Mixing coordinate systems will misplace objects. |
| Preserve time-keyed caches | Pivots confirm after delay; direct current-bar history references do not reconstruct pivot-neighbor candles reliably. |
| `legStartTime` is the active window | Records before it are previous-leg state and should be pruned or ignored. |
| OB placeholders are intentional | `obZones` can contain `na` when no valid OB resolves; the placeholder keeps arrays aligned. |
| Ticking and scoring are confirmed-bar oriented | Scores update on `barstate.isconfirmed` so live and historical readings agree. |
| Tables are library-owned | The indicator decides whether to call them, but table cell layout belongs to `DIR`. |
