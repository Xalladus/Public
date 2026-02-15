# Candlestick Pattern Library (CPL)

## Overview

The Candlestick Pattern Library is a comprehensive Pine Script library designed for TradingView that analyzes and identifies common candlestick patterns in real-time. This library provides a modular, reusable framework for detecting both single-candle and two-candle patterns with customizable tolerance settings.

## File Relationships

- `Indicators/Candle-Patterns/Candle-Pattern+Lib.ps`: Combined dev script that contains both the indicator scaffold and the full library code in one place for iterative testing.
- `Public/Libraries/Candle-Patterns/1CG-Candle-Pattern-Library.ps`: Published library file; the exported version of the library functions/types consumed via `import`.
- `Indicators/Candle-Patterns/Candle-Pattern-Indicator.ps`: Lean indicator that imports the published library (`1CG-Candle-Pattern-Library.ps`) and handles inputs, logging, and visualization.

When iterating: edit and test inside `Candle-Pattern+Lib.ps`, then mirror the library changes into `1CG-Candle-Pattern-Library.ps` and keep the indicator logic in `Candle-Pattern-Indicator.ps` in sync with the combined dev file.

## Architecture

The library is currently implemented as a standalone indicator for testing purposes. Once complete, it will be extracted into a separate library file that can be imported into other strategies and indicators.

### Design Philosophy

- **Pattern Detection at Bar Close**: All pattern identification occurs when a new candle opens, analyzing the previous (completed) candle
- **Configurable Tolerances**: Uses both absolute and percentage-based tolerances for flexible pattern matching across different instruments and timeframes
- **Comprehensive Analysis**: Returns complete candle data including size, direction, range metrics, and pattern classification
- **Clean Separation**: Library functions (prefixed with `CPL_`) are isolated and side-effect free, making extraction straightforward

## Detected Patterns

### Single Candle Patterns

The library identifies the following single-candle patterns:

**Marubozu Patterns:**
- **Bullish Marubozu**: Open equals low, close equals high (strong buying pressure)
- **Bearish Marubozu**: Open equals high, close equals low (strong selling pressure)

**Small Body Patterns:**
- **Hammer**: Small body in upper section of wick range (potential bullish reversal)
- **Shooting Star**: Small body in lower section of wick range (potential bearish reversal)
- **Spinning Top**: Small body in middle section of wick range (indecision)

**Doji Patterns:**
- **Regular Doji**: Short candle with open ≈ close
- **Long Legged Doji**: Normal/long candle with open ≈ close, body in middle
- **Cross Doji**: Normal/long candle with open ≈ close, body in upper section
- **Inverted Cross Doji**: Normal/long candle with open ≈ close, body in lower section
- **Dragonfly Doji**: Open ≈ close ≈ high (long lower shadow)
- **Gravestone Doji**: Open ≈ close ≈ low (long upper shadow)
- **4 Price Doji**: All prices (open, high, low, close) essentially equal

**Standard Patterns:**
- **Regular Bullish**: Normal candle with close > open
- **Regular Bearish**: Normal candle with close < open

### Two Candle Patterns

The library also detects the following two-candle patterns:

**Engulfing Patterns:**
- **Bullish Engulfing (Strong)**: Second candle body engulfs first candle body, closes outside first candle's range
- **Bullish Engulfing (Weak)**: Second candle body engulfs first candle body, closes inside first candle's range
- **Bearish Engulfing (Strong)**: Second candle body engulfs first candle body, closes outside first candle's range
- **Bearish Engulfing (Weak)**: Second candle body engulfs first candle body, closes inside first candle's range

**Other Patterns:**
- **Inside Bar**: Second candle's high/low completely contained within first candle's high/low
- **Tweezer Top**: Equal highs, first candle bullish, second candle bearish, close of first = open of second
- **Tweezer Bottom**: Equal lows, first candle bearish, second candle bullish, close of first = open of second

## Core Data Types

### CandleData

Contains complete analysis results for a single candle:

```pine
type CandleData
    float           wickRange       // Total range from high to low
    float           bodyRange       // Range from open to close (absolute)
    float           bodyHigh        // Higher of open/close
    float           bodyLow         // Lower of open/close
    CandleSize      size            // Short/Normal/Long classification
    CandleDirection direction       // Bearish/Neutral/Bullish
    CandlePattern   pattern         // Identified pattern
    float           openPrice       // Original open price
    float           highPrice       // Original high price
    float           lowPrice        // Original low price
    float           closePrice      // Original close price
```

### TwoCandleData

Contains analysis results for two-candle patterns:

```pine
type TwoCandleData
    TwoCandlePattern pattern        // Identified two-candle pattern
    CandleData       candle1        // Analysis data for first (older) candle
    CandleData       candle2        // Analysis data for second (newer) candle
```

## Core Library Functions

### CPL_analyzeCandle()

Main analysis function that examines a single candle and returns comprehensive pattern data.

**Parameters:**
- `_open` (float): Candle open price
- `_high` (float): Candle high price
- `_low` (float): Candle low price
- `_close` (float): Candle close price
- `_avgSize` (float): Average candle size for comparison (from ATR or manual setting)
- `_sizeThresholdPct` (float): Percentage threshold for size classification
- `_equivTolerance` (float): Absolute equivalence tolerance for Marubozu/Doji detection
- `_bodyTolerance` (float): Absolute body tolerance for small body detection
- `_positionThreshold` (int): Position threshold (75-99) for hammer/shooting star classification

**Returns:** `CandleData` - Complete candle analysis

**Example:**
```pine
CandleData analysis = CPL_analyzeCandle(
    open, high, low, close,
    avgSize,
    30.0,           // size threshold %
    0.01,           // equivalence tolerance
    0.05,           // body tolerance
    75              // position threshold
)
```

### CPL_analyzeTwoCandlePattern()

Analyzes two consecutive candles for two-candle patterns.

**Parameters:**
- `_candle1` (CandleData): Analysis data for first (older) candle
- `_candle2` (CandleData): Analysis data for second (newer) candle
- `_equivTolerance` (float): Absolute equivalence tolerance

**Returns:** `TwoCandleData` - Complete two-candle pattern analysis

**Example:**
```pine
TwoCandleData twoCandle = CPL_analyzeTwoCandlePattern(
    prevCandleAnalysis,
    currentCandleAnalysis,
    0.01
)
```

### Helper Functions

**CPL_getPatternName()** - Returns human-readable name for a CandlePattern enum
**CPL_getTwoCandlePatternName()** - Returns human-readable name for a TwoCandlePattern enum
**CPL_getSizeName()** - Returns human-readable name for a CandleSize enum
**CPL_getDirectionName()** - Returns human-readable name for a CandleDirection enum

## Configuration Parameters

### Size Classification

Candles are classified based on their wick range compared to an average size:

- **Average Size Source**: Either ATR-based (dynamic) or manual (static)
- **Size Threshold**: Percentage deviation from average
  - **Long**: Wick range > average × (1 + threshold%)
  - **Short**: Wick range < average × (1 - threshold%)
  - **Normal**: Everything else

### Tolerance Settings

**Equivalence Tolerance** (absolute price value):
- Used for testing if prices are "equivalent" (e.g., open ≈ close for Doji)
- Used for Marubozu detection (open ≈ high/low, close ≈ high/low)
- Used for Tweezer pattern high/low matching
- Default: 0.01

**Body Tolerance** (absolute price value):
- Used for determining "small body" patterns (Hammer, Shooting Star, Spinning Top)
- Less strict than equivalence tolerance
- Default: 0.05

### Position Thresholds

Controls where the candle body must be positioned to qualify as Hammer/Shooting Star:

| Setting | Upper % | Lower % | Description |
|---------|---------|---------|-------------|
| 99:1    | 99      | 1       | Very strict - body must be at extreme ends |
| 75:25   | 75      | 25      | Moderate - body in top/bottom 25% |
| 50:50   | 50      | 50      | Lenient - body just needs to be in upper/lower half |

- **Hammer**: Body position ≥ upper threshold
- **Shooting Star**: Body position ≤ lower threshold
- **Spinning Top**: Body in middle section

## Pattern Detection Logic

### Detection Priority

Patterns are detected in priority order (first match wins):

1. **4 Price Doji** - All prices equal (rare, special case)
2. **Dragonfly Doji** - Body at the high
3. **Gravestone Doji** - Body at the low
4. **Cross Doji** - Body in upper section (normal/long candle)
5. **Inverted Cross Doji** - Body in lower section (normal/long candle)
6. **Long Legged Doji** - Body in middle (normal/long candle)
7. **Regular Doji** - Short candle with equivalent body
8. **Bullish Marubozu** - Open ≈ low, close ≈ high
9. **Bearish Marubozu** - Open ≈ high, close ≈ low
10. **Hammer/Shooting Star/Spinning Top** - Small body patterns by position
11. **Regular Bullish/Bearish** - Standard directional candles

### Key Detection Concepts

**Equivalence Testing:**
- Two prices are "equivalent" if their absolute difference ≤ tolerance
- Example: `|open - close| <= equivTolerance` → Doji candidate

**Small Body Detection:**
- Body is "small" if `|open - close| <= bodyTolerance`
- Body is not "equivalent" (stricter test must fail)
- Candle must be Normal or Long size

**Body Position Calculation:**
- Position = (bodyLow - candleLow) / wickRange
- Returns percentage (0.0 to 1.0) of where body sits in wick range
- 0.0 = body at low, 1.0 = body at high

## Usage Example

```pine
//@version=6
indicator("My Strategy with CPL", overlay=true)

// Configuration
i_useAtr = input.bool(false, "Use ATR")
i_atrLength = input.int(14, "ATR Length")
i_avgSize = input.float(1.0, "Average Size")
i_sizeThreshold = input.float(30.0, "Size Threshold %")
i_equivTolerance = input.float(0.01, "Equivalence Tolerance")
i_bodyTolerance = input.float(0.05, "Body Tolerance")
i_positionThreshold = 75

// Calculate average size
atrValue = ta.atr(i_atrLength)
avgSize = i_useAtr ? atrValue : i_avgSize

// Analyze current candle
CandleData currentCandle = CPL_analyzeCandle(
    open, high, low, close,
    avgSize,
    i_sizeThreshold,
    i_equivTolerance,
    i_bodyTolerance,
    i_positionThreshold
)

// Analyze previous candle
CandleData prevCandle = CPL_analyzeCandle(
    open[1], high[1], low[1], close[1],
    avgSize,
    i_sizeThreshold,
    i_equivTolerance,
    i_bodyTolerance,
    i_positionThreshold
)

// Check for two-candle patterns
TwoCandleData twoCandlePattern = CPL_analyzeTwoCandlePattern(
    prevCandle,
    currentCandle,
    i_equivTolerance
)

// Use the results
if barstate.isconfirmed
    if currentCandle.pattern == CandlePattern.Hammer
        // Bullish reversal signal
        strategy.entry("Long", strategy.long)
    
    if twoCandlePattern.pattern == TwoCandlePattern.BullishEngulfingStrong
        // Strong bullish signal
        strategy.entry("Long", strategy.long)
```

## Extraction Process

When extracting this library from the indicator:

1. **Copy the Library Section**: Extract all functions prefixed with `CPL_`
2. **Copy the UDT Definitions**: Extract `CandleData`, `TwoCandleData`, and all enum types
3. **Remove Debug Code**: Remove all plotting, barcolor, and log statements
4. **Update Function Headers**: Add proper Pine Script library export annotations
5. **Create Library File**: Save as a `.pine` library file
6. **Import in Indicators**: Use `import` statement to include the library

**Example library header:**
```pine
//@version=6
library("CandlestickPatternLibrary", overlay=true)

// Export types
export CandleSize
export CandleDirection
export CandlePattern
export TwoCandlePattern
export CandleData
export TwoCandleData

// Export functions
export CPL_analyzeCandle
export CPL_analyzeTwoCandlePattern
export CPL_getPatternName
export CPL_getTwoCandlePatternName
export CPL_getSizeName
export CPL_getDirectionName
```

## Testing Recommendations

1. **Test Across Timeframes**: Verify pattern detection works on 1m, 5m, 1h, 1D charts
2. **Test Different Instruments**: Stocks, forex, crypto have different price characteristics
3. **Adjust Tolerances**: Fine-tune equivalence and body tolerances for your use case
4. **Visual Verification**: Use debug mode to visually confirm patterns are correctly identified
5. **Historical Validation**: Check that patterns match well-known historical examples

## Version History

- **v1.2.0** (Current) - Complete single and two-candle pattern detection with configurable tolerances
- Initial implementation with comprehensive Doji variations and small body patterns

## License

This library is designed for personal and educational use within TradingView.
