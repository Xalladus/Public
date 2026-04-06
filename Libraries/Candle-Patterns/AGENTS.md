# AGENTS.md - Public/Libraries/Candle-Patterns

Pine Script v6 library for single, two-candle, and three-candle pattern analysis.
The library classifies candle size and direction first, then derives higher-level
patterns from `CandleData` objects. It also exports candle sentiment scoring for
ATR-normalized strength measurements. Published as `OneCleverGuy/CandlePatternLibrary/4`.

```pine
import OneCleverGuy/CandlePatternLibrary/4 as CPL
```

---

## Function Tree

```
CPL
|
+-- PRIMARY ANALYSIS PIPELINE
|   |
|   +-- analyzeCandle(_open, _high, _low, _close, _avgSize,
|   |      _sizeThresholdPct, _equivTolerance, _bodyTolerance,
|   |      _positionThreshold) -> CandleData
|   |    Analyzes one candle and returns size, direction, pattern, and OHLC.
|   |
|   +-- analyzeTwoCandlePattern(_candle1, _candle2, _equivTolerance,
|   |      _positionThreshold) -> TwoCandleData
|   |    Detects engulfing, inside bar, tweezer, and railroad patterns.
|   |    Candle order must be older -> newer.
|   |
|   +-- analyzeThreeCandlePattern(_candle1, _candle2, _candle3, _minGap)
|        -> ThreeCandleData
|        Detects three-candle structures, abandoned babies, stars,
|        engulfing sandwich, and fair value gaps.
|        Candle order must be oldest -> newest.
|
+-- CANDLE SENTIMENT SCORING
|   |
|   +-- scoreCandleSentiment(_open, _high, _low, _close,
|          _atr, _atrMult, _wickWeight) -> [score, shape, power, bodyRatio, wickBias]
|        Scores one candle on [-100..+100] as:
|        (bodyRatio + _wickWeight * wickBias) * min(range / (_atrMult * _atr), 1) * 100.
|
+-- TWO-CANDLE HELPERS
|   |
|   +-- isEngulfingPattern(_bodyEngulfs, _c2Dir, _targetDir) -> bool
|   +-- isInsideBarPattern(_c2High, _c1High, _c2Low, _c1Low) -> bool
|   +-- isTweezerTopPattern(_notShortCandles, _equalHighs,
|   |      _c1CloseEqualsC2Open, _c1Dir, _c2Dir,
|   |      _c1UpperWickPct, _c2UpperWickPct, _minWickPct) -> bool
|   +-- isTweezerBottomPattern(_notShortCandles, _equalLows,
|   |      _c1CloseEqualsC2Open, _c1Dir, _c2Dir,
|   |      _c1LowerWickPct, _c2LowerWickPct, _minWickPct) -> bool
|   +-- isRailRoadPattern(_c1Pattern, _c2Pattern, _c1Dir, _c2Dir,
|        _targetDir) -> bool
|
+-- THREE-CANDLE HELPERS
|   |
|   +-- isThreeWhiteSoldiersPattern(_c1Size, _c2Size, _c3Size,
|   |      _c1Dir, _c2Dir, _c3Dir) -> bool
|   +-- isThreeBlackCrowsPattern(_c1Size, _c2Size, _c3Size,
|   |      _c1Dir, _c2Dir, _c3Dir) -> bool
|   +-- isMorningStarPattern(_c1Size, _c1Dir, _c2Pattern, _c3Size,
|   |      _c3Dir) -> bool
|   +-- isEveningStarPattern(_c1Size, _c1Dir, _c2Pattern, _c3Size,
|   |      _c3Dir) -> bool
|   +-- isBullishAbandonedBabyPattern(_c1Size, _c1Dir, _c1Low, _c2Pattern,
|   |      _c2High, _c3Size, _c3Dir, _c3Low) -> bool
|   +-- isBearishAbandonedBabyPattern(_c1Size, _c1Dir, _c1High, _c2Pattern,
|   |      _c2Low, _c3Size, _c3Dir, _c3High) -> bool
|   +-- isEngulfingSandwichPattern(_c1High, _c1Low, _c2High, _c2Low,
|   |      _c3High, _c3Low) -> bool
|   +-- isBullishFairValueGapPattern(_c1High, _c3Low, _minGap) -> bool
|   +-- isBearishFairValueGapPattern(_c1Low, _c3High, _minGap) -> bool
|
+-- DISPLAY / LOGGING HELPERS
    |
    +-- getPatternName(_pattern) -> string
    +-- getTwoCandlePatternName(_pattern) -> string
    +-- getThreeCandlePatternName(_pattern) -> string
    +-- getSizeName(_size) -> string
    +-- getDirectionName(_direction) -> string
```

---

## Use Instructions

### Standard Call Order

1. Import the library as `CPL`.
2. Compute `CandleData` for each candle you want to analyze with `analyzeCandle()`.
3. Pass candles in chronological order into the multi-candle analyzers:
   `analyzeTwoCandlePattern(older, newer, ...)`
   `analyzeThreeCandlePattern(oldest, middle, newest, ...)`
4. Read enum outputs from the returned UDTs.
5. Use the naming helpers only for labels, tables, logs, or alerts.

### Standard Integration Pattern

```pine
import OneCleverGuy/CandlePatternLibrary/4 as CPL

float avgSize           = 250.0
float sizeThresholdPct  = 50.0
float equivTolerance    = 10.0
float bodyTolerance     = 50.0
int   positionThreshold = 85
float minGap            = 10.0

CPL.CandleData candleNewest = CPL.analyzeCandle(
    open, high, low, close,
    avgSize, sizeThresholdPct, equivTolerance, bodyTolerance, positionThreshold)

CPL.CandleData candleMiddle = CPL.analyzeCandle(
    open[1], high[1], low[1], close[1],
    avgSize, sizeThresholdPct, equivTolerance, bodyTolerance, positionThreshold)

CPL.CandleData candleOldest = CPL.analyzeCandle(
    open[2], high[2], low[2], close[2],
    avgSize, sizeThresholdPct, equivTolerance, bodyTolerance, positionThreshold)

CPL.TwoCandleData twoBar = CPL.analyzeTwoCandlePattern(
    candleMiddle, candleNewest, equivTolerance, positionThreshold)

CPL.ThreeCandleData threeBar = CPL.analyzeThreeCandlePattern(
    candleOldest, candleMiddle, candleNewest, minGap)

[sentScore, sentShape, sentPower, sentBody, sentWick] = CPL.scoreCandleSentiment(
    open, high, low, close,
    ta.atr(14), 2.0, 0.5)
```

---

## Exported Types

| Type | Purpose |
|------|---------|
| `CandleData` | Full single-candle analysis: ranges, direction, size, pattern, OHLC |
| `TwoCandleData` | Two-candle pattern result plus both analyzed candles |
| `ThreeCandleData` | Three-candle pattern result plus all three analyzed candles |

---

## Enum Reference

| Enum | Values |
|------|--------|
| `CandleSize` | `Short`, `Normal`, `Long` |
| `CandleDirection` | `Bearish`, `Neutral`, `Bullish` |
| `CandlePattern` | `Unknown`, `RegularBullish`, `RegularBearish`, `BullishMarubozu`, `BearishMarubozu`, `Hammer`, `ShootingStar`, `SpinningTop`, `Doji`, `LongLeggedDoji`, `CrossDoji`, `DragonflyDoji`, `InvertedCrossDoji`, `GravestoneDoji`, `FourPriceDoji` |
| `TwoCandlePattern` | `None`, `BullishEngulfingWeak`, `BullishEngulfingStrong`, `BearishEngulfingWeak`, `BearishEngulfingStrong`, `InsideBar`, `TweezerTop`, `TweezerBottom`, `BullishRailRoad`, `BearishRailRoad` |
| `ThreeCandlePattern` | `None`, `ThreeWhiteSoldiers`, `ThreeBlackCrows`, `ThreeWhiteSoldiersWithBullishFVG`, `ThreeWhiteSoldiersWithBearishFVG`, `ThreeBlackCrowsWithBullishFVG`, `ThreeBlackCrowsWithBearishFVG`, `MorningStar`, `EveningStar`, `BullishAbandonedBaby`, `BearishAbandonedBaby`, `EngulfingSandwich`, `BullishFairValueGap`, `BearishFairValueGap` |

Use enum references as `CPL.CandlePattern.Hammer`, `CPL.TwoCandlePattern.InsideBar`,
and `CPL.ThreeCandlePattern.MorningStar`.

---

## Parameter Notes

| Parameter | Meaning |
|-----------|---------|
| `_avgSize` | Baseline wick range used to classify candles as `Short`, `Normal`, or `Long` |
| `_sizeThresholdPct` | Percent deviation from `_avgSize` for size classification |
| `_equivTolerance` | Absolute tolerance used for equality tests like Doji, Marubozu, and Tweezers |
| `_bodyTolerance` | Absolute tolerance used for "small body" classification |
| `_positionThreshold` | Integer threshold such as `85` for `85:15` body placement and wick checks |
| `_minGap` | Minimum absolute gap required for Fair Value Gap detection |
| `_atr` | ATR yardstick for sentiment scoring (must be > 0 to produce non-zero power) |
| `_atrMult` | Range/ATR target for full sentiment power (`power = 1`) |
| `_wickWeight` | Wick influence in sentiment shape, from `0` (body-only) to `1` (max wick influence) |

---

## FVG Helper Signatures (Strict)

Use the helper functions with exactly **three float arguments** each:

```pine
bool bullishFvg = CPL.isBullishFairValueGapPattern(c1High, c3Low, minGap)
bool bearishFvg = CPL.isBearishFairValueGapPattern(c1Low, c3High, minGap)
```

Argument order:

| Function | Argument 1 | Argument 2 | Argument 3 |
|----------|------------|------------|------------|
| `isBullishFairValueGapPattern` | `_c1High` | `_c3Low` | `_minGap` |
| `isBearishFairValueGapPattern` | `_c1Low` | `_c3High` | `_minGap` |

Do not pass `CandleData` objects directly into these helpers. Extract the required
price fields first, or call `analyzeThreeCandlePattern()` which handles this wiring.

---

## Rules & Pitfalls

| Rule | Detail |
|------|--------|
| Analyze single candles first | Multi-candle functions expect fully built `CandleData` inputs |
| Preserve chronological order | Two-candle uses older -> newer; three-candle uses oldest -> newest |
| Keep tolerances in price units | `_equivTolerance`, `_bodyTolerance`, and `_minGap` are absolute values, not ticks or percentages |
| Use one consistent threshold set | Recompute all candles in a sequence with the same `_avgSize`, tolerances, and `_positionThreshold` |
| `positionThreshold` is an integer | Use values like `75`, `80`, `85`, `90`, `95`, not strings like `"85:15"` |
| Sentiment ATR must be valid | If `_atr` is `na` or `<= 0`, sentiment power returns `0`, so score is `0` |
| Naming helpers are cosmetic | Pattern logic should compare enums, not returned strings |
| Neutral candles can collapse to Doji | If no directional pattern matches, `analyzeCandle()` can still end as `Doji` |
| FVG checks are independent | Three-candle analysis can classify pure bullish/bearish FVGs even when star/soldier/crow patterns do not match |

---

## Recommended Script Inputs

Expose these in indicators or strategies that use the library:

| Input | Typical Default |
|------|------------------|
| Average Candle Size | `250.0` |
| Size Threshold (%) | `50.0` |
| Equivalence Tolerance | `10.0` |
| Body Tolerance | `50.0` |
| Position Threshold | `85` |
| Minimum Fair Value Gap | `10.0` |
| Sentiment ATR Length | `14` |
| Sentiment ATR Mult | `2.0` |
| Sentiment Wick Weight | `0.5` |

If using ATR for `_avgSize`, compute that in the script and pass the resulting float
into `analyzeCandle()`. The library does not compute ATR internally.
