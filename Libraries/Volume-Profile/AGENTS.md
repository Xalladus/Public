# AGENTS.md - Libraries/Volume-Profile

Pine Script v6 state, calculation, and rendering library for fixed and recurring volume profiles. The library owns range membership, volume distribution, POC/Value Area statistics, drawing-budget resolution, history lifecycle, and chart objects. Consumers own inputs, persistent state/config declarations, drawing limits, missing-volume handling, and lower-timeframe requests. Published as:

```pine
import OneCleverGuy/VolumeProfileLibraryTESTB/3 as VP
```

The library has no imported dependencies. A consumer may use `InputLibrary` to build compatible enum, timezone, line-style, and text-size values.

---

## Public API

```pine
VP.createState() -> ProfileState
VP.getMostRecentPoc(ProfileState _state) -> float
VP.getMostRecentLevels(ProfileState _state) -> [float, float, float]
VP.update(ProfileState _state, ProfileConfig _cfg, bool _useIntrabar,
    array<float> _ltfHighs, array<float> _ltfLows,
    array<float> _ltfCloses, array<float> _ltfVolumes) -> ProfileState
```

Exported limits:

| Constant | Value | Meaning |
|----------|------:|---------|
| `MIN_ROW_COUNT` | `1` | Minimum accepted price-row count. |
| `MAX_ROW_COUNT` | `49` | Maximum shared row count for box and polyline renderers. |
| `MAX_HISTORY_COUNT` | `100` | Maximum requested completed-session history. |

---

## Exported Enums

### `RangeMode`

| Value | Meaning |
|-------|---------|
| `FromTime` | One profile from an absolute timestamp through the current bar. |
| `BetweenTimes` | One profile between two absolute timestamps. |
| `DailyAnchor` | Recurring profile that resets at a configured HHMM time. |
| `DailySession` | Recurring profile active only inside a configured session. |

### `ProfileDisplay`

| Value | Meaning |
|-------|---------|
| `Boxes` | Stacked row boxes using total volume or split buy/sell volume. |
| `Polylines` | Straight contours through row endpoints. |
| `CurvedPolylines` | Curved contours through the same endpoints. |

### `PolylineFill`

| Value | Meaning |
|-------|---------|
| `None` | Draw contour lines without a region fill. |
| `Solid` | Draw one filled ribbon per region. |
| `Gradient` | Draw three to eight adaptive ribbons per region. |

---

## Exported Types

### `ProfileStyle`

Visual styling passed through `ProfileConfig.style`.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `buyColor` | `color` | `#26a69a` | Estimated buy-volume color. |
| `sellColor` | `color` | `#ef5350` | Estimated sell-volume color. |
| `totalColor` | `color` | `#2962ff` | Unsplit total-volume color. |
| `outsideValueAreaTransparency` | `int` | `75` | Transparency for box rows outside the Value Area. |
| `rangeBackgroundColor` | `color` | `#2962ff12` | Range-box fill. |
| `pocColor` | `color` | `#ff0000` | POC line color. |
| `pocWidth` | `int` | `2` | POC line width. |
| `pocStyle` | `string` | `line.style_solid` | POC line style. |
| `pocTextColor` | `color` | `#ff0000` | POC-label text color. |
| `pocTextSize` | `string` | `size.small` | POC-label text size. |
| `valueAreaColor` | `color` | `#2962ff` | VAH/VAL line color. |
| `valueAreaWidth` | `int` | `1` | VAH/VAL line width. |
| `valueAreaStyle` | `string` | `line.style_solid` | VAH/VAL line style. |
| `valueAreaTextColor` | `color` | `#2962ff` | VAH/VAL-label text color. |
| `valueAreaTextSize` | `string` | `size.small` | VAH/VAL-label text size. |

### `ProfileConfig`

Consumer-owned configuration sanitized in place by `update()`.

| Field | Type | Default | Meaning |
|-------|------|---------|---------|
| `rangeMode` | `RangeMode` | `DailySession` | Range lifecycle. |
| `rangeStartTime` | `int` | `na` | Absolute start for fixed modes. |
| `rangeEndTime` | `int` | `na` | Absolute end for `BetweenTimes`. |
| `anchorHHMM` | `string` | `"1600"` | Daily-anchor reset time. |
| `sessionString` | `string` | `"0930-1600"` | Daily-session window. |
| `timezone` | `string` | `"America/New_York"` | Timezone for recurring ranges and level-label dates. |
| `rowCount` | `int` | `30` | Rows used by statistics and both renderers. |
| `valueAreaPercent` | `float` | `70.0` | Target share of volume in the Value Area. |
| `splitBuySell` | `bool` | `true` | Split each profile into estimated buy/sell regions. |
| `showVisuals` | `bool` | `true` | Draw chart objects without disabling calculations. |
| `widthPercent` | `float` | `100.0` | Maximum row width as a share of the profile range width. |
| `profileDisplay` | `ProfileDisplay` | `Boxes` | Active renderer. |
| `polylineFill` | `PolylineFill` | `Gradient` | Polyline region-fill mode. |
| `historyCount` | `int` | `2` | Requested completed recurring sessions. |
| `showRangeBox` | `bool` | `true` | Draw each represented session's range box. |
| `dimOutsideValueArea` | `bool` | `true` | Apply outside-VA transparency to box rows. |
| `showPoc` | `bool` | `true` | Draw POC lines. |
| `showPocLabel` | `bool` | `true` | Draw POC origin labels. |
| `showValueAreaLines` | `bool` | `false` | Draw VAH and VAL lines. |
| `showValueAreaLabels` | `bool` | `true` | Draw VAH and VAL origin labels. |
| `extendLevelsExtraSession` | `bool` | `false` | Extend completed recurring levels through the next session. |
| `style` | `ProfileStyle` | `na` | Visual styling; replaced with defaults when `na`. |

### `ProfileState`

Persistent engine state. Declare once with `var`; read diagnostic and latest-value fields but let the library mutate lifecycle fields.

| Field | Type | Meaning |
|-------|------|---------|
| `active` | `Profile` | Open profile, or the retained completed fixed profile. |
| `history` | `array<Profile>` | Fully drawn completed recurring profiles. |
| `levelHistory` | `array<Profile>` | Lower-tier profiles retaining range/level drawings. |
| `effectiveHistoryCount` | `int` | Whole completed-session capacity for full drawings. |
| `levelOnlyHistoryCount` | `int` | Requested history remaining in the lower tier. |
| `resolvedFillBands` | `int` | Resolved ribbons per region: `0`, `1`, or `3-8`. |
| `lastAnchorKey` | `int` | Most recent in-range recurring anchor. |
| `anchorMinute` | `int` | Cached Daily Anchor minute. |
| `sessionStartMinute` | `int` | Cached Daily Session start minute. |
| `sessionEndMinute` | `int` | Cached Daily Session end minute. |
| `mostRecentPoc` | `float` | Latest POC retained across session gaps. |
| `mostRecentValueAreaHigh` | `float` | Latest VAH retained across session gaps. |
| `mostRecentValueAreaLow` | `float` | Latest VAL retained across session gaps. |
| `mostRecentBarCount` | `int` | Bar count of the active or latest profile. |
| `mostRecentTotalVolume` | `float` | Total volume of the active or latest profile. |

### `Profile`

Exported because it is stored inside `ProfileState`, but operationally engine-owned. It contains timestamps, source-entry arrays, distributed row arrays, calculated POC/VA values, completion state, and all drawing IDs. Consumers should not construct or mutate it directly.

---

## Function Reference

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `createState` | None | `ProfileState` | Creates empty persistent history arrays and state. |
| `getMostRecentPoc` | `_state` | `float` | Returns the retained latest POC, or `na` before calculation. |
| `getMostRecentLevels` | `_state` | `[float, float, float]` | Returns retained POC, VAH, and VAL. |
| `update` | `_state`, `_cfg`, `_useIntrabar`, `_ltfHighs`, `_ltfLows`, `_ltfCloses`, `_ltfVolumes` | `ProfileState` | Sanitizes config, advances range/profile state, accumulates volume, computes statistics, resolves budgets, and renders. Mutates `_state` and `_cfg`. |

---

## Function Hierarchy

```text
VP
|
+-- Range Resolution
|   +-- ensureSessionCache(...)
|   +-- resolveRangeMembership(...)
|
+-- Volume And Statistics
|   +-- createProfile(...)
|   +-- appendBarsToProfile(...)
|   |   +-- estimateBuyVolume(...)
|   |   +-- rebuildRows(...)
|   |       +-- distributeEntryIntoRows(...)
|   +-- computeStatistics(...)
|
+-- Drawing And History
|   +-- resolveDrawingPlan(...)
|   +-- archiveCompletedProfile(...)
|   |   +-- demoteToLevelHistory(...)
|   +-- renderProfile(...)
|       +-- renderRangeBox(...)
|       +-- renderVolumeProfile(...)
|       |   +-- renderRowBoxes(...)
|       |   +-- renderProfilePolylines(...)
|       +-- renderProfileLevels(...)
|   +-- refreshLevelExtensions(...)
|
+-- Public Lifecycle
    +-- createState() -> ProfileState
    +-- update(...) -> ProfileState
    +-- getMostRecentPoc(...) -> float
    +-- getMostRecentLevels(...) -> [float, float, float]
```

---

## Standard Integration Pattern

```pine
import OneCleverGuy/VolumeProfileLibraryTESTB/3 as VP

indicator("Volume Profile Consumer", overlay = true, max_boxes_count = 500,
    max_lines_count = 500, max_labels_count = 500, max_polylines_count = 100)

var VP.ProfileState profileState = VP.createState()
var VP.ProfileStyle profileStyle = VP.ProfileStyle.new()
var VP.ProfileConfig profileConfig = VP.ProfileConfig.new(style = profileStyle)

string intrabarTimeframe = timeframe.in_seconds() > 60 ? "1" : timeframe.period
[ltfHighs, ltfLows, ltfCloses, ltfVolumes] = request.security_lower_tf(
    syminfo.tickerid, intrabarTimeframe, [high, low, close, volume])

profileState := VP.update(
    profileState, profileConfig, not timeframe.isseconds,
    ltfHighs, ltfLows, ltfCloses, ltfVolumes)

[poc, vaHigh, vaLow] = VP.getMostRecentLevels(profileState)
```

---

## Execution Ownership

1. The consumer resolves inputs, fixed timestamps, recurring timezone values, style/config, volume availability, and intrabar arrays.
2. `update()` sanitizes configuration and resolves the drawing plan in whole sessions.
3. Range membership closes or opens profiles as anchors change.
4. Covered intrabars are accumulated; uncovered bars fall back to chart OHLCV.
5. Statistics update after every accumulated bar.
6. Completed recurring profiles rotate through full and lower history tiers.
7. Active drawings and optional successor-dependent level extensions refresh on the last bar.

---

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Intrabar request ownership | Call `request.security_lower_tf()` in the consumer at global scope and pass all four arrays to `update()`. |
| Persistent objects | Declare `ProfileState`, `ProfileStyle`, and `ProfileConfig` with `var`. Inputs restart the script, so their values remain valid for that execution. |
| Fixed timestamps | Pass `input.time` values unchanged. `timezone` applies only to recurring anchors/sessions and label dates. |
| Drawing declarations | Consumers should declare 500 boxes, 100 polylines, 500 lines, and 500 labels. |
| Range-box priority | When enabled, one range box is reserved for every requested completed session plus the active profile before row-box capacity is calculated. |
| Whole-session capacity | Box and polyline capacities are floored; `effectiveHistoryCount` never represents a fractional session. |
| History semantics | `historyCount` requests completed recurring sessions. The active session consumes drawing budget but is not part of that requested count. |
| Fallback data | When `_useIntrabar` is false or arrays have no coverage, the library distributes chart-bar OHLCV. |
| Volume precondition | The consumer should raise or suppress execution on symbols without usable volume. |
| Engine-owned `Profile` | Do not mutate profile arrays or drawing IDs from consumer code. |
| Typed branch fallbacks | Use a terminal `int(na)` only when terminal branches otherwise resolve to incompatible types. |
