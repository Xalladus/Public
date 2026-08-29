# Volume Profile Guide

This internal guide explains how `VolumeProfileLibrary` divides ownership, processes profile data, resolves drawing budgets, and retains recurring-session history.

## Architecture and Ownership

`VolumeProfileLibrary` is a stateful volume-distribution engine with an embedded renderer. It supports fixed and recurring ranges, box and polyline displays, Point of Control (POC), Value Area High (VAH), Value Area Low (VAL), retained session history, and optional lower-timeframe accumulation.

The consumer owns:

- User inputs and conversion of input-library enums.
- Persistent `ProfileState`, `ProfileStyle`, and `ProfileConfig` declarations.
- The `indicator()` or `strategy()` drawing-limit declarations.
- Missing-volume validation.
- The global `request.security_lower_tf()` call and its four returned arrays.
- Any plots, tables, alerts, or strategy logic that consume the calculated levels.

The library owns:

- Configuration sanitization.
- Fixed and recurring range membership.
- Profile creation, accumulation, and completion.
- Volume distribution into price rows.
- POC and Value Area calculations.
- Drawing-budget resolution in whole sessions.
- Full and level-only history tiers.
- Range boxes, profile bodies, level lines, labels, and level extensions.

The per-bar flow is:

```text
Consumer inputs and intrabar arrays
    -> sanitize configuration
    -> resolve range membership
    -> close or open a profile
    -> accumulate and distribute volume
    -> calculate POC, VAH, and VAL
    -> resolve drawing capacity
    -> archive or demote completed sessions
    -> render on the last bar
    -> expose retained levels and diagnostics
```

## State and Configuration

### `ProfileState`

Create the state once with `VP.createState()` and retain it with `var`. The state contains:

- `active`: The open profile, or the retained completed profile in a fixed mode.
- `history`: Completed recurring sessions that retain their full profile drawings.
- `levelHistory`: Older recurring sessions that retain their requested range box and level drawings but no row body.
- `effectiveHistoryCount`: Whole completed-session capacity for full profile drawings.
- `levelOnlyHistoryCount`: Requested completed sessions assigned to the lower tier.
- `resolvedFillBands`: Polyline ribbons per region: `0`, `1`, or `3-8`.
- Retained POC, VAH, VAL, bar count, and total-volume values.

The consumer should read diagnostic and latest-value fields but should not mutate the engine-owned lifecycle fields or `Profile` objects.

### `ProfileConfig`

`ProfileConfig` combines range, calculation, display, history, and appearance settings. Important relationships include:

- `rowCount` drives the statistics and every renderer.
- `valueAreaPercent` controls the target volume share used for VAH and VAL.
- `splitBuySell` changes the number of box rows or polyline regions per profile.
- `showVisuals` hides chart objects without stopping calculations.
- `profileDisplay` selects boxes, straight polylines, or curved polylines.
- `polylineFill` selects no fill, a solid fill, or an adaptive gradient.
- `historyCount` requests completed recurring sessions. The active session is additional.
- `showRangeBox` reserves and draws a range box for every represented session.
- POC and Value Area lines and labels have separate visibility and style settings.
- `extendLevelsExtraSession` extends completed recurring levels through their successor.

The shared `MAX_ROW_COUNT` is `49`. This is a library design limit shared by boxes and polylines so one Rows setting controls the complete profile. It is not the point limit of an individual Pine polyline.

## Processing Pipeline

### 1. Range Resolution

`resolveRangeMembership()` determines whether the current chart bar belongs to the configured range and returns its anchor key.

- `FromTime` creates one profile from an absolute start timestamp onward.
- `BetweenTimes` creates one profile between two absolute timestamps.
- `DailyAnchor` starts a new recurring profile at the configured time each day.
- `DailySession` accumulates only during the configured recurring session.

Fixed `input.time` values are absolute UNIX timestamps and pass through unchanged. The configured timezone applies to daily anchors, daily sessions, and level-label dates.

### 2. Volume Accumulation

`appendBarsToProfile()` accepts either the lower-timeframe arrays supplied by the consumer or fallback chart-bar OHLCV data.

Each source entry stores its high, low, volume, and estimated buy volume. `distributeEntryIntoRows()` assigns volume proportionally to every overlapping price row. A flat source bar distributes evenly across its intersected rows and uses a 50% buy-volume estimate.

Row boundaries depend on the complete profile high and low. When a new entry expands that range, `rebuildRows()` clears the row arrays and replays the stored entries against the new boundaries. Entries that remain inside the existing range can be added incrementally.

### 3. Statistics

`computeStatistics()` identifies the row with the greatest volume as the POC. Starting from that row, it compares the next row above and below and adds the larger one until the configured Value Area percentage is reached.

The active or latest profile values are retained in `ProfileState`, allowing consumers to read them during gaps between recurring sessions.

### 4. Drawing-Plan Resolution

`resolveDrawingPlan()` converts Pine's drawing limits into whole-profile capacity.

For box displays:

- One range box is reserved for the active profile and every requested completed session when range boxes are enabled.
- The remaining box capacity is divided by `rowCount` and by one or two sides depending on `splitBuySell`.
- The result is floored before the active-profile slot is removed, so `effectiveHistoryCount` is always a whole completed-session count.

For polyline displays:

- Each unsplit profile uses one outline; each split profile uses two.
- No Fill adds no ribbons per region.
- Solid adds one ribbon per region.
- Gradient begins at eight ribbons per region and steps down to three when necessary.
- Full sessions are reduced only after the gradient reaches its three-band minimum.

The range-box reservation is independent of the selected body renderer. This guarantees that every represented session retains its range box when the user enables it.

### 5. History Lifecycle

Completed recurring profiles move through two tiers:

1. `history` retains the newest sessions with their full row boxes or polylines, range box, and selected levels.
2. `levelHistory` retains the remaining requested sessions with their range box and selected level drawings only.

When a profile completes, its source-entry arrays are cleared because its price range can no longer expand. When it moves to `levelHistory`, its row drawings and row arrays are also cleared. This prevents long intrabar sessions and large requested histories from retaining unnecessary data.

Fixed profiles do not rotate through recurring history. A completed fixed profile remains in `active` so its values and drawings remain available.

### 6. Rendering and Level Extensions

`renderProfile()` treats the profile body and its levels separately:

- The body contains the range box plus box rows or polylines.
- The levels contain the POC, VAH, and VAL lines and their optional labels.
- Disabling `showVisuals` deletes both parts while calculations continue.

When `extendLevelsExtraSession` is enabled, a completed recurring profile's selected levels end at its successor's end time. Until a successor exists, the newest completed profile follows the latest bar. The extension refresh runs on the last bar because the endpoints depend on the current session chain.

## Consumer Integration

The consumer must declare the full drawing budgets used by the engine:

```pine
indicator("Volume Profile Consumer", overlay = true,
    max_boxes_count = 500, max_lines_count = 500,
    max_labels_count = 500, max_polylines_count = 100)
```

The library intentionally leaves the lower-timeframe request in the consumer. Request the chart timeframe at or below one minute to avoid requesting a timeframe that is not lower than the chart:

```pine
import OneCleverGuy/VolumeProfileLibraryTESTB/3 as VP

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

If lower-timeframe use is disabled or the arrays contain no entries for a chart bar, `update()` falls back to that chart bar's OHLCV data. The consumer should still stop or suppress execution on symbols that provide no usable volume.

## Operational Rules

- Call `VP.update()` exactly once per bar from global scope.
- Store `ProfileState`, `ProfileStyle`, and `ProfileConfig` with `var`.
- Pass all four lower-timeframe arrays directly to `update()`.
- Declare 500 boxes, 500 lines, 500 labels, and 100 polylines in the consumer.
- Treat `historyCount` as completed recurring sessions; the active session is not included.
- Use `effectiveHistoryCount/historyCount` for the whole-session drawing-plan diagnostic.
- Do not expose `resolvedFillBands` as a user-facing statistic unless it is needed for debugging.
- Do not construct or mutate exported `Profile` instances in consumer code.

## Summary

The consumer supplies configuration and data. The library owns the state machine, volume distribution, statistics, drawing allocation, history lifecycle, and rendering. This boundary keeps integrations small while ensuring that range boxes, profile bodies, and levels remain within Pine's object budgets.
