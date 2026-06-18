# Candle POI Direction Library - Integration Guide

Candle POI Direction Library is a direction-scoring engine for Pine Script v6. It combines
structure, order blocks, fair value gaps, and candle-pattern conviction
into normalized bullish and bearish scores that a host indicator can display.

Published as `OneCleverGuy/CandlePOIDirectionLibrary/1`.

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR
import OneCleverGuy/CandlePatternLibrary/3 as CPL
```

## What the library owns

- Data types for order blocks, fair value gaps, swings, zones, and candle scores
- State machines for swing lifecycle and zone lifecycle
- Raw scoring helpers for structure, zones, and candle patterns
- Normalization helpers for converting raw scores into `[0, 1]`
- Optional score and legend tables

## What stays in the indicator

- All `input.*` declarations
- Any indicator-only UDTs for lines, boxes, or labels
- All visual drawing for structure, zones, labels, and debugging
- Execution flow that decides when to create, update, and prune records
- Any final weighting logic for total bullish vs bearish scores

## Dependency note

Candle POI Direction Library imports CandlePatternLibrary. Structure context uses
library-owned enums:

- `DIR.dir_RelClass` and `DIR.dir_Direction`
- `CPL.CandlePattern`, `CPL.TwoCandlePattern`, `CPL.ThreeCandlePattern`
- `CPL.CandleData`

If your script calls candle-pattern typed helpers directly, import `CPL`
alongside `DIR`.

## Typical workflow

1. Use a public pivot adapter to build and maintain your pivot store and current
   leg direction.
2. When a new swing pivot confirms, create a `DIR.dir_Swing` with
   `DIR.dir_newSwing(...)`.
3. Resolve raw order block geometry from the pivot using
   `DIR.dir_resolveOBLow(...)` or `DIR.dir_resolveOBHigh(...)`.
4. Detect three-candle fair value gaps with `DIR.dir_detectFVG(...)`.
5. Wrap valid order blocks and FVGs into `DIR.dir_Zone` records with
   `DIR.dir_newZone(...)`.
6. On each bar, mutate existing swings and zones with
   `DIR.dir_applySwingEvent(...)` and `DIR.dir_tickZone(...)`.
7. Compute raw category scores with `DIR.computeCandleScores(...)`,
   `DIR.computeStructureScore(...)`, and `DIR.computeZoneScore(...)`.
8. Normalize category scores with `DIR.normalizeScore(...)`, then combine them
   into your final bullish and bearish totals.
9. Optionally draw the table outputs with `DIR.dir_drawScoreTable(...)` and
   `DIR.dir_drawLegendTable(...)`.

## Quick start

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR
import OneCleverGuy/CandlePatternLibrary/3 as CPL

var DIR.dir_Swing[] swings = array.new<DIR.dir_Swing>()
var DIR.dir_Zone[] orderBlocks = array.new<DIR.dir_Zone>()
var DIR.dir_Zone[] fvgs = array.new<DIR.dir_Zone>()

// Example: create a swing when a pivot confirms.
DIR.dir_RelClass relClass = DIR.dir_RelClass.HH
DIR.dir_Swing swing = DIR.dir_newSwing(relClass, isHigh, pivotPrice, pivotTime, volRatio)
array.push(swings, swing)

// Example: resolve an order block from the pivot candle and its neighbors.
DIR.dir_OrderBlock obRaw = isHigh
    ? DIR.dir_resolveOBHigh(pivotHigh, pivotOpen, pivotClose, prevOpen, prevClose, nextOpen, nextClose, prevTime)
    : DIR.dir_resolveOBLow(pivotLow, pivotOpen, pivotClose, prevOpen, prevClose, nextOpen, nextClose, prevTime)

if obRaw.valid
    DIR.dir_Zone zone = DIR.dir_newZone(
        obRaw.top, obRaw.bottom, obRaw.anchorTime, obRaw.isBullish, bar_index, false, volRatio)
    array.push(orderBlocks, zone)

// Example: detect an FVG from a 3-candle window.
DIR.dir_FairValueGap fvgRaw = DIR.dir_detectFVG(high[2], low[2], high, low, minGap, time)
if fvgRaw.valid
    DIR.dir_Zone zone = DIR.dir_newZone(
        fvgRaw.top, fvgRaw.bottom, fvgRaw.anchorTime, fvgRaw.isBullish, bar_index, true, volRatio)
    array.push(fvgs, zone)

// Example: update zone state every bar.
for i = 0 to array.size(orderBlocks) - 1
    DIR.dir_Zone z = array.get(orderBlocks, i)
    z := DIR.dir_tickZone(z, high, low, close, bar_index)
    array.set(orderBlocks, i, z)

DIR.CandleScores cs = DIR.computeCandleScores(atrValue, equivAtrMult, bodyAtrMult)
[bullStructRaw, bearStructRaw] = DIR.computeStructureScore(swings, legStartTime, dir, volMinMult, volMaxMult)
[bullObRaw, bearObRaw] = DIR.computeZoneScore(orderBlocks, legStartTime, dir, DIR.dir_FvgWeightMode.Even, volMinMult, volMaxMult)
[bullFvgRaw, bearFvgRaw] = DIR.computeZoneScore(fvgs, legStartTime, dir, fvgMode, volMinMult, volMaxMult)

float structMax = DIR.structureMaxRaw(swings, legStartTime, volMaxMult)
float zoneMaxOb = DIR.zoneMaxRaw(orderBlocks, legStartTime, DIR.dir_FvgWeightMode.Even, volMaxMult)
float zoneMaxFv = DIR.zoneMaxRaw(fvgs, legStartTime, fvgMode, volMaxMult)

float bullStructNrm = DIR.normalizeScore(bullStructRaw, structMax)
float bearStructNrm = DIR.normalizeScore(bearStructRaw, structMax)
float bullObNrm = DIR.normalizeScore(bullObRaw, zoneMaxOb)
float bearObNrm = DIR.normalizeScore(bearObRaw, zoneMaxOb)
float bullFvgNrm = DIR.normalizeScore(bullFvgRaw, zoneMaxFv)
float bearFvgNrm = DIR.normalizeScore(bearFvgRaw, zoneMaxFv)

DIR.dir_drawScoreTable(
    bullStructNrm, bearStructNrm,
    bullObNrm, bearObNrm,
    bullFvgNrm, bearFvgNrm,
    cs,
    bullTotal, bearTotal,
    dir)
```

## Enums

### `DIR.dir_FvgWeightMode`

- `Even`: Every FVG scores with the same proximity multiplier of `1.0`.
- `PrioritizeBase`: Older zones closer to the leg origin get more weight.

### `DIR.dir_SwingState`

- `New`
- `Swept`
- `Broken`

### `DIR.dir_ZoneState`

- `New`
- `Tested`
- `Respected`
- `Swept`
- `Broken`

## UDTs

### `DIR.dir_OrderBlock`

- `valid`: `true` only when the resolved geometry is usable
- `top`
- `bottom`
- `anchorTime`
- `isBullish`

### `DIR.dir_FairValueGap`

- `valid`
- `top`
- `bottom`
- `anchorTime`
- `isBullish`

### `DIR.dir_Swing`

- `relClass`: Relative pivot classification
- `isHigh`: `true` for swing highs, `false` for swing lows
- `price`
- `anchorTime`
- `swingState`
- `brokeOpposing`
- `volRatio`

### `DIR.dir_Zone`

- `top`
- `bottom`
- `anchorTime`
- `isBullish`
- `creationBar`
- `zoneState`
- `brokeOpposing`
- `hasEntered`
- `everWickedFar`
- `resolved`
- `readyForRetest`
- `volRatio`

### `DIR.CandleScores`

- `bull1`, `bear1`: Single-candle conviction
- `bull2`, `bear2`: Two-candle conviction
- `bull3`, `bear3`: Three-candle conviction
- `abbr1`, `abbr2`, `abbr3`: Table abbreviations for each candle column

### `DIR.dir_PatternMeta`

- `fullName`: Human-readable pattern name for legends or external displays
- `abbr`: Compact table abbreviation
- `weightText`: Display value for the weight column
- `bullWeight`: Bullish score contribution in `[0, 1]`
- `bearWeight`: Bearish score contribution in `[0, 1]`

## Exported functions

### Order block and FVG helpers

- `DIR.dir_resolveOBLow(...)`: Builds a bullish order block from a swing-low pivot.
- `DIR.dir_resolveOBHigh(...)`: Builds a bearish order block from a swing-high pivot.
- `DIR.dir_detectFVG(...)`: Detects a bullish or bearish fair value gap from candles 1 and 3.

### Swing lifecycle

- `DIR.dir_newSwing(...)`: Creates a new swing record in state `New`.
- `DIR.dir_applySwingEvent(_swing, _isBreak, _isSweep)`: Mutates a swing after a break or sweep event.
- `DIR.dir_findCreditSwingIdx(_swings, _brokenWasHigh)`: Finds the newest non-broken swing on the opposite side.
- `DIR.dir_swingStateName(_state)`: Converts a swing state enum into a display string.

### Zone lifecycle

- `DIR.dir_newZone(...)`: Creates a new zone record in state `New`.
- `DIR.dir_tickZone(_zone, _high, _low, _close, _barIdx)`: Advances a zone state one bar at a time.
- `DIR.dir_findCreditZoneIdx(_zones, _brokenWasHigh)`: Finds the newest non-broken zone aligned with the direction of the break.
- `DIR.dir_zoneStateName(_state)`: Converts a zone state enum into a display string.

### Scoring helpers

- `DIR.volClamp(_ratio, _minMult, _maxMult)`: Clamps volume ratio into the user range.
- `DIR.relClassMultiplier(_relClass)`: Scores a swing based on its relative classification.
- `DIR.dirAlignmentMultiplier(_dir, _isBullishSignal)`: Adds a bonus or discount based on external direction alignment.
- `DIR.swingStateScore(_state)`: Converts swing state into a raw score.
- `DIR.zoneStateScore(_state)`: Converts zone state into a raw score.
- `DIR.computeSeekStructureBias(_swings, _zones, _legStartTime, _dir)`: Returns hidden continuation bias after respected or swept zones.
- `DIR.proximityWeight(_posInLeg, _fvgMode)`: Weights zones by position inside the current leg.

### Candle-pattern helpers

- `DIR.dir_singlePatternMeta(_cd)`: Returns single-candle name, abbreviation, display weight, and bull/bear weights.
- `DIR.dir_twoPatternMeta(_pat)`: Returns two-candle name, abbreviation, display weight, and bull/bear weights.
- `DIR.dir_threePatternMeta(_pat)`: Returns three-candle name, abbreviation, display weight, and bull/bear weights.
- `DIR.dir_singlePatternRows()`: Returns the single-candle legend metadata rows.
- `DIR.dir_twoPatternRows()`: Returns the two-candle legend metadata rows.
- `DIR.dir_threePatternRows()`: Returns the three-candle legend metadata rows.
- `DIR.computeCandleScores(_atr, _equivMult, _bodyMult)`: Analyzes the current 3-candle window and returns bundled scores plus display abbreviations.

### Category scoring

- `DIR.computeStructureScore(_swings, _legStartTime, _dir, _volMinMult, _volMaxMult)`: Returns raw bullish and bearish structure scores.
- `DIR.computeZoneScore(_zones, _legStartTime, _dir, _mode, _volMinMult, _volMaxMult)`: Returns raw bullish and bearish zone scores for order blocks or FVGs.
- `DIR.normalizeScore(_rawScore, _maxRaw)`: Converts a raw category total into a `[0, 1]` value.
- `DIR.structureMaxRaw(_swings, _legStartTime, _volMaxMult)`: Returns the theoretical maximum structure score for normalization.
- `DIR.zoneMaxRaw(_zones, _legStartTime, _mode, _volMaxMult)`: Returns the theoretical maximum zone score for normalization.

### Table rendering

- `DIR.dir_drawScoreTable(...)`: Draws the 8x3 score table in the top-right corner.
- `DIR.dir_drawLegendTable(_wCan1, _wCan2, _wCan3)`: Draws the 3x40 candle-pattern legend in the middle-right area.

## State model notes

- Broken swings credit the opposite direction in structure scoring.
- Broken zones credit the opposite direction in zone scoring.
- `Tested` zones are mainly visual. Their score remains the same as `New`.
- `Respected` zones carry the strongest non-terminal zone score.
- `Swept` means price wicked beyond the far side but closed back through the near side.
- `readyForRetest` prevents fresh zones from scoring a retest on the creation bar noise.
- In `PrioritizeBase` mode, older FVGs closer to the leg origin receive higher proximity weight.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Importing only `DIR` when calling candle-pattern typed helpers directly | Import `CPL` when you call helpers that expose `CPL.*` types |
| Passing raw volume instead of `volume / volumeSma` | Precompute a volume ratio and store that in `volRatio` |
| Mutating a swing or zone but not writing it back to the array | Reassign with `array.set(...)` after `dir_applySwingEvent(...)` or `dir_tickZone(...)` |
| Treating the library as a full indicator | Keep inputs, visuals, and orchestration in the host indicator |
| Expecting `DIR.computeCandleScores(...)` to calculate ATR for you | Pass an ATR value that your indicator already computed |
| Forgetting that `dir_drawScoreTable(...)` and `dir_drawLegendTable(...)` only render on `barstate.islast` | Call them every bar, but expect visible output only on the last bar |



