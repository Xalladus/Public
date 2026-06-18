# AGENTS.md - Libraries/Candle-POI-Direction-Library

Pine Script v6 public scoring library for candle patterns plus points of
interest. The library owns exported UDTs, state transitions, raw score math,
normalization, and optional score/legend tables. Host indicators own inputs,
pivot detection, drawing objects, array pruning, alerts, and final weighting.
Published as:

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR
```

Dependency note: this library imports `OneCleverGuy/CandlePatternLibrary/3 as
CPL`. It must not import `MarketStructureLibrary`; consumers translate their
own structure source into `DIR.dir_RelClass` and `DIR.dir_Direction`.

---

## Public API

```pine
DIR.dir_resolveOBLow(float _lowPrice, float _pivotOpen, float _pivotClose, float _prevOpen, float _prevClose, float _nextOpen, float _nextClose, int _prevTime) -> DIR.dir_OrderBlock
DIR.dir_resolveOBHigh(float _highPrice, float _pivotOpen, float _pivotClose, float _prevOpen, float _prevClose, float _nextOpen, float _nextClose, int _prevTime) -> DIR.dir_OrderBlock
DIR.dir_detectFVG(float _c1High, float _c1Low, float _c3High, float _c3Low, float _minGap, int _anchorTime) -> DIR.dir_FairValueGap

DIR.dir_newSwing(DIR.dir_RelClass _relClass, bool _isHigh, float _price, int _anchorTime, float _volRatio) -> DIR.dir_Swing
DIR.dir_applySwingEvent(DIR.dir_Swing _swing, bool _isBreak, bool _isSweep) -> DIR.dir_Swing
DIR.dir_findCreditSwingIdx(DIR.dir_Swing[] _swings, bool _brokenWasHigh) -> int
DIR.dir_swingStateName(DIR.dir_SwingState _state) -> string

DIR.dir_newZone(float _top, float _bottom, int _anchorTime, bool _isBullish, int _creationBar, bool _readyForRetest, float _volRatio) -> DIR.dir_Zone
DIR.dir_tickZone(DIR.dir_Zone _zone, float _high, float _low, float _close, int _barIdx) -> DIR.dir_Zone
DIR.dir_findCreditZoneIdx(DIR.dir_Zone[] _zones, bool _brokenWasHigh) -> int
DIR.dir_zoneStateName(DIR.dir_ZoneState _state) -> string

DIR.volClamp(float _ratio, float _minMult, float _maxMult) -> float
DIR.relClassMultiplier(DIR.dir_RelClass _relClass) -> float
DIR.dirAlignmentMultiplier(DIR.dir_Direction _dir, bool _isBullishSignal) -> float
DIR.swingStateScore(DIR.dir_SwingState _state) -> float
DIR.zoneStateScore(DIR.dir_ZoneState _state) -> float
DIR.computeSeekStructureBias(DIR.dir_Swing[] _swings, DIR.dir_Zone[] _zones, int _legStartTime, DIR.dir_Direction _dir) -> [float, float]
DIR.proximityWeight(float _posInLeg, DIR.dir_FvgWeightMode _fvgMode) -> float

DIR.dir_singlePatternMeta(CPL.CandleData _cd) -> DIR.dir_PatternMeta
DIR.dir_twoPatternMeta(CPL.TwoCandlePattern _pat) -> DIR.dir_PatternMeta
DIR.dir_threePatternMeta(CPL.ThreeCandlePattern _pat) -> DIR.dir_PatternMeta
DIR.dir_singlePatternRows() -> DIR.dir_PatternMeta[]
DIR.dir_twoPatternRows() -> DIR.dir_PatternMeta[]
DIR.dir_threePatternRows() -> DIR.dir_PatternMeta[]
DIR.computeCandleScores(float _atr, float _equivMult, float _bodyMult) -> DIR.CandleScores

DIR.computeStructureScore(DIR.dir_Swing[] _swings, int _legStartTime, DIR.dir_Direction _dir, float _volMinMult, float _volMaxMult) -> [float, float]
DIR.computeZoneScore(DIR.dir_Zone[] _zones, int _legStartTime, DIR.dir_Direction _dir, DIR.dir_FvgWeightMode _mode, float _volMinMult, float _volMaxMult) -> [float, float]
DIR.normalizeScore(float _rawScore, float _maxRaw) -> float
DIR.structureMaxRaw(DIR.dir_Swing[] _swings, int _legStartTime, float _volMaxMult) -> float
DIR.zoneMaxRaw(DIR.dir_Zone[] _zones, int _legStartTime, DIR.dir_FvgWeightMode _mode, float _volMaxMult) -> float

DIR.dir_drawScoreTable(float _bullStNrm, float _bearStNrm, float _bullObNrm, float _bearObNrm, float _bullFvgNrm, float _bearFvgNrm, DIR.CandleScores _cs, float _bullTotal, float _bearTotal, DIR.dir_Direction _dir) -> void
DIR.dir_drawLegendTable(int _wCan1, int _wCan2, int _wCan3) -> void
```

---

## Exported Enums

| Enum | Value | Meaning |
|------|-------|---------|
| `dir_FvgWeightMode` | `Even` | All zones receive equal proximity weight. |
| `dir_FvgWeightMode` | `PrioritizeBase` | Older zones nearer the leg origin receive more weight. |
| `dir_RelClass` | `HH`, `HL`, `LH`, `LL` | Relative high/low classifications used for structure weighting. |
| `dir_RelClass` | `EH`, `EL` | Equal high/equal low classifications. |
| `dir_RelClass` | `XX` | Unknown or neutral relative classification. |
| `dir_Direction` | `Unknown`, `Rising`, `Falling`, `Ranging` | Current leg or external direction context. |
| `dir_SwingState` | `New`, `Swept`, `Broken` | Swing lifecycle. `Broken` is terminal. |
| `dir_ZoneState` | `New`, `Tested`, `Respected`, `Swept`, `Broken` | Zone lifecycle. `Broken` is terminal. |

---

## Exported Types

| Type | Fields | Purpose |
|------|--------|---------|
| `dir_OrderBlock` | `valid`, `top`, `bottom`, `anchorTime`, `isBullish` | Raw OB geometry resolved from a pivot and neighboring candles. |
| `dir_FairValueGap` | `valid`, `top`, `bottom`, `anchorTime`, `isBullish` | Raw FVG geometry resolved from a 3-candle window. |
| `dir_Swing` | `relClass`, `isHigh`, `price`, `anchorTime`, `swingState`, `brokeOpposing`, `volRatio` | Active structure scoring record for one confirmed pivot. |
| `dir_Zone` | `top`, `bottom`, `anchorTime`, `isBullish`, `creationBar`, `zoneState`, `brokeOpposing`, `hasEntered`, `everWickedFar`, `resolved`, `readyForRetest`, `volRatio` | Shared lifecycle record for OBs and FVGs. |
| `dir_PatternMeta` | `fullName`, `abbr`, `weightText`, `bullWeight`, `bearWeight` | Shared source of truth for candle-pattern labels and scoring weights. |
| `CandleScores` | `bull1`, `bear1`, `bull2`, `bear2`, `bull3`, `bear3`, `abbr1`, `abbr2`, `abbr3` | Per-bar candle-pattern score bundle and table labels. |

Persist `dir_Swing[]` and `dir_Zone[]` arrays with `var` in the consuming
indicator. The library does not retain global state for those records.

---

## Function Reference

| Group | Functions | Returns | Meaning |
|-------|-----------|---------|---------|
| OB/FVG geometry | `dir_resolveOBLow`, `dir_resolveOBHigh`, `dir_detectFVG` | Raw `dir_OrderBlock` or `dir_FairValueGap` | Converts pivot/window OHLC into public-safe geometry records. |
| Swing lifecycle | `dir_newSwing`, `dir_applySwingEvent`, `dir_findCreditSwingIdx`, `dir_swingStateName` | Swing records, indexes, labels | Creates and advances swing state, and finds the newest opposite-side credit target after breaks. |
| Zone lifecycle | `dir_newZone`, `dir_tickZone`, `dir_findCreditZoneIdx`, `dir_zoneStateName` | Zone records, indexes, labels | Creates and advances OB/FVG state, including retests, respects, sweeps, and breaks. |
| Scoring primitives | `volClamp`, `relClassMultiplier`, `dirAlignmentMultiplier`, `swingStateScore`, `zoneStateScore`, `proximityWeight` | `float` | Converts state, volume, structure class, proximity, and direction into score multipliers. |
| Hidden continuation | `computeSeekStructureBias` | `[bullBias, bearBias]` | Credits continuation bias from respected/swept zones toward newer target swings. |
| Candle metadata | `dir_singlePatternMeta`, `dir_twoPatternMeta`, `dir_threePatternMeta`, `dir_singlePatternRows`, `dir_twoPatternRows`, `dir_threePatternRows`, `computeCandleScores` | `dir_PatternMeta`, arrays, `CandleScores` | Keeps full names, abbreviations, display weights, and bull/bear weights together. |
| Category scoring | `computeStructureScore`, `computeZoneScore`, `normalizeScore`, `structureMaxRaw`, `zoneMaxRaw` | Raw and normalized scores | Scores active leg records and provides matching theoretical maxima. |
| Tables | `dir_drawScoreTable`, `dir_drawLegendTable` | `void` | Optional last-bar table rendering owned by the library. |

Important side effects:

- `dir_applySwingEvent()` mutates the passed `dir_Swing` record and returns it.
- `dir_tickZone()` mutates the passed `dir_Zone` record and returns it.
- Table functions create/update `table` objects internally and only render on
  `barstate.islast`.

---

## Function Hierarchy

```text
DIR
|
+-- Geometry
|   +-- dir_resolveOBLow(...) -> dir_OrderBlock
|   +-- dir_resolveOBHigh(...) -> dir_OrderBlock
|   +-- dir_detectFVG(...) -> dir_FairValueGap
|
+-- Lifecycle
|   +-- dir_newSwing(...) -> dir_Swing
|   +-- dir_applySwingEvent(...) -> dir_Swing
|   +-- dir_findCreditSwingIdx(...) -> int
|   +-- dir_newZone(...) -> dir_Zone
|   +-- dir_tickZone(...) -> dir_Zone
|   +-- dir_findCreditZoneIdx(...) -> int
|
+-- Scoring
|   +-- volClamp(...) -> float
|   +-- relClassMultiplier(...) -> float
|   +-- dirAlignmentMultiplier(...) -> float
|   +-- swingStateScore(...) -> float
|   +-- zoneStateScore(...) -> float
|   +-- computeSeekStructureBias(...) -> [float, float]
|   +-- computeStructureScore(...) -> [float, float]
|   +-- computeZoneScore(...) -> [float, float]
|   +-- normalizeScore(...) -> float
|   +-- structureMaxRaw(...) -> float
|   +-- zoneMaxRaw(...) -> float
|
+-- Candle Patterns
|   +-- computeCandleScores(...) -> CandleScores
|   |   +-- CPL.analyzeCandle(...)
|   |   +-- CPL.analyzeTwoCandlePattern(...)
|   |   +-- CPL.analyzeThreeCandlePattern(...)
|   +-- dir_singlePatternMeta(...) -> dir_PatternMeta
|   +-- dir_twoPatternMeta(...) -> dir_PatternMeta
|   +-- dir_threePatternMeta(...) -> dir_PatternMeta
|   +-- dir_single/two/threePatternRows(...) -> dir_PatternMeta[]
|
+-- Tables
    +-- dir_drawScoreTable(...) -> void
    +-- dir_drawLegendTable(...) -> void
```

---

## Standard Integration Pattern

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR

var DIR.dir_Swing[] swings = array.new<DIR.dir_Swing>()
var DIR.dir_Zone[] obs = array.new<DIR.dir_Zone>()
var DIR.dir_Zone[] fvgs = array.new<DIR.dir_Zone>()

DIR.dir_Swing swing = DIR.dir_newSwing(relClass, isHigh, pivotPrice, pivotTime, volRatio)
array.push(swings, swing)

DIR.dir_OrderBlock ob = isHigh
    ? DIR.dir_resolveOBHigh(pivotPrice, pivotOpen, pivotClose, prevOpen, prevClose, nextOpen, nextClose, prevTime)
    : DIR.dir_resolveOBLow(pivotPrice, pivotOpen, pivotClose, prevOpen, prevClose, nextOpen, nextClose, prevTime)

if ob.valid
    array.push(obs, DIR.dir_newZone(ob.top, ob.bottom, ob.anchorTime, ob.isBullish, bar_index, false, volRatio))

for [i, z] in obs
    z := DIR.dir_tickZone(z, high, low, close, bar_index)
    array.set(obs, i, z)

DIR.CandleScores cs = DIR.computeCandleScores(atrValue, equivAtrMult, bodyAtrMult)
[bullStRaw, bearStRaw] = DIR.computeStructureScore(swings, legStartTime, dir, volMinMult, volMaxMult)
[bullObRaw, bearObRaw] = DIR.computeZoneScore(obs, legStartTime, dir, DIR.dir_FvgWeightMode.Even, volMinMult, volMaxMult)

float stMax = DIR.structureMaxRaw(swings, legStartTime, volMaxMult)
float obMax = DIR.zoneMaxRaw(obs, legStartTime, DIR.dir_FvgWeightMode.Even, volMaxMult)
float bullSt = DIR.normalizeScore(bullStRaw, stMax)
float bearSt = DIR.normalizeScore(bearStRaw, stMax)
float bullOb = DIR.normalizeScore(bullObRaw, obMax)
float bearOb = DIR.normalizeScore(bearObRaw, obMax)
```

---

## Execution Ownership

| Ownership area | Library owns | Consumer owns |
|----------------|--------------|---------------|
| Structure context | Public enums, structure score weighting | Pivot detection, rel-class assignment, leg origin selection |
| OB/FVG records | Raw geometry helpers and zone state machine | Drawing boxes/labels and deciding which records to retain |
| Candle analysis | CandlePatternLibrary calls, conviction mapping, abbreviations | ATR/tolerance inputs and final display choices |
| Score math | Raw category scoring and normalization helpers | Category weights, totals, alerts, and strategy decisions |
| Tables | Optional score/legend table rendering | Calling tables only when wanted and managing other chart objects |

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Keep the library public-safe | Do not add private imports or `MS.*` types to exported signatures. |
| Inputs stay outside | Libraries cannot expose `input.*`; the indicator must supply ATR, volume ratio, weights, and modes. |
| Preserve record arrays | Persist `dir_Swing[]` and `dir_Zone[]` with `var`; pruning belongs to the consumer. |
| Write back returned records when in doubt | `dir_applySwingEvent()` and `dir_tickZone()` return the updated record; using `array.set(...)` after mutation is the safest integration pattern. |
| Match maxima to score calls | Normalize structure with `structureMaxRaw()` and zones with `zoneMaxRaw()` using the same arrays, leg start, volume cap, and FVG mode. |
| `legStartTime` is a filter | Records before the active leg origin are ignored by score functions and should usually be pruned by the indicator. |
| OB and FVG zones share APIs | Use `DIR.dir_FvgWeightMode.Even` for OBs when you do not want proximity weighting to alter OB scores. |
| `computeCandleScores()` uses chart history | Call it from chart scope with valid ATR/tolerance inputs; it analyzes the current three-candle window. |
| Tables are optional | Table helpers render only on `barstate.islast`; they do not replace indicator-side drawings. |
