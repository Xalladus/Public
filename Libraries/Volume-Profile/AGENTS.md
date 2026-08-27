# AGENTS.md - Libraries/Volume-Profile

Pine Script v6 compute and rendering library for fixed range volume profiles. The library owns the volume profile accumulation, statistics (POC, Value Area), and drawing orchestration (boxes, polylines, labels). The consumer script owns the user inputs, global state (`var`), and `request.security_lower_tf()` calls for intrabar data accuracy. Published as:

```pine
import OneCleverGuy/VolumeProfileLibraryTESTA/8 as VP
```

---

## Public API

```pine
VP.createState() -> ProfileState
VP.getMostRecentPoc(ProfileState _state) -> float
VP.getMostRecentLevels(ProfileState _state) -> [float, float, float]
VP.update(ProfileState _state, ProfileConfig _cfg, bool _useIntrabar, array<float> _ltfHighs, array<float> _ltfLows, array<float> _ltfCloses, array<float> _ltfVolumes) -> ProfileState
```

---

## Exported Enums

### `RangeMode`

| Value | Meaning |
|-------|---------|
| `FromTime` | One profile from an absolute timestamp to the current bar. |
| `BetweenTimes` | One profile between two absolute timestamps. |
| `DailyAnchor` | Recurring profile that resets every day at a specific HH:MM time. |
| `DailySession` | Recurring profile drawn only inside an HHMM-HHMM session. |

### `ProfileDisplay`

| Value | Meaning |
|-------|---------|
| `Boxes` | Draw rows using stacked boxes (split buy/sell or total volume). |
| `Polylines` | Draw connecting row endpoints with straight lines. |
| `CurvedPolylines` | Draw connecting row endpoints with curved lines. |

---

## Exported Types

### `ProfileStyle`

Styling container for visual elements.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `buyColor` | `color` | `#26a69a` | Row color for estimated buy volume |
| `sellColor` | `color` | `#ef5350` | Row color for estimated sell volume |
| `totalColor` | `color` | `#2962ff` | Row color when buy/sell split is disabled |
| `outsideValueAreaTransparency` | `int` | `75` | Transparency for rows outside the value area |
| `rangeBackgroundColor` | `color` | `#2962ff12`| Fill color for the range highlight box |
| `pocColor` | `color` | `#ff0000` | POC line color |
| `pocWidth` | `int` | `2` | POC line width |
| `pocStyle` | `string` | `line.style_solid`| POC line style |
| `pocTextColor` | `color` | `#ff0000` | POC origin label text color |
| `pocTextSize` | `string` | `size.small` | POC origin label text size |

### `ProfileConfig`

Consumer configuration defining engine limits, time anchors, and styling.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `rangeMode` | `RangeMode` | `DailySession`| Range anchor mode |
| `rangeStartTime` | `int` | `na` | Absolute start timestamp |
| `rangeEndTime` | `int` | `na` | Absolute end timestamp |
| `anchorHHMM` | `string` | `"1600"` | Daily reset time |
| `sessionString` | `string` | `"0930-1600"`| Session window for DailySession |
| `timezone` | `string` | `"America/New_York"`| Timezone for timestamps and labels |
| `rowCount` | `int` | `30` | Number of price rows |
| `valueAreaPercent` | `float` | `70.0` | Value Area volume percentage |
| `splitBuySell` | `bool` | `true` | Show separate buy/sell volume |
| `showVisuals` | `bool` | `true` | Draw profile visuals; calculations continue when disabled |
| `profileDisplay` | `ProfileDisplay`| `Boxes` | Renderer display type |
| `showPolylineGradient` | `bool` | `false` | Fill polyline regions with gradients |
| `gradientBandsPerSide` | `int` | `5` | Number of gradient bands per region |
| `historyCount` | `int` | `2` | Number of completed recurring profiles to retain |

### `ProfileState`

Persistent engine state owned by the consumer via `var`.

| Field | Type | Meaning |
|-------|------|---------|
| `active` | `Profile` | Currently open profile or last completed non-recurring profile |
| `history` | `array<Profile>`| Completed recurring profiles kept alive for drawings |
| `mostRecentPoc` | `float` | Retained latest calculated POC |
| `mostRecentValueAreaHigh`| `float` | Retained latest VA High |
| `mostRecentValueAreaLow` | `float` | Retained latest VA Low |

---

## Function Reference

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `createState` | None | `ProfileState` | Instantiates a fresh `var` state instance for the engine. |
| `getMostRecentPoc` | `_state` | `float` | Fetches the most recently calculated POC, surviving session breaks. |
| `getMostRecentLevels` | `_state` | `[float, float, float]`| Fetches the latest POC, VA High, and VA Low. |
| `update` | `_state`, `_cfg`, `_useIntrabar`, `_ltfHighs`, `_ltfLows`, `_ltfCloses`, `_ltfVolumes` | `ProfileState` | Main engine loop. Run once per bar at global scope. Will handle volume accumulation, statistics generation, and drawing orchestration. |

---

## Function Hierarchy

```text
VP
|
+-- Lifecycle
|   +-- createState() -> ProfileState
|   +-- update(...) -> ProfileState
|       +-- archiveCompletedProfile(...)
|       +-- createProfile(...)
|       +-- appendBarsToProfile(...) -> mutate Profile arrays
|           +-- rebuildRows(...)
|           +-- distributeEntryIntoRows(...)
|       +-- computeStatistics(...)
|       +-- renderProfile(...)
|           +-- renderRangeBox(...)
|           +-- renderVolumeProfile(...)
|           +-- renderPocLine(...)
|
+-- State Access
    +-- getMostRecentPoc(...) -> float
    +-- getMostRecentLevels(...) -> [float, float, float]
```

---

## Standard Integration Pattern

```pine
import OneCleverGuy/VolumeProfileLibraryTESTA/8 as VP

// 1. Declare persistent state
var VP.ProfileState profileState = VP.createState()

// 2. Build config from user inputs
var VP.ProfileStyle profileStyle = VP.ProfileStyle.new()
var VP.ProfileConfig profileConfig = VP.ProfileConfig.new(style = profileStyle)

// 3. Fetch lower-timeframe intrabar volume arrays for accuracy
string intrabarTimeframe = (timeframe.in_seconds() > 60 ? "1" : timeframe.period)
[ltfHighs, ltfLows, ltfCloses, ltfVolumes] = request.security_lower_tf(syminfo.tickerid, intrabarTimeframe, [high, low, close, volume])

// 4. Update engine per bar
profileState := VP.update(profileState, profileConfig, not timeframe.isseconds, ltfHighs, ltfLows, ltfCloses, ltfVolumes)

// 5. Read computed statistics
[poc, vaHigh, vaLow] = VP.getMostRecentLevels(profileState)
```

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Intrabar Fetch Ownership | Consumers *must* perform the `request.security_lower_tf()` call themselves and pass the arrays. The library cannot execute `request.*` internally without locking the execution context. |
| `var` State Declaration | `profileState` and `profileConfig` must be declared using `var` in the consumer script, allowing them to persist across bars and hold historical drawings. |
| Drawing Limits | Chart drawing constraints belong to the consumer indicator (`max_boxes_count`, etc). The library manages its own arrays, but heavy configurations (high `rowCount`, high `historyCount`) may breach limit thresholds if max counts are not set appropriately in the consumer. |
| Volume Data Precondition | The consumer is responsible for skipping calculation if the chart/ticker has no volume data (e.g. guard against `na(volume)`). |
