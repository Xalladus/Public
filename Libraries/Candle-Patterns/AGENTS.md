# AGENTS.md - Public/Libraries/Candle-Patterns

Pine Script v6 library for classifying candles and detecting single-candle,
two-candle, and three-candle patterns. This file is the AI-facing contract for
the library. Do not guess exported names, enum values, type fields, argument
order, or return fields; use the exact API below.

Published import:

```pine
import OneCleverGuy/CandlePatternLibrary/4 as CPL
```

Source of truth in this folder:

- `1CG-Candle-Pattern-Library.ps` - Pine v6 library source.
- `Candle-Pattern-Library-Docs.txt` - public-facing docs.
- `AGENTS.md` - AI usage contract.

## Required Call Order

1. Import the library as `CPL`.
2. Build `CPL.CandleData` with `CPL.analyzeCandle(...)` for every candle.
3. Pass `CandleData` objects to multi-candle analyzers in chronological order:
   older to newer for two candles, oldest to newest for three candles.
4. Compare enum outputs directly. Use name helpers only for labels, tables,
   logging, and alerts.

```pine
CPL.CandleData newest = CPL.analyzeCandle(open, high, low, close, avgSize, 50.0, 10.0, 50.0, 85)
CPL.CandleData middle = CPL.analyzeCandle(open[1], high[1], low[1], close[1], avgSize, 50.0, 10.0, 50.0, 85)
CPL.CandleData oldest = CPL.analyzeCandle(open[2], high[2], low[2], close[2], avgSize, 50.0, 10.0, 50.0, 85)

CPL.TwoCandleData two = CPL.analyzeTwoCandlePattern(middle, newest, 10.0, 85)
CPL.ThreeCandleData three = CPL.analyzeThreeCandlePattern(oldest, middle, newest, 10.0)

bool hammer = newest.pattern == CPL.CandlePattern.Hammer
bool morningStar = three.pattern == CPL.ThreeCandlePattern.MorningStar

[sentScore, sentShape, sentPower, sentBodyRatio, sentWickBias] =
    CPL.scoreCandleSentiment(open, high, low, close, ta.atr(14), 2.0, 0.5)
```

## Exported Enums

Use enum values as `CPL.EnumName.Value`.

### `CandleSize`

| Value | Meaning |
|---|---|
| `Short` | Wick range is below the short threshold. |
| `Normal` | Wick range is between short and long thresholds. |
| `Long` | Wick range is above the long threshold. |

Size thresholds used by `analyzeCandle()`:

```pine
longThreshold = _avgSize * (1.0 + _sizeThresholdPct / 100.0)
shortThreshold = _avgSize * (1.0 - _sizeThresholdPct / 100.0)
```

### `CandleDirection`

| Value | Meaning |
|---|---|
| `Bearish` | `_close < _open - _equivTolerance`. |
| `Neutral` | `_close` is within `_equivTolerance` of `_open`. |
| `Bullish` | `_close > _open + _equivTolerance`. |

### `CandlePattern`

| Value | Meaning |
|---|---|
| `Unknown` | Initial/default value before detection; can be used as a no-pattern sentinel. |
| `RegularBullish` | Direction is bullish and no higher-priority single-candle pattern matched. |
| `RegularBearish` | Direction is bearish and no higher-priority single-candle pattern matched. |
| `BullishMarubozu` | Open equals low and close equals high within `_equivTolerance`. |
| `BearishMarubozu` | Open equals high and close equals low within `_equivTolerance`. |
| `Hammer` | Normal/long small-body candle with body in the upper section. |
| `ShootingStar` | Normal/long small-body candle with body in the lower section. |
| `SpinningTop` | Normal/long small-body candle with body in the middle section. |
| `Doji` | Open and close are equivalent on a short candle, or neutral fallback. |
| `LongLeggedDoji` | Equivalent-body normal/long candle with body in the middle section. |
| `CrossDoji` | Equivalent-body normal/long candle with body in the upper section. |
| `DragonflyDoji` | Equivalent body at the high. |
| `InvertedCrossDoji` | Equivalent-body normal/long candle with body in the lower section. |
| `GravestoneDoji` | Equivalent body at the low. |
| `FourPriceDoji` | High equals low and open equals close within `_equivTolerance`. |

### `TwoCandlePattern`

| Value | Meaning |
|---|---|
| `None` | No two-candle pattern detected. |
| `BullishEngulfingWeak` | Second candle body engulfs first body, second direction is bullish, and second close remains inside first candle high/low range. |
| `BullishEngulfingStrong` | Second candle body engulfs first body, second direction is bullish, and second close is outside first candle high/low range. |
| `BearishEngulfingWeak` | Second candle body engulfs first body, second direction is bearish, and second close remains inside first candle high/low range. |
| `BearishEngulfingStrong` | Second candle body engulfs first body, second direction is bearish, and second close is outside first candle high/low range. |
| `InsideBar` | Second candle high is below first high and second low is above first low. |
| `TweezerTop` | Non-short bullish then bearish candles with equal highs, joined close/open, and sufficient upper wicks. |
| `TweezerBottom` | Non-short bearish then bullish candles with equal lows, joined close/open, and sufficient lower wicks. |
| `BullishRailRoad` | Opposite-direction marubozu pair where second candle is bullish. |
| `BearishRailRoad` | Opposite-direction marubozu pair where second candle is bearish. |

### `ThreeCandlePattern`

| Value | Meaning |
|---|---|
| `None` | No three-candle pattern detected. |
| `ThreeWhiteSoldiers` | Three normal/long bullish candles. |
| `ThreeBlackCrows` | Three normal/long bearish candles. |
| `ThreeWhiteSoldiersWithBullishFVG` | Three White Soldiers plus bullish fair value gap. |
| `ThreeWhiteSoldiersWithBearishFVG` | Three White Soldiers plus bearish fair value gap. |
| `ThreeBlackCrowsWithBullishFVG` | Three Black Crows plus bullish fair value gap. |
| `ThreeBlackCrowsWithBearishFVG` | Three Black Crows plus bearish fair value gap. |
| `MorningStar` | Normal/long bearish candle, any Doji middle candle, normal/long bullish candle. |
| `EveningStar` | Normal/long bullish candle, any Doji middle candle, normal/long bearish candle. |
| `BullishAbandonedBaby` | Morning Star where middle Doji high is below both neighboring lows. |
| `BearishAbandonedBaby` | Evening Star where middle Doji low is above both neighboring highs. |
| `EngulfingSandwich` | Candle 2 range engulfs candle 1 range and candle 3 range is inside candle 2 range. |
| `BullishFairValueGap` | Candle 3 low minus candle 1 high is at least `_minGap`. |
| `BearishFairValueGap` | Candle 1 low minus candle 3 high is at least `_minGap`. |

## Exported Types

### `CandleData`

`CandleData` is returned by `analyzeCandle()` and is also the required input type
for multi-candle analyzers. Field order below is the `CandleData.new(...)`
constructor order.

| Field | Type | Set by | Meaning |
|---|---|---|---|
| `wickRange` | `float` | `_high - _low` | Full candle high/low range. |
| `bodyRange` | `float` | `math.max(_open, _close) - math.min(_open, _close)` | Absolute candle body size. |
| `bodyHigh` | `float` | `math.max(_open, _close)` | Higher body price. |
| `bodyLow` | `float` | `math.min(_open, _close)` | Lower body price. |
| `size` | `CandleSize` | `determineCandleSize(...)` | Size classification. |
| `direction` | `CandleDirection` | `determineCandleDirection(...)` | Direction classification. |
| `pattern` | `CandlePattern` | Single-candle pattern priority rules. |
| `openPrice` | `float` | `_open` | Original open price. |
| `highPrice` | `float` | `_high` | Original high price. |
| `lowPrice` | `float` | `_low` | Original low price. |
| `closePrice` | `float` | `_close` | Original close price. |

Constructor order if manually creating test data:

```pine
CPL.CandleData.new(wickRange, bodyRange, bodyHigh, bodyLow, size, direction, pattern, openPrice, highPrice, lowPrice, closePrice)
```

### `TwoCandleData`

Returned by `analyzeTwoCandlePattern(...)`.

| Field | Type | Meaning |
|---|---|---|
| `pattern` | `TwoCandlePattern` | Detected two-candle pattern or `TwoCandlePattern.None`. |
| `candle1` | `CandleData` | First/older candle passed to the analyzer. |
| `candle2` | `CandleData` | Second/newer candle passed to the analyzer. |

Constructor order:

```pine
CPL.TwoCandleData.new(pattern, candle1, candle2)
```

### `ThreeCandleData`

Returned by `analyzeThreeCandlePattern(...)`.

| Field | Type | Meaning |
|---|---|---|
| `pattern` | `ThreeCandlePattern` | Detected three-candle pattern or `ThreeCandlePattern.None`. |
| `candle1` | `CandleData` | First/oldest candle passed to the analyzer. |
| `candle2` | `CandleData` | Second/middle candle passed to the analyzer. |
| `candle3` | `CandleData` | Third/newest candle passed to the analyzer. |

Constructor order:

```pine
CPL.ThreeCandleData.new(pattern, candle1, candle2, candle3)
```

## Exported Analysis Functions

### `analyzeCandle`

```pine
CPL.CandleData CPL.analyzeCandle(
    float _open,
    float _high,
    float _low,
    float _close,
    float _avgSize,
    float _sizeThresholdPct,
    float _equivTolerance,
    float _bodyTolerance,
    int _positionThreshold)
```

| Argument | Type | Meaning |
|---|---|---|
| `_open` | `float` | Candle open price. |
| `_high` | `float` | Candle high price. |
| `_low` | `float` | Candle low price. |
| `_close` | `float` | Candle close price. |
| `_avgSize` | `float` | Baseline wick range for `CandleSize` classification. |
| `_sizeThresholdPct` | `float` | Percent above/below `_avgSize` used for long/short thresholds. |
| `_equivTolerance` | `float` | Absolute price tolerance for equality checks: direction neutrality, Doji body, Marubozu, and equivalent OHLC comparisons. |
| `_bodyTolerance` | `float` | Absolute price tolerance for small-body classification. |
| `_positionThreshold` | `int` | Percent threshold for body placement and wick-section checks. Example: `85` means an 85:15 split. |

Returns `CandleData` with all fields populated.

Single-candle classification details:

- `wickRange = _high - _low`.
- `bodyHigh = math.max(_open, _close)`.
- `bodyLow = math.min(_open, _close)`.
- `bodyRange = bodyHigh - bodyLow`.
- If `wickRange <= 0`, body-position checks use `safeWickRange = _avgSize * 0.001`.
- `hasSmallBody = math.abs(_open - _close) <= _bodyTolerance`.
- `hasEquivBody = math.abs(_open - _close) <= _equivTolerance`.
- `upperThreshold = _low + safeWickRange * (_positionThreshold / 100.0)`.
- `lowerThreshold = _low + safeWickRange * ((100.0 - _positionThreshold) / 100.0)`.
- `bodyInUpperSection = bodyLow >= upperThreshold`.
- `bodyInLowerSection = bodyHigh <= lowerThreshold`.
- `bodyInMiddle = not bodyInUpperSection and not bodyInLowerSection`.

Pattern priority in `analyzeCandle()`:

1. `FourPriceDoji`
2. `DragonflyDoji`
3. `GravestoneDoji`
4. `CrossDoji`
5. `InvertedCrossDoji`
6. `LongLeggedDoji`
7. `Doji`
8. `BullishMarubozu`
9. `BearishMarubozu`
10. `Hammer`, `ShootingStar`, or `SpinningTop`
11. `RegularBullish`
12. `RegularBearish`
13. fallback `Doji`

### `analyzeTwoCandlePattern`

```pine
CPL.TwoCandleData CPL.analyzeTwoCandlePattern(
    CPL.CandleData _candle1,
    CPL.CandleData _candle2,
    float _equivTolerance,
    int _positionThreshold)
```

| Argument | Type | Meaning |
|---|---|---|
| `_candle1` | `CandleData` | First/older candle. Must already come from `analyzeCandle()` or match `CandleData` field semantics. |
| `_candle2` | `CandleData` | Second/newer candle. Must already come from `analyzeCandle()` or match `CandleData` field semantics. |
| `_equivTolerance` | `float` | Absolute price tolerance for equal highs, equal lows, equal body edges, and close/open joins. |
| `_positionThreshold` | `int` | Percent threshold used to compute tweezer minimum wick percent: `(100.0 - _positionThreshold) / 100.0`. |

Returns `TwoCandleData.new(pattern, _candle1, _candle2)`.

Two-candle priority in `analyzeTwoCandlePattern()`:

1. `TweezerTop`
2. `TweezerBottom`
3. `BullishRailRoad`
4. `BearishRailRoad`
5. `BullishEngulfingWeak` or `BullishEngulfingStrong`
6. `BearishEngulfingWeak` or `BearishEngulfingStrong`
7. `InsideBar`
8. fallback `None`

Engulfing details:

- `strictEngulfs = c2BodyHigh > c1BodyHigh and c2BodyLow < c1BodyLow`.
- `tolerantBullishEngulf` requires mismatched directions, second candle bullish,
  `c2BodyHigh > c1BodyHigh`, and `c2BodyLow < c1BodyLow` or equivalent body lows.
- `tolerantBearishEngulf` requires mismatched directions, second candle bearish,
  `c2BodyLow < c1BodyLow`, and `c2BodyHigh > c1BodyHigh` or equivalent body highs.
- `closedInsideRange = c2Close <= c1High and c2Close >= c1Low`.
- Weak engulfing means `closedInsideRange` is true; strong means it is false.

### `analyzeThreeCandlePattern`

```pine
CPL.ThreeCandleData CPL.analyzeThreeCandlePattern(
    CPL.CandleData _candle1,
    CPL.CandleData _candle2,
    CPL.CandleData _candle3,
    float _minGap)
```

| Argument | Type | Meaning |
|---|---|---|
| `_candle1` | `CandleData` | First/oldest candle. |
| `_candle2` | `CandleData` | Second/middle candle. |
| `_candle3` | `CandleData` | Third/newest candle. |
| `_minGap` | `float` | Minimum absolute price gap for fair value gap checks. |

Returns `ThreeCandleData.new(pattern, _candle1, _candle2, _candle3)`.

Three-candle priority in `analyzeThreeCandlePattern()`:

1. `ThreeWhiteSoldiersWithBullishFVG`
2. `ThreeWhiteSoldiersWithBearishFVG`
3. `ThreeBlackCrowsWithBullishFVG`
4. `ThreeBlackCrowsWithBearishFVG`
5. `ThreeWhiteSoldiers`
6. `ThreeBlackCrows`
7. `BullishAbandonedBaby`
8. `BearishAbandonedBaby`
9. `MorningStar`
10. `EveningStar`
11. `EngulfingSandwich`
12. `BullishFairValueGap`
13. `BearishFairValueGap`
14. fallback `None`

### `scoreCandleSentiment`

```pine
[score, shape, power, bodyRatio, wickBias] =
    CPL.scoreCandleSentiment(_open, _high, _low, _close, _atr, _atrMult, _wickWeight)
```

| Argument | Type | Meaning |
|---|---|---|
| `_open` | `float` | Candle open price. |
| `_high` | `float` | Candle high price. |
| `_low` | `float` | Candle low price. |
| `_close` | `float` | Candle close price. |
| `_atr` | `float` | ATR yardstick for range normalization. Must be non-`na` and `> 0` for nonzero power. |
| `_atrMult` | `float` | ATR multiple required for full power. Example: `2.0`. |
| `_wickWeight` | `float` | Wick influence on shape. Intended range is `0.0` to `1.0`. |

Returns a 5-float tuple in this exact order:

| Position | Name | Formula |
|---|---|---|
| 1 | `score` | `shape * power * 100.0` |
| 2 | `shape` | `bodyRatio + _wickWeight * wickBias` |
| 3 | `power` | `math.min((_high - _low) / (_atrMult * _atr), 1.0)` when range and ATR are valid, otherwise `0.0` |
| 4 | `bodyRatio` | `(_close - _open) / (_high - _low)` when range is valid, otherwise `0.0` |
| 5 | `wickBias` | `(lowerWick - upperWick) / (_high - _low)` when range is valid, otherwise `0.0` |

Additional formulas:

```pine
rng = _high - _low
body = _close - _open
upperWick = _high - math.max(_open, _close)
lowerWick = math.min(_open, _close) - _low
rngOk = not na(rng) and rng > 0.0
atrOk = not na(_atr) and _atr > 0.0
```

## Exported Two-Candle Helper Functions

These helpers return `bool`. They do not accept `CandleData` unless explicitly
shown in the signature.

### Helper Argument Glossary

Every exported helper argument is listed here. If an argument appears in multiple
helpers, it has the same meaning in each helper.

| Argument | Type | Meaning |
|---|---|---|
| `_bodyEngulfs` | `bool` | Precomputed result indicating that candle 2 body engulfs candle 1 body. |
| `_c1Pattern` | `CandlePattern` | Single-candle pattern for candle 1, the older/oldest candle. |
| `_c2Pattern` | `CandlePattern` | Single-candle pattern for candle 2, the newer candle in two-candle helpers or middle candle in three-candle helpers. |
| `_c1Dir` | `CandleDirection` | Direction for candle 1, the older/oldest candle. |
| `_c2Dir` | `CandleDirection` | Direction for candle 2, the newer candle in two-candle helpers or middle candle in three-candle helpers. |
| `_c3Dir` | `CandleDirection` | Direction for candle 3, the newest candle. |
| `_targetDir` | `CandleDirection` | Direction that candle 2 must match for helper success. |
| `_c1Size` | `CandleSize` | Size classification for candle 1, the older/oldest candle. |
| `_c2Size` | `CandleSize` | Size classification for candle 2, the middle candle in three-candle helpers. |
| `_c3Size` | `CandleSize` | Size classification for candle 3, the newest candle. |
| `_notShortCandles` | `bool` | True when all relevant candles are not `CandleSize.Short`. |
| `_equalHighs` | `bool` | True when candle 1 and candle 2 highs are equivalent within tolerance. |
| `_equalLows` | `bool` | True when candle 1 and candle 2 lows are equivalent within tolerance. |
| `_c1CloseEqualsC2Open` | `bool` | True when candle 1 close equals candle 2 open within tolerance. |
| `_c1UpperWickPct` | `float` | Candle 1 upper wick divided by candle 1 wick range. |
| `_c2UpperWickPct` | `float` | Candle 2 upper wick divided by candle 2 wick range. |
| `_c1LowerWickPct` | `float` | Candle 1 lower wick divided by candle 1 wick range. |
| `_c2LowerWickPct` | `float` | Candle 2 lower wick divided by candle 2 wick range. |
| `_minWickPct` | `float` | Minimum wick percentage required for tweezer validation. |
| `_c1High` | `float` | High of candle 1, the older/oldest candle. |
| `_c1Low` | `float` | Low of candle 1, the older/oldest candle. |
| `_c2High` | `float` | High of candle 2, the newer candle in two-candle helpers or middle candle in three-candle helpers. |
| `_c2Low` | `float` | Low of candle 2, the newer candle in two-candle helpers or middle candle in three-candle helpers. |
| `_c3High` | `float` | High of candle 3, the newest candle. |
| `_c3Low` | `float` | Low of candle 3, the newest candle. |
| `_minGap` | `float` | Minimum absolute gap size for fair value gap validation. |

### `isEngulfingPattern`

```pine
bool CPL.isEngulfingPattern(bool _bodyEngulfs, CPL.CandleDirection _c2Dir, CPL.CandleDirection _targetDir)
```

Returns `_bodyEngulfs and _c2Dir == _targetDir`.

| Argument | Type | Meaning |
|---|---|---|
| `_bodyEngulfs` | `bool` | True if the second candle body engulfs the first candle body. |
| `_c2Dir` | `CandleDirection` | Direction of the second/newer candle. |
| `_targetDir` | `CandleDirection` | Direction required for the pattern, usually `Bullish` or `Bearish`. |

### `isInsideBarPattern`

```pine
bool CPL.isInsideBarPattern(float _c2High, float _c1High, float _c2Low, float _c1Low)
```

Returns `_c2High < _c1High and _c2Low > _c1Low`.

| Argument | Type | Meaning |
|---|---|---|
| `_c2High` | `float` | High of the second/newer candle. |
| `_c1High` | `float` | High of the first/older candle. |
| `_c2Low` | `float` | Low of the second/newer candle. |
| `_c1Low` | `float` | Low of the first/older candle. |

### `isTweezerTopPattern`

```pine
bool CPL.isTweezerTopPattern(
    bool _notShortCandles,
    bool _equalHighs,
    bool _c1CloseEqualsC2Open,
    CPL.CandleDirection _c1Dir,
    CPL.CandleDirection _c2Dir,
    float _c1UpperWickPct,
    float _c2UpperWickPct,
    float _minWickPct)
```

Returns true when all of these are true:

- `_notShortCandles`
- `_equalHighs`
- `_c1CloseEqualsC2Open`
- `_c1Dir == CPL.CandleDirection.Bullish`
- `_c2Dir == CPL.CandleDirection.Bearish`
- `_c1UpperWickPct >= _minWickPct`
- `_c2UpperWickPct >= _minWickPct`

### `isTweezerBottomPattern`

```pine
bool CPL.isTweezerBottomPattern(
    bool _notShortCandles,
    bool _equalLows,
    bool _c1CloseEqualsC2Open,
    CPL.CandleDirection _c1Dir,
    CPL.CandleDirection _c2Dir,
    float _c1LowerWickPct,
    float _c2LowerWickPct,
    float _minWickPct)
```

Returns true when all of these are true:

- `_notShortCandles`
- `_equalLows`
- `_c1CloseEqualsC2Open`
- `_c1Dir == CPL.CandleDirection.Bearish`
- `_c2Dir == CPL.CandleDirection.Bullish`
- `_c1LowerWickPct >= _minWickPct`
- `_c2LowerWickPct >= _minWickPct`

### `isRailRoadPattern`

```pine
bool CPL.isRailRoadPattern(
    CPL.CandlePattern _c1Pattern,
    CPL.CandlePattern _c2Pattern,
    CPL.CandleDirection _c1Dir,
    CPL.CandleDirection _c2Dir,
    CPL.CandleDirection _targetDir)
```

Returns true when:

- `_c1Pattern` is `BullishMarubozu` or `BearishMarubozu`.
- `_c2Pattern` is `BullishMarubozu` or `BearishMarubozu`.
- `_c1Dir != _c2Dir`.
- `_c2Dir == _targetDir`.

## Exported Three-Candle Helper Functions

These helpers return `bool`.

### `isThreeWhiteSoldiersPattern`

```pine
bool CPL.isThreeWhiteSoldiersPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleSize _c2Size,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c1Dir,
    CPL.CandleDirection _c2Dir,
    CPL.CandleDirection _c3Dir)
```

Returns true when all three sizes are `Normal` or `Long`, and all three
directions are `Bullish`.

### `isThreeBlackCrowsPattern`

```pine
bool CPL.isThreeBlackCrowsPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleSize _c2Size,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c1Dir,
    CPL.CandleDirection _c2Dir,
    CPL.CandleDirection _c3Dir)
```

Returns true when all three sizes are `Normal` or `Long`, and all three
directions are `Bearish`.

### `isMorningStarPattern`

```pine
bool CPL.isMorningStarPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleDirection _c1Dir,
    CPL.CandlePattern _c2Pattern,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c3Dir)
```

Returns true when:

- Candle 1 is `Normal` or `Long`.
- Candle 1 direction is `Bearish`.
- Candle 2 pattern is one of the Doji variants listed in "Doji Pattern Set".
- Candle 3 is `Normal` or `Long`.
- Candle 3 direction is `Bullish`.

### `isEveningStarPattern`

```pine
bool CPL.isEveningStarPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleDirection _c1Dir,
    CPL.CandlePattern _c2Pattern,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c3Dir)
```

Returns true when:

- Candle 1 is `Normal` or `Long`.
- Candle 1 direction is `Bullish`.
- Candle 2 pattern is one of the Doji variants listed in "Doji Pattern Set".
- Candle 3 is `Normal` or `Long`.
- Candle 3 direction is `Bearish`.

### `isBullishAbandonedBabyPattern`

```pine
bool CPL.isBullishAbandonedBabyPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleDirection _c1Dir,
    float _c1Low,
    CPL.CandlePattern _c2Pattern,
    float _c2High,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c3Dir,
    float _c3Low)
```

Returns true when:

- Candle 1 is `Normal` or `Long`.
- Candle 1 direction is `Bearish`.
- Candle 2 pattern is one of the Doji variants listed in "Doji Pattern Set".
- `_c2High < _c1Low`.
- `_c2High < _c3Low`.
- Candle 3 is `Normal` or `Long`.
- Candle 3 direction is `Bullish`.

### `isBearishAbandonedBabyPattern`

```pine
bool CPL.isBearishAbandonedBabyPattern(
    CPL.CandleSize _c1Size,
    CPL.CandleDirection _c1Dir,
    float _c1High,
    CPL.CandlePattern _c2Pattern,
    float _c2Low,
    CPL.CandleSize _c3Size,
    CPL.CandleDirection _c3Dir,
    float _c3High)
```

Returns true when:

- Candle 1 is `Normal` or `Long`.
- Candle 1 direction is `Bullish`.
- Candle 2 pattern is one of the Doji variants listed in "Doji Pattern Set".
- `_c2Low > _c1High`.
- `_c2Low > _c3High`.
- Candle 3 is `Normal` or `Long`.
- Candle 3 direction is `Bearish`.

### `isEngulfingSandwichPattern`

```pine
bool CPL.isEngulfingSandwichPattern(
    float _c1High,
    float _c1Low,
    float _c2High,
    float _c2Low,
    float _c3High,
    float _c3Low)
```

Returns `_c2High > _c1High and _c2Low < _c1Low and _c3High < _c2High and _c3Low > _c2Low`.

### `isBullishFairValueGapPattern`

```pine
bool CPL.isBullishFairValueGapPattern(float _c1High, float _c3Low, float _minGap)
```

Returns `_c3Low - _c1High >= _minGap`.

Argument order is strict:

| Argument | Type | Meaning |
|---|---|---|
| `_c1High` | `float` | High of the first/oldest candle. |
| `_c3Low` | `float` | Low of the third/newest candle. |
| `_minGap` | `float` | Minimum absolute gap size. |

### `isBearishFairValueGapPattern`

```pine
bool CPL.isBearishFairValueGapPattern(float _c1Low, float _c3High, float _minGap)
```

Returns `_c1Low - _c3High >= _minGap`.

Argument order is strict:

| Argument | Type | Meaning |
|---|---|---|
| `_c1Low` | `float` | Low of the first/oldest candle. |
| `_c3High` | `float` | High of the third/newest candle. |
| `_minGap` | `float` | Minimum absolute gap size. |

## Exported Name Helpers

These helpers are cosmetic. Compare enum values in logic.

### `getPatternName`

```pine
string CPL.getPatternName(CPL.CandlePattern _pattern)
```

| Input enum | Returned string |
|---|---|
| `CandlePattern.Unknown` | `"Unknown"` |
| `CandlePattern.RegularBullish` | `"Regular Bullish"` |
| `CandlePattern.RegularBearish` | `"Regular Bearish"` |
| `CandlePattern.BullishMarubozu` | `"Bullish Marubozu"` |
| `CandlePattern.BearishMarubozu` | `"Bearish Marubozu"` |
| `CandlePattern.Hammer` | `"Hammer"` |
| `CandlePattern.ShootingStar` | `"Shooting Star"` |
| `CandlePattern.SpinningTop` | `"Spinning Top"` |
| `CandlePattern.Doji` | `"Doji"` |
| `CandlePattern.LongLeggedDoji` | `"Long Legged Doji"` |
| `CandlePattern.CrossDoji` | `"Cross Doji"` |
| `CandlePattern.DragonflyDoji` | `"Dragonfly Doji"` |
| `CandlePattern.InvertedCrossDoji` | `"Inverted Cross Doji"` |
| `CandlePattern.GravestoneDoji` | `"Gravestone Doji"` |
| `CandlePattern.FourPriceDoji` | `"4 Price Doji"` |
| fallback | `"Unknown"` |

### `getTwoCandlePatternName`

```pine
string CPL.getTwoCandlePatternName(CPL.TwoCandlePattern _pattern)
```

| Input enum | Returned string |
|---|---|
| `TwoCandlePattern.None` | `"None"` |
| `TwoCandlePattern.BullishEngulfingWeak` | `"Bullish Engulfing (Weak)"` |
| `TwoCandlePattern.BullishEngulfingStrong` | `"Bullish Engulfing (Strong)"` |
| `TwoCandlePattern.BearishEngulfingWeak` | `"Bearish Engulfing (Weak)"` |
| `TwoCandlePattern.BearishEngulfingStrong` | `"Bearish Engulfing (Strong)"` |
| `TwoCandlePattern.InsideBar` | `"Inside Bar"` |
| `TwoCandlePattern.TweezerTop` | `"Tweezer Top"` |
| `TwoCandlePattern.TweezerBottom` | `"Tweezer Bottom"` |
| `TwoCandlePattern.BullishRailRoad` | `"Bullish Rail Road"` |
| `TwoCandlePattern.BearishRailRoad` | `"Bearish Rail Road"` |
| fallback | `"None"` |

### `getThreeCandlePatternName`

```pine
string CPL.getThreeCandlePatternName(CPL.ThreeCandlePattern _pattern)
```

| Input enum | Returned string |
|---|---|
| `ThreeCandlePattern.None` | `"None"` |
| `ThreeCandlePattern.ThreeWhiteSoldiers` | `"Three White Soldiers"` |
| `ThreeCandlePattern.ThreeBlackCrows` | `"Three Black Crows"` |
| `ThreeCandlePattern.ThreeWhiteSoldiersWithBullishFVG` | `"Three White Soldiers + Bullish FVG"` |
| `ThreeCandlePattern.ThreeWhiteSoldiersWithBearishFVG` | `"Three White Soldiers + Bearish FVG"` |
| `ThreeCandlePattern.ThreeBlackCrowsWithBullishFVG` | `"Three Black Crows + Bullish FVG"` |
| `ThreeCandlePattern.ThreeBlackCrowsWithBearishFVG` | `"Three Black Crows + Bearish FVG"` |
| `ThreeCandlePattern.MorningStar` | `"Morning Star"` |
| `ThreeCandlePattern.EveningStar` | `"Evening Star"` |
| `ThreeCandlePattern.BullishAbandonedBaby` | `"Bullish Abandoned Baby"` |
| `ThreeCandlePattern.BearishAbandonedBaby` | `"Bearish Abandoned Baby"` |
| `ThreeCandlePattern.EngulfingSandwich` | `"Engulfing Sandwich"` |
| `ThreeCandlePattern.BullishFairValueGap` | `"Bullish Fair Value Gap"` |
| `ThreeCandlePattern.BearishFairValueGap` | `"Bearish Fair Value Gap"` |
| fallback | `"None"` |

### `getSizeName`

```pine
string CPL.getSizeName(CPL.CandleSize _size)
```

| Input enum | Returned string |
|---|---|
| `CandleSize.Short` | `"Short"` |
| `CandleSize.Normal` | `"Normal"` |
| `CandleSize.Long` | `"Long"` |
| fallback | `"Unknown"` |

### `getDirectionName`

```pine
string CPL.getDirectionName(CPL.CandleDirection _direction)
```

| Input enum | Returned string |
|---|---|
| `CandleDirection.Bearish` | `"Bearish"` |
| `CandleDirection.Neutral` | `"Neutral"` |
| `CandleDirection.Bullish` | `"Bullish"` |
| fallback | `"Unknown"` |

## Shared Pattern Sets

### Doji Pattern Set

The star and abandoned-baby helpers treat these `CandlePattern` values as Doji
middle candles:

- `Doji`
- `LongLeggedDoji`
- `CrossDoji`
- `DragonflyDoji`
- `InvertedCrossDoji`
- `GravestoneDoji`
- `FourPriceDoji`

`CrossDoji` and `InvertedCrossDoji` are intentionally distinct enum values.

### Normal Or Long Size Set

Many three-candle helpers require each relevant candle size to be either:

- `CPL.CandleSize.Normal`
- `CPL.CandleSize.Long`

They reject `CPL.CandleSize.Short`.

## Parameter Guidance

| Parameter | Units | Consumer guidance |
|---|---|---|
| `_avgSize` | Price units | Baseline candle wick range. The library does not calculate ATR internally for this value. |
| `_sizeThresholdPct` | Percent | Example `50.0` means long above 150% of `_avgSize`, short below 50% of `_avgSize`. |
| `_equivTolerance` | Price units | Absolute tolerance, not ticks unless caller converts ticks to price. |
| `_bodyTolerance` | Price units | Absolute small-body tolerance, usually wider than `_equivTolerance`. |
| `_positionThreshold` | Integer percent | Use values such as `75`, `80`, `85`, `90`, `95`, `99`. Do not pass a string like `"85:15"`. |
| `_minGap` | Price units | Minimum absolute gap for FVG helpers and three-candle analyzer. |
| `_atr` | Price units | Use `ta.atr(length)` or another ATR-equivalent series. If `na` or `<= 0`, sentiment power and score are `0`. |
| `_atrMult` | Multiplier | Full sentiment power occurs when range reaches `_atrMult * _atr`. |
| `_wickWeight` | Unitless | Intended range `0.0` to `1.0`; the library does not clamp it. |

## Integration Rules And Pitfalls

| Rule | Why it matters |
|---|---|
| Analyze single candles first. | Multi-candle analyzers depend on `CandleData.size`, `direction`, `pattern`, and OHLC fields. |
| Preserve chronological order. | Passing newest first reverses bullish/bearish pattern semantics. |
| Use the same thresholds across a candle sequence. | Mixed threshold settings make multi-candle outputs inconsistent. |
| Keep tolerances in price units. | `_equivTolerance`, `_bodyTolerance`, and `_minGap` are direct absolute comparisons. |
| Compare enums, not strings. | Name helper strings are display output and can be slower/noisier in logic. |
| Do not pass `CandleData` into FVG helpers. | FVG helpers require raw floats in strict order. |
| Do not assume sentiment score is clamped. | The power term is capped at `1.0`, but shape can exceed `[-1, 1]` if `_wickWeight` is outside normal bounds. |
| Treat helper functions as predicate primitives. | Higher-level analyzers add priority rules and weak/strong classification that helpers alone do not provide. |

## Recommended Indicator Inputs

Expose these inputs in scripts that use the library:

| User input | Type | Typical default | Passed to |
|---|---|---|---|
| Use ATR for Average Size | `bool` | `false` | Caller decides `_avgSize`. |
| ATR Length | `int` | `14` | `ta.atr(length)` for `_avgSize` or sentiment `_atr`. |
| Average Candle Size | `float` | `250.0` | `_avgSize` when ATR is not used. |
| Size Threshold (%) | `float` | `50.0` | `_sizeThresholdPct`. |
| Equivalence Tolerance | `float` | `10.0` | `_equivTolerance`. |
| Body Tolerance | `float` | `50.0` | `_bodyTolerance`. |
| Position Threshold | `int` | `85` | `_positionThreshold`. |
| Minimum Fair Value Gap | `float` | `10.0` | `_minGap`. |
| Sentiment ATR Mult | `float` | `2.0` | `_atrMult`. |
| Sentiment Wick Weight | `float` | `0.5` | `_wickWeight`. |
