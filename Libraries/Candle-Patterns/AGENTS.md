# AGENTS.md - Libraries/Candle-Patterns

Pine Script v6 candle-analysis library for classifying single candles and
detecting two-candle and three-candle patterns. The library owns candle
classification, pattern detection, enum naming helpers, and sentiment scoring.
The consumer owns input declarations, average-size/ATR sourcing, bar ordering,
rendering, alerts, and any strategy logic built on top of the returned data.
Published as:

```pine
import OneCleverGuy/CandlePatternLibrary/4 as CPL
```

Dependency note: this library has no imported-library dependencies.

---

## Public API

```pine
CPL.analyzeCandle(
    float _open,
    float _high,
    float _low,
    float _close,
    float _avgSize,
    float _sizeThresholdPct,
    float _equivTolerance,
    float _bodyTolerance,
    int _positionThreshold) -> CPL.CandleData

CPL.scoreCandleSentiment(
    float _open,
    float _high,
    float _low,
    float _close,
    float _atr,
    float _atrMult,
    float _wickWeight) -> [float, float, float, float, float]

CPL.isEngulfingPattern(bool _bodyEngulfs, CPL.CandleDirection _c2Dir, CPL.CandleDirection _targetDir) -> bool
CPL.isInsideBarPattern(float _c2High, float _c1High, float _c2Low, float _c1Low) -> bool
CPL.isTweezerTopPattern(bool _notShortCandles, bool _equalHighs, bool _c1CloseEqualsC2Open, CPL.CandleDirection _c1Dir, CPL.CandleDirection _c2Dir, float _c1UpperWickPct, float _c2UpperWickPct, float _minWickPct) -> bool
CPL.isTweezerBottomPattern(bool _notShortCandles, bool _equalLows, bool _c1CloseEqualsC2Open, CPL.CandleDirection _c1Dir, CPL.CandleDirection _c2Dir, float _c1LowerWickPct, float _c2LowerWickPct, float _minWickPct) -> bool
CPL.isRailRoadPattern(CPL.CandlePattern _c1Pattern, CPL.CandlePattern _c2Pattern, CPL.CandleDirection _c1Dir, CPL.CandleDirection _c2Dir, CPL.CandleDirection _targetDir) -> bool

CPL.isThreeWhiteSoldiersPattern(CPL.CandleSize _c1Size, CPL.CandleSize _c2Size, CPL.CandleSize _c3Size, CPL.CandleDirection _c1Dir, CPL.CandleDirection _c2Dir, CPL.CandleDirection _c3Dir) -> bool
CPL.isThreeBlackCrowsPattern(CPL.CandleSize _c1Size, CPL.CandleSize _c2Size, CPL.CandleSize _c3Size, CPL.CandleDirection _c1Dir, CPL.CandleDirection _c2Dir, CPL.CandleDirection _c3Dir) -> bool
CPL.isMorningStarPattern(CPL.CandleSize _c1Size, CPL.CandleDirection _c1Dir, CPL.CandlePattern _c2Pattern, CPL.CandleSize _c3Size, CPL.CandleDirection _c3Dir) -> bool
CPL.isEveningStarPattern(CPL.CandleSize _c1Size, CPL.CandleDirection _c1Dir, CPL.CandlePattern _c2Pattern, CPL.CandleSize _c3Size, CPL.CandleDirection _c3Dir) -> bool
CPL.isBullishAbandonedBabyPattern(CPL.CandleSize _c1Size, CPL.CandleDirection _c1Dir, float _c1Low, CPL.CandlePattern _c2Pattern, float _c2High, CPL.CandleSize _c3Size, CPL.CandleDirection _c3Dir, float _c3Low) -> bool
CPL.isBearishAbandonedBabyPattern(CPL.CandleSize _c1Size, CPL.CandleDirection _c1Dir, float _c1High, CPL.CandlePattern _c2Pattern, float _c2Low, CPL.CandleSize _c3Size, CPL.CandleDirection _c3Dir, float _c3High) -> bool
CPL.isEngulfingSandwichPattern(float _c1High, float _c1Low, float _c2High, float _c2Low, float _c3High, float _c3Low) -> bool
CPL.isBullishFairValueGapPattern(float _c1High, float _c3Low, float _minGap) -> bool
CPL.isBearishFairValueGapPattern(float _c1Low, float _c3High, float _minGap) -> bool

CPL.analyzeTwoCandlePattern(CPL.CandleData _candle1, CPL.CandleData _candle2, float _equivTolerance, int _positionThreshold) -> CPL.TwoCandleData
CPL.analyzeThreeCandlePattern(CPL.CandleData _candle1, CPL.CandleData _candle2, CPL.CandleData _candle3, float _minGap) -> CPL.ThreeCandleData

CPL.getPatternName(CPL.CandlePattern _pattern) -> string
CPL.getTwoCandlePatternName(CPL.TwoCandlePattern _pattern) -> string
CPL.getThreeCandlePatternName(CPL.ThreeCandlePattern _pattern) -> string
CPL.getSizeName(CPL.CandleSize _size) -> string
CPL.getDirectionName(CPL.CandleDirection _direction) -> string
```

---

## Exported Enums

### `CandleSize`

| Value | Meaning |
|-------|---------|
| `Short` | Wick range is below the short threshold derived from `_avgSize` and `_sizeThresholdPct`. |
| `Normal` | Wick range is between the short and long thresholds. |
| `Long` | Wick range is above the long threshold derived from `_avgSize` and `_sizeThresholdPct`. |

### `CandleDirection`

| Value | Meaning |
|-------|---------|
| `Bearish` | `_close < _open - _equivTolerance`. |
| `Neutral` | `_close` is within `_equivTolerance` of `_open`. |
| `Bullish` | `_close > _open + _equivTolerance`. |

### `CandlePattern`

| Value | Meaning |
|-------|---------|
| `Unknown` | Initial/default sentinel before single-candle classification. |
| `RegularBullish` | Bullish candle with no higher-priority single-candle match. |
| `RegularBearish` | Bearish candle with no higher-priority single-candle match. |
| `BullishMarubozu` | Open equals low and close equals high within tolerance. |
| `BearishMarubozu` | Open equals high and close equals low within tolerance. |
| `Hammer` | Small-body normal/long candle with the body in the upper section. |
| `ShootingStar` | Small-body normal/long candle with the body in the lower section. |
| `SpinningTop` | Small-body normal/long candle with the body in the middle section. |
| `Doji` | Equivalent-body short candle, or final neutral fallback. |
| `LongLeggedDoji` | Equivalent-body normal/long candle with the body in the middle section. |
| `CrossDoji` | Equivalent-body normal/long candle with the body in the upper section. |
| `DragonflyDoji` | Equivalent body positioned at the high. |
| `InvertedCrossDoji` | Equivalent-body normal/long candle with the body in the lower section. |
| `GravestoneDoji` | Equivalent body positioned at the low. |
| `FourPriceDoji` | High equals low and open equals close within tolerance. |

### `TwoCandlePattern`

| Value | Meaning |
|-------|---------|
| `None` | No two-candle pattern detected. |
| `BullishEngulfingWeak` | Bullish engulfing where candle 2 closes inside candle 1's full range. |
| `BullishEngulfingStrong` | Bullish engulfing where candle 2 closes outside candle 1's full range. |
| `BearishEngulfingWeak` | Bearish engulfing where candle 2 closes inside candle 1's full range. |
| `BearishEngulfingStrong` | Bearish engulfing where candle 2 closes outside candle 1's full range. |
| `InsideBar` | Candle 2 high is below candle 1 high and candle 2 low is above candle 1 low. |
| `TweezerTop` | Equal highs plus bullish-then-bearish reversal shape with sufficient upper wicks. |
| `TweezerBottom` | Equal lows plus bearish-then-bullish reversal shape with sufficient lower wicks. |
| `BullishRailRoad` | Opposite-direction marubozu pair where candle 2 is bullish. |
| `BearishRailRoad` | Opposite-direction marubozu pair where candle 2 is bearish. |

### `ThreeCandlePattern`

| Value | Meaning |
|-------|---------|
| `None` | No three-candle pattern detected. |
| `ThreeWhiteSoldiers` | Three normal/long bullish candles. |
| `ThreeBlackCrows` | Three normal/long bearish candles. |
| `ThreeWhiteSoldiersWithBullishFVG` | Three White Soldiers plus a bullish fair value gap. |
| `ThreeWhiteSoldiersWithBearishFVG` | Three White Soldiers plus a bearish fair value gap. |
| `ThreeBlackCrowsWithBullishFVG` | Three Black Crows plus a bullish fair value gap. |
| `ThreeBlackCrowsWithBearishFVG` | Three Black Crows plus a bearish fair value gap. |
| `MorningStar` | Bearish candle, Doji-family middle candle, bullish candle. |
| `EveningStar` | Bullish candle, Doji-family middle candle, bearish candle. |
| `BullishAbandonedBaby` | Morning Star with the middle Doji fully gapped below both neighbors. |
| `BearishAbandonedBaby` | Evening Star with the middle Doji fully gapped above both neighbors. |
| `EngulfingSandwich` | Candle 2 range engulfs candle 1 range and candle 3 range sits inside candle 2 range. |
| `BullishFairValueGap` | Candle 3 low minus candle 1 high is at least `_minGap`. |
| `BearishFairValueGap` | Candle 1 low minus candle 3 high is at least `_minGap`. |

Doji-family patterns used by star and abandoned-baby helpers are:
`Doji`, `LongLeggedDoji`, `CrossDoji`, `DragonflyDoji`,
`InvertedCrossDoji`, `GravestoneDoji`, and `FourPriceDoji`.

---

## Exported Types

### `CandleData`

Returned by `analyzeCandle()` and required by the multi-candle analyzers.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `wickRange` | `float` | computed | Full candle high/low range. |
| `bodyRange` | `float` | computed | Absolute body size. |
| `bodyHigh` | `float` | computed | Higher of open and close. |
| `bodyLow` | `float` | computed | Lower of open and close. |
| `size` | `CandleSize` | computed | Size classification from `determineCandleSize(...)`. |
| `direction` | `CandleDirection` | computed | Direction classification from `determineCandleDirection(...)`. |
| `pattern` | `CandlePattern` | computed | Final single-candle classification. |
| `openPrice` | `float` | input | Original open price. |
| `highPrice` | `float` | input | Original high price. |
| `lowPrice` | `float` | input | Original low price. |
| `closePrice` | `float` | input | Original close price. |

Manual constructor order:

```pine
CPL.CandleData.new(
    wickRange, bodyRange, bodyHigh, bodyLow, size, direction, pattern,
    openPrice, highPrice, lowPrice, closePrice)
```

### `TwoCandleData`

Returned by `analyzeTwoCandlePattern()`.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `pattern` | `TwoCandlePattern` | computed | Final two-candle classification or `None`. |
| `candle1` | `CandleData` | input | Older candle passed to the analyzer. |
| `candle2` | `CandleData` | input | Newer candle passed to the analyzer. |

### `ThreeCandleData`

Returned by `analyzeThreeCandlePattern()`.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `pattern` | `ThreeCandlePattern` | computed | Final three-candle classification or `None`. |
| `candle1` | `CandleData` | input | Oldest candle passed to the analyzer. |
| `candle2` | `CandleData` | input | Middle candle passed to the analyzer. |
| `candle3` | `CandleData` | input | Newest candle passed to the analyzer. |

---

## Function Reference

### Core analyzers

#### `analyzeCandle`

```pine
CPL.analyzeCandle(
    float _open,
    float _high,
    float _low,
    float _close,
    float _avgSize,
    float _sizeThresholdPct,
    float _equivTolerance,
    float _bodyTolerance,
    int _positionThreshold) -> CPL.CandleData
```

| Argument | Type | Required | Meaning |
|----------|------|----------|---------|
| `_open` | `float` | yes | Candle open price. |
| `_high` | `float` | yes | Candle high price. |
| `_low` | `float` | yes | Candle low price. |
| `_close` | `float` | yes | Candle close price. |
| `_avgSize` | `float` | yes | Baseline wick range used for `CandleSize` thresholds. |
| `_sizeThresholdPct` | `float` | yes | Percent above and below `_avgSize` used for long/short cutoffs. |
| `_equivTolerance` | `float` | yes | Absolute tolerance for equal-price checks and neutral direction. |
| `_bodyTolerance` | `float` | yes | Absolute tolerance for small-body classification. |
| `_positionThreshold` | `int` | yes | Percent used to split wick space into upper/middle/lower sections. |

Returns a fully populated `CandleData`.

Behavior notes:

- Computes `wickRange`, `bodyRange`, `bodyHigh`, and `bodyLow`.
- Uses `_avgSize * 0.001` as a safe fallback when `wickRange <= 0`.
- Uses `_equivTolerance` for Doji/Marubozu/equality logic and
  `_bodyTolerance` for small-body logic.
- Applies this priority order:

```text
FourPriceDoji
DragonflyDoji
GravestoneDoji
CrossDoji
InvertedCrossDoji
LongLeggedDoji
Doji
BullishMarubozu
BearishMarubozu
Hammer / ShootingStar / SpinningTop
RegularBullish
RegularBearish
fallback Doji
```

#### `scoreCandleSentiment`

```pine
CPL.scoreCandleSentiment(
    float _open,
    float _high,
    float _low,
    float _close,
    float _atr,
    float _atrMult,
    float _wickWeight) -> [float, float, float, float, float]
```

| Argument | Type | Required | Meaning |
|----------|------|----------|---------|
| `_open` | `float` | yes | Candle open price. |
| `_high` | `float` | yes | Candle high price. |
| `_low` | `float` | yes | Candle low price. |
| `_close` | `float` | yes | Candle close price. |
| `_atr` | `float` | yes | ATR yardstick for range normalization. |
| `_atrMult` | `float` | yes | Range/ATR multiple required for full power. |
| `_wickWeight` | `float` | yes | Wick contribution added to body-driven shape. |

Returns a five-float tuple in this exact order:

| Position | Name | Meaning |
|----------|------|---------|
| `1` | `score` | `shape * power * 100.0`. |
| `2` | `shape` | `bodyRatio + _wickWeight * wickBias`. |
| `3` | `power` | Range strength capped at `1.0` when ATR and range are valid. |
| `4` | `bodyRatio` | Signed body share of total range. |
| `5` | `wickBias` | Lower wick minus upper wick, normalized by total range. |

If range is invalid, `bodyRatio` and `wickBias` return `0.0`. If ATR is `na`
or `<= 0`, `power` and `score` return `0.0`.

#### `analyzeTwoCandlePattern`

```pine
CPL.analyzeTwoCandlePattern(
    CPL.CandleData _candle1,
    CPL.CandleData _candle2,
    float _equivTolerance,
    int _positionThreshold) -> CPL.TwoCandleData
```

| Argument | Type | Required | Meaning |
|----------|------|----------|---------|
| `_candle1` | `CandleData` | yes | Older candle. |
| `_candle2` | `CandleData` | yes | Newer candle. |
| `_equivTolerance` | `float` | yes | Absolute tolerance for equal highs, lows, and close/open joins. |
| `_positionThreshold` | `int` | yes | Used to derive the minimum wick percentage for tweezer checks. |

Returns `TwoCandleData.new(pattern, _candle1, _candle2)`.

Priority order:

```text
TweezerTop
TweezerBottom
BullishRailRoad
BearishRailRoad
BullishEngulfingWeak / BullishEngulfingStrong
BearishEngulfingWeak / BearishEngulfingStrong
InsideBar
fallback None
```

Weak vs strong engulfing is determined by whether candle 2 closes inside or
outside candle 1's full high/low range.

#### `analyzeThreeCandlePattern`

```pine
CPL.analyzeThreeCandlePattern(
    CPL.CandleData _candle1,
    CPL.CandleData _candle2,
    CPL.CandleData _candle3,
    float _minGap) -> CPL.ThreeCandleData
```

| Argument | Type | Required | Meaning |
|----------|------|----------|---------|
| `_candle1` | `CandleData` | yes | Oldest candle. |
| `_candle2` | `CandleData` | yes | Middle candle. |
| `_candle3` | `CandleData` | yes | Newest candle. |
| `_minGap` | `float` | yes | Minimum absolute gap required for fair value gap checks. |

Returns `ThreeCandleData.new(pattern, _candle1, _candle2, _candle3)`.

Priority order:

```text
ThreeWhiteSoldiersWithBullishFVG
ThreeWhiteSoldiersWithBearishFVG
ThreeBlackCrowsWithBullishFVG
ThreeBlackCrowsWithBearishFVG
ThreeWhiteSoldiers
ThreeBlackCrows
BullishAbandonedBaby
BearishAbandonedBaby
MorningStar
EveningStar
EngulfingSandwich
BullishFairValueGap
BearishFairValueGap
fallback None
```

### Exported helper predicates

#### Two-candle helpers

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `isEngulfingPattern` | `_bodyEngulfs`, `_c2Dir`, `_targetDir` | `bool` | True when body engulfing is already confirmed and candle 2 matches the requested direction. |
| `isInsideBarPattern` | `_c2High`, `_c1High`, `_c2Low`, `_c1Low` | `bool` | True when candle 2 is fully inside candle 1's range. |
| `isTweezerTopPattern` | `_notShortCandles`, `_equalHighs`, `_c1CloseEqualsC2Open`, `_c1Dir`, `_c2Dir`, `_c1UpperWickPct`, `_c2UpperWickPct`, `_minWickPct` | `bool` | True for the library's bullish-then-bearish equal-high reversal definition. |
| `isTweezerBottomPattern` | `_notShortCandles`, `_equalLows`, `_c1CloseEqualsC2Open`, `_c1Dir`, `_c2Dir`, `_c1LowerWickPct`, `_c2LowerWickPct`, `_minWickPct` | `bool` | True for the library's bearish-then-bullish equal-low reversal definition. |
| `isRailRoadPattern` | `_c1Pattern`, `_c2Pattern`, `_c1Dir`, `_c2Dir`, `_targetDir` | `bool` | True when both candles are marubozu, directions oppose, and candle 2 matches the requested direction. |

#### Three-candle helpers

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `isThreeWhiteSoldiersPattern` | `_c1Size`, `_c2Size`, `_c3Size`, `_c1Dir`, `_c2Dir`, `_c3Dir` | `bool` | True when all three candles are normal/long and bullish. |
| `isThreeBlackCrowsPattern` | `_c1Size`, `_c2Size`, `_c3Size`, `_c1Dir`, `_c2Dir`, `_c3Dir` | `bool` | True when all three candles are normal/long and bearish. |
| `isMorningStarPattern` | `_c1Size`, `_c1Dir`, `_c2Pattern`, `_c3Size`, `_c3Dir` | `bool` | True for the library's bearish-Doji-bullish star definition. |
| `isEveningStarPattern` | `_c1Size`, `_c1Dir`, `_c2Pattern`, `_c3Size`, `_c3Dir` | `bool` | True for the library's bullish-Doji-bearish star definition. |
| `isBullishAbandonedBabyPattern` | `_c1Size`, `_c1Dir`, `_c1Low`, `_c2Pattern`, `_c2High`, `_c3Size`, `_c3Dir`, `_c3Low` | `bool` | True when the Morning Star shape includes a fully isolated middle Doji below both neighbors. |
| `isBearishAbandonedBabyPattern` | `_c1Size`, `_c1Dir`, `_c1High`, `_c2Pattern`, `_c2Low`, `_c3Size`, `_c3Dir`, `_c3High` | `bool` | True when the Evening Star shape includes a fully isolated middle Doji above both neighbors. |
| `isEngulfingSandwichPattern` | `_c1High`, `_c1Low`, `_c2High`, `_c2Low`, `_c3High`, `_c3Low` | `bool` | True when candle 2 engulfs candle 1 and candle 3 sits inside candle 2. |
| `isBullishFairValueGapPattern` | `_c1High`, `_c3Low`, `_minGap` | `bool` | True when the candle 3 low is at least `_minGap` above candle 1 high. |
| `isBearishFairValueGapPattern` | `_c1Low`, `_c3High`, `_minGap` | `bool` | True when the candle 1 low is at least `_minGap` above candle 3 high. |

These helpers are predicate primitives. They do not apply the full analyzer
priority stacks or weak/strong engulfing classification on their own.

### Name helpers

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `getPatternName` | `_pattern` | `string` | Converts `CandlePattern` values into display labels. |
| `getTwoCandlePatternName` | `_pattern` | `string` | Converts `TwoCandlePattern` values into display labels. |
| `getThreeCandlePatternName` | `_pattern` | `string` | Converts `ThreeCandlePattern` values into display labels. |
| `getSizeName` | `_size` | `string` | Converts `CandleSize` values into display labels. |
| `getDirectionName` | `_direction` | `string` | Converts `CandleDirection` values into display labels. |

Use these for labels, tables, logs, or alerts. Compare enums directly in logic.

---

## Function Hierarchy

```text
CPL
|
+-- Core classification
|   +-- analyzeCandle(...) -> CandleData
|   |   +-- determineCandleSize(...) [private]
|   |   +-- determineCandleDirection(...) [private]
|   |   +-- isEquivalent(...) [private]
|   |   +-- is4PriceDoji(...) [private]
|   |   +-- isDragonflyDoji(...) [private]
|   |   +-- isGravestoneDoji(...) [private]
|   |   +-- isCrossDoji(...) [private]
|   |   +-- isInvertedCrossDoji(...) [private]
|   |   +-- isLongLeggedDoji(...) [private]
|   |   +-- isRegularDoji(...) [private]
|   |   +-- isBullishMarubozu(...) [private]
|   |   +-- isBearishMarubozu(...) [private]
|   |   +-- isSmallBodyPattern(...) [private]
|   |   +-- getSmallBodyPatternType(...) [private]
|   +-- scoreCandleSentiment(...) -> [score, shape, power, bodyRatio, wickBias]
|
+-- Two-candle primitives
|   +-- isEngulfingPattern(...) -> bool
|   +-- isInsideBarPattern(...) -> bool
|   +-- isTweezerTopPattern(...) -> bool
|   +-- isTweezerBottomPattern(...) -> bool
|   +-- isRailRoadPattern(...) -> bool
|   +-- analyzeTwoCandlePattern(...) -> TwoCandleData
|
+-- Three-candle primitives
|   +-- isThreeWhiteSoldiersPattern(...) -> bool
|   +-- isThreeBlackCrowsPattern(...) -> bool
|   +-- isMorningStarPattern(...) -> bool
|   +-- isEveningStarPattern(...) -> bool
|   +-- isBullishAbandonedBabyPattern(...) -> bool
|   +-- isBearishAbandonedBabyPattern(...) -> bool
|   +-- isEngulfingSandwichPattern(...) -> bool
|   +-- isBullishFairValueGapPattern(...) -> bool
|   +-- isBearishFairValueGapPattern(...) -> bool
|   +-- analyzeThreeCandlePattern(...) -> ThreeCandleData
|
+-- Display helpers
    +-- getPatternName(...) -> string
    +-- getTwoCandlePatternName(...) -> string
    +-- getThreeCandlePatternName(...) -> string
    +-- getSizeName(...) -> string
    +-- getDirectionName(...) -> string
```

---

## Standard Integration Pattern

```pine
import OneCleverGuy/CandlePatternLibrary/4 as CPL

float avgSize = ta.atr(14)
float equivTolerance = 10.0
float bodyTolerance = 50.0
int positionThreshold = 85
float minGap = 10.0

CPL.CandleData newest = CPL.analyzeCandle(
    open, high, low, close,
    avgSize, 50.0, equivTolerance, bodyTolerance, positionThreshold)

CPL.CandleData middle = CPL.analyzeCandle(
    open[1], high[1], low[1], close[1],
    avgSize, 50.0, equivTolerance, bodyTolerance, positionThreshold)

CPL.CandleData oldest = CPL.analyzeCandle(
    open[2], high[2], low[2], close[2],
    avgSize, 50.0, equivTolerance, bodyTolerance, positionThreshold)

CPL.TwoCandleData two = CPL.analyzeTwoCandlePattern(
    middle, newest, equivTolerance, positionThreshold)

CPL.ThreeCandleData three = CPL.analyzeThreeCandlePattern(
    oldest, middle, newest, minGap)

bool hammer = newest.pattern == CPL.CandlePattern.Hammer
bool morningStar = three.pattern == CPL.ThreeCandlePattern.MorningStar

[score, shape, power, bodyRatio, wickBias] =
    CPL.scoreCandleSentiment(open, high, low, close, ta.atr(14), 2.0, 0.5)
```

Chronological order is strict:

- Two-candle analyzers expect older then newer.
- Three-candle analyzers expect oldest, middle, newest.

---

## Execution Ownership

| Ownership area | Library owns | Consumer owns |
|----------------|--------------|---------------|
| Candle classification | Size, direction, Doji/Marubozu/small-body rules | Choosing thresholds and sourcing inputs |
| Pattern detection | Two-candle and three-candle predicate logic plus analyzer priority | Deciding which patterns matter for entries, exits, or rendering |
| State | Stateless calculations from current inputs | Any persistent arrays, labels, tables, alerts, or strategy state |
| Naming | Enum-to-string conversion helpers | UI wording beyond the provided helper labels |
| Sentiment | Candle score computation | How the score is filtered, plotted, or traded |

This library does not call `alert()`, does not create chart objects, and does
not manage persistent runtime containers.

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Analyze single candles first | `analyzeTwoCandlePattern()` and `analyzeThreeCandlePattern()` assume valid `CandleData` inputs with correct `size`, `direction`, `pattern`, and OHLC fields already populated. |
| Preserve chronological order | Reversing candle order changes engulfing, star, railroad, and fair-value-gap semantics. |
| Keep thresholds consistent across a sequence | Mixed `_avgSize`, `_equivTolerance`, `_bodyTolerance`, or `_positionThreshold` settings can make multi-candle results inconsistent. |
| Tolerances are absolute price units | `_equivTolerance`, `_bodyTolerance`, and `_minGap` are not ticks unless the consumer converts ticks to price first. |
| Name helpers are cosmetic | Use enums in logic and helper strings only for display. |
| FVG helpers take raw floats, not `CandleData` | `isBullishFairValueGapPattern()` and `isBearishFairValueGapPattern()` expect strict float argument order. |
| `scoreCandleSentiment()` is not fully clamped | `power` is capped at `1.0`, but `shape` can exceed the usual range if `_wickWeight` is pushed beyond normal bounds. |
| `CandleData` constructor order is strict | If manually constructing test objects, field order must match the exported type declaration exactly. |
| Helper predicates are lower-level than analyzers | Calling helpers directly bypasses analyzer-level priority ordering and weak/strong engulfing classification. |
| Zero-range candles use a safe wick-range fallback | `analyzeCandle()` substitutes `_avgSize * 0.001` for section placement math when `wickRange <= 0`. |
