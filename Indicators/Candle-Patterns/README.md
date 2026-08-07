# Candle Pattern Library And Indicator

Pine Script v6 candle-analysis tools for TradingView. The published library
classifies single candles, detects two- and three-candle patterns, scores candle
sentiment, and exposes result-family helpers. Indicators own inputs, visuals,
debug logging, and any alert or strategy policy.

## Files

- `Libraries/Candle-Patterns/1CG-Candle-Pattern-Library.pine` is the canonical
  publishable library source.
- `Libraries/Candle-Patterns/1CG-Candle-Pattern-Library-Publication.txt` is the
  TradingView publication description and release history.
- `Libraries/Candle-Patterns/AGENTS.md` is the complete engineering contract.
- `Indicators/Candle-Patterns/Candle-Pattern-Indicator.pine` is the current
  consumer/reference indicator for the published API.
- `Indicators/Candle-Patterns/Candle-Pattern+Lib.pine` is the legacy combined
  development harness. It is retained as historical reference; do not treat its
  inline definitions as the canonical library source.

## Import

After publishing version 6:

```pine
import OneCleverGuy/CandlePatternLibrary/6 as CPL
```

## Standard Analysis Flow

```pine
float averageRange = ta.atr(14)

CPL.CandleAnalysisConfig candleConfig = CPL.CandleAnalysisConfig.new(
    sizeThresholdPct  = 50.0,
    equivTolerance    = syminfo.mintick * 10.0,
    bodyTolerance     = syminfo.mintick * 50.0,
    positionThreshold = 85,
    fvgMinGap         = averageRange * 0.5,
    atrMultiplier     = 2.0,
    wickWeight        = 0.5)

CPL.CandleData newest = CPL.analyzeCandle(
    open, high, low, close, averageRange, candleConfig)

CPL.CandleData middle = CPL.analyzeCandle(
    open[1], high[1], low[1], close[1], averageRange[1], candleConfig)

CPL.CandleData oldest = CPL.analyzeCandle(
    open[2], high[2], low[2], close[2], averageRange[2], candleConfig)

CPL.TwoCandleData two = CPL.analyzeTwoCandlePattern(
    middle, newest, candleConfig)

CPL.ThreeCandleData three = CPL.analyzeThreeCandlePattern(
    oldest, middle, newest, candleConfig)
```

Chronological order is strict: older then newer for two-candle analysis, and
oldest, middle, newest for three-candle analysis.

## Result Families

Version 6 adds helpers for classifying completed enum results without repeating
enum-member lists in every consumer:

```pine
bool isReversal = CPL.isReversalSinglePattern(newest.pattern)
bool isDoji = CPL.isDojiPattern(newest.pattern)
bool isMarubozu = CPL.isMarubozuPattern(newest.pattern)
bool isEngulfing = CPL.isEngulfingTwoPattern(two.pattern)
bool isTweezer = CPL.isTweezerTwoPattern(two.pattern)
bool isRailRoad = CPL.isRailRoadTwoPattern(two.pattern)
bool isStandaloneFvg =
    CPL.isStandaloneFairValueGapThreePattern(three.pattern)
```

These helpers classify results only. Consumers still decide which families to
display, journal, alert, or trade. For example, requiring a Long second candle
for engulfing is consumer policy unless the core analyzer definition is changed
deliberately.

## Detected Results

### Single candle

- Regular bullish and bearish candles
- Bullish and bearish Marubozu
- Hammer, Shooting Star, and Spinning Top
- Doji, Long-Legged Doji, Cross Doji, Inverted Cross Doji, Dragonfly Doji,
  Gravestone Doji, and Four-Price Doji

### Two candle

- Bullish and bearish engulfing, weak and strong
- Inside Bar
- Tweezer Top and Bottom
- Bullish and bearish Rail Road

### Three candle

- Three White Soldiers and Three Black Crows
- Soldier/crow combinations with bullish or bearish FVG
- Morning Star and Evening Star
- Bullish and bearish Abandoned Baby
- Engulfing Sandwich
- Standalone bullish and bearish Fair Value Gap

## Ownership Rules

- The library owns stateless classification, detection, sentiment scoring,
  result-family membership, and enum display names.
- Consumers own average-size and ATR series, tolerance conversion, input
  declarations, persistent state, rendering, alerts, and strategy decisions.
- Tolerances and FVG minimums are absolute price units. Convert ticks or ATR
  multiples before constructing `CandleAnalysisConfig`.
- Reuse one config across every candle in the same multi-candle sequence.
- The library imports nothing and creates no chart objects.

See `Libraries/Candle-Patterns/AGENTS.md` for the full API, priority rules, data
contracts, and integration pitfalls.
