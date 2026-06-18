# Candle POI Direction - Indicator Guide

`Indicators/Candle-POI-Direction/Candle-POI-Direction.ps` is the public
indicator layer for the Direction scoring system. It imports
`CandlePOIDirectionLibrary`, maintains public pivot state and chart drawings,
and converts the library's raw category scores into final bullish and bearish
percentages.

## Purpose

The indicator combines four signal families:

1. Structure swings from a local public pivot adapter.
2. Order blocks resolved from confirmed pivots.
3. Fair value gaps detected from 3-candle windows.
4. Candle-pattern conviction scored by `DIR`.

Bullish and bearish scores are normalized independently, so both sides can be
strong or weak at the same time. The output is directional pressure, not a
zero-sum oscillator.

## Dependency

```pine
import OneCleverGuy/CandlePOIDirectionLibrary/1 as DIR
```

The indicator intentionally has no `MarketStructureLibrary` import. It uses
`ta.pivothigh()` and `ta.pivotlow()` to build `PublicPivot` records, then maps
them into `DIR.dir_RelClass` and `DIR.dir_Direction`.

## Inputs

| Group | Inputs |
|-------|--------|
| Shared | `Draw Extension (bars)` |
| ATR | `ATR Length`, `Equality Mult`, `Body Mult` |
| Structure | `Show Swing Lines`, `Pivot Left / Right`, `ATR Confirmation`, `Pivot ATR Mult`, `Equality Mult`, swing colors, `Line Width` |
| Order Blocks | `Show Order Blocks`, bull/bear colors |
| Fair Value Gaps | `Show FVGs`, `Minimum Gap (ATRx)`, bull/bear colors, `Leg Weight Mode` |
| Scoring Weights | `Structure`, `Order Blocks`, `Fair Value Gaps`, `1-Candle Patterns`, `2-Candle Patterns`, `3-Candle Patterns` |
| Volume Confluence | `Volume SMA Length`, `Max Multiplier`, `Min Multiplier` |
| Display | `Show Scoring Table`, `Show Pattern Log` |
| Debug | `Enable Debug` |

## Local UDTs

### `SwingVisual`

Pairs one structure swing with its local chart objects:

- `swingLine`
- `swingLabel`
- `obBox`
- `obLabel`
- `pivotIdx`
- `isHigh`
- `touched`

This array stays parallel with `swings` and `obZones`.

### `FVGVisual`

Stores chart objects for one fair value gap:

- `fvgBox`
- `fvgLabel`

This array stays parallel with `fvgZones`.

### `PublicPivot` and `PublicEvent`

`PublicPivot` stores confirmed pivot side, price, bar time, and
`DIR.dir_RelClass`. `PublicEvent` carries newly confirmed pivot indexes through
the current bar's materialization phase.

## Runtime State

The indicator persists:

- `pivots`: confirmed public pivots.
- `msDir`: current derived `DIR.dir_Direction`.
- `cacheOpen`, `cacheClose`, `cacheVolRatio`, `cachePrevTime`,
  `cacheNextTime`: time-keyed maps for delayed pivot reconstruction.
- `swingVisuals`, `swings`, `obZones`: parallel structure/OB arrays.
- `fvgVisuals`, `fvgZones`: parallel FVG arrays.

## Execution Flow

### 1. Cache update

Each bar computes `atrValue`, `volRatioCur`, and stores OHLC plus previous/next
time links in maps keyed by `time`.

### 2. Structure engine

The indicator confirms pivots with `ta.pivothigh()` and `ta.pivotlow()`.
Confirmed pivots are classified against the newest same-side pivot as
`HH`, `HL`, `LH`, `LL`, `EH`, `EL`, or `XX`.

`legStartTime` is derived from the older of the current external high/low pair.
That timestamp is the scoring window origin and pruning boundary.

### 3. Pivot materialization

When a pivot confirms:

- create a `DIR.dir_Swing`
- draw the swing line and pivot label when enabled
- reconstruct pivot-neighbor OHLC from the time cache
- resolve an OB with `DIR.dir_resolveOBLow(...)` or
  `DIR.dir_resolveOBHigh(...)`
- create a `DIR.dir_Zone` for valid OBs
- push matching entries into `swingVisuals`, `swings`, and `obZones`

### 4. Swing tick

Every active swing is checked for wick-through sweeps or close-through breaks.
State transitions use `DIR.dir_applySwingEvent(...)`. Close-through breaks also
credit the nearest eligible opposing swing, OB, and FVG.

### 5. Zone tick

Each active OB and FVG is advanced with `DIR.dir_tickZone(...)`, then restyled
locally according to `New`, `Tested`, `Respected`, `Swept`, or `Broken` state.

### 6. Active POI extension

The newest unbroken swing high, swing low, bullish OB, bearish OB, bullish FVG,
and bearish FVG are extended to the configured future time.

### 7. FVG scan

On confirmed bars with at least three candles available, the indicator detects
FVGs with `DIR.dir_detectFVG(...)`, creates a `DIR.dir_Zone`, draws the box and
label, and appends matching visual/record entries.

### 8. Leg pruning

When `legStartTime` advances, records with `anchorTime < legStartTime` are
removed. Chart objects are deleted before the matching records are shifted.

### 9. Scoring engine

The indicator computes:

- candle scores with `DIR.computeCandleScores(...)`
- structure scores with `DIR.computeStructureScore(...)`
- OB scores with `DIR.computeZoneScore(...)`
- FVG scores with `DIR.computeZoneScore(...)`

It then normalizes category totals, applies current-bar volume to candle
columns, computes weighted bullish and bearish totals, and caps each final
percentage at `99`.

### 10. Tables

Optional outputs are delegated to the library:

- `DIR.dir_drawScoreTable(...)`
- `DIR.dir_drawLegendTable(...)`

## Important Implementation Details

- OBs start with `readyForRetest=false` because price is still at the pivot.
- FVGs start with `readyForRetest=true` because the gap implies departure.
- `volRatioCur` is `volume / ta.sma(volume, i_volLen)` with a fallback of `1.0`.
- The indicator relies on `time`-keyed caches to rebuild pivot-neighbor candles
  after confirmation delay.
- `obZones` may contain `na` placeholders when no OB resolves; this preserves
  parallel-array alignment.

## Safe Edit Points

| Area | Edit point |
|------|------------|
| Input UX | `Inputs` region |
| Swing / zone styling | Local helper functions in `State Styling` |
| Pivot logic | `Structure Engine` and `Pivot Events` |
| Sweep / break behavior | `Swing Tick` |
| Zone behavior | `Zone Tick` |
| FVG creation rules | `FVG Scan` |
| Score composition | `Scoring Engine` |
| Display toggles | `Scoring Table` and `Legend Table` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reintroducing `MarketStructureLibrary` | Keep the public pivot adapter local and pass only `DIR` enums into the library. |
| Breaking parallel-array alignment | When pushing, shifting, or mutating one side, update the matching arrays in the same block. |
| Using `bar_index` where the script expects time coordinates | Keep drawings on `xloc.bar_time`. |
| Forgetting pivots confirm after delay | Use cache maps to reconstruct pivot-neighbor OHLC data. |
| Pruning only visuals or only records | Delete objects and shift matching arrays together. |
| Treating the final score as zero-sum | Bull and bear totals are independent percentages. |
