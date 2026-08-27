# Volume Profile Guide

This guide maps the underlying architecture and execution pipeline of the `VolumeProfileLibrary` to explain **how** the components fit together and **why** it handles state the way it does.

## Core Philosophy & Data Flow

The `VolumeProfileLibrary` operates as a **stateful accumulation engine** bundled with an **embedded renderer**. 

Unlike stateless mathematical helpers, the library requires persistent state (via `ProfileState`) to retain multiple price rows, drawings, and past session history. However, it still enforces a strict **separation of concerns** by forcing the consumer to own user inputs, chart limit boundaries, and data fetching (such as lower timeframe arrays).

The pipeline follows three major steps on every bar: **Configure -> Accumulate -> Orchestrate Drawings**.

## Pipeline Stages

### 1. Configure and Initialize (State Management)
To maintain performance and keep drawings alive, the consumer establishes a persistent environment:
- **`createState()`**: Generates an empty `ProfileState` that holds the currently active profile and an array of historical completed profiles.
- **`ProfileConfig`**: Gathers all rules about when profiles start/stop (e.g. `RangeMode`), styling instructions (`ProfileStyle`), and resolution fidelity (`rowCount`, `gradientBandsPerSide`).

*Beneath the surface:* The state object serves as a ledger. Because drawing limits in Pine Script can easily be exhausted, the state is responsible for archiving completed sessions into history and stripping visuals from old ones to free up limits.

### 2. Volume Accumulation
Triggered on every bar by `update()`, the engine resolves if the current chart bar belongs inside an active profile based on the range mode (`FromTime`, `BetweenTimes`, `DailyAnchor`, `DailySession`).
- **`appendBarsToProfile()`**: Accepts either the fallback chart-bar OHLCV or the highly accurate arrays provided by `request.security_lower_tf()`.
- **`distributeEntryIntoRows()`**: Distributes the volume data proportionally across the constructed price rows (`rowVolumes` and `rowBuyVolumes`). 
- **`rebuildRows()`**: If an incoming candle expands the price boundaries of the profile beyond what was already seen, the library must recalibrate the price row heights and fully rebuild the rows by replaying the stored entry history.

### 3. Statistics and Rendering
Once data is accumulated, the engine resolves actionable analytics and handles the rendering payload.
- **`computeStatistics()`**: Walks outward from the Point of Control (POC) to compute the Value Area (VA) by accumulating the fattest price rows until `valueAreaPercent` is reached.
- **`renderProfile()`**: Removes all profile drawings when `ProfileConfig.showVisuals` is false; otherwise dispatches to the selected box or polyline renderer.
- **Level Exposure**: By calling `getMostRecentLevels()`, the consumer can pull the calculated POC and VA boundaries out of the library state to use for external analysis (e.g., plotting, triggering webhook alerts, etc.).

## Critical Rules / Execution Patterns

### Intrabar Data Fetching
Pine Script does not allow `request.*` function calls inside loops or complex conditional library blocks securely. **Lower timeframe data must be fetched manually by the consumer** and passed directly into `update()`.

```pine
// Correct Pattern:
string intrabarTimeframe = (timeframe.in_seconds() > 60 ? "1" : timeframe.period)
[ltfHighs, ltfLows, ltfCloses, ltfVolumes] = request.security_lower_tf(syminfo.tickerid, intrabarTimeframe, [high, low, close, volume])
profileState := VP.update(profileState, profileConfig, true, ltfHighs, ltfLows, ltfCloses, ltfVolumes)
```

### Maximum Objects Budgeting
Because the engine uses native Pine boxes, lines, and polylines, it is heavily susceptible to limits. Consumers **must** increase object counts in their `indicator()` or `strategy()` declaration. For example, 49 split rows × 5 profiles = 490 boxes. Declare `max_boxes_count = 500` to prevent chart glitches.

## Conclusion

1. **Consumer-Driven Data**: The consumer handles inputs, chart settings, and fetches intrabar data, passing them down into the engine to keep the runtime clean.
2. **Stateful Processing**: The library holds its own state, tracking arrays of volume data per row and managing the recycling of older historical drawings.
3. **Internal Orchestration**: Rendering logic for complex gradient polylines and stacked boxes is cleanly abstracted, freeing the consumer script to focus on logic and signals.
