# AGENTS.md - Indicators/Fixed-Range-Volume-Profile

## Overview
An official indicator utilizing the `VolumeProfileLibrary` to display highly customizable, fixed-range and session-based volume profiles. 
The indicator owns all user inputs (colors, modes, limits), timezone conversions, and the `request.security_lower_tf()` call for intrabar accuracy. It delegates the actual volume accumulation and drawing orchestration to the imported library.

## Dependencies

| Library | Alias | Role |
|---------|-------|------|
| `OneCleverGuy/VolumeProfileLibraryTESTA/8` | `VP` | Core engine handling volume accumulation, stats math, and chart rendering. |
| `OneCleverGuy/InputLibrary/4` | `iLib` | UI enums, timezone mappings, and formatting helpers. |

## Input Groups

| Group | Purpose |
|-------|---------|
| `Timezone` | Controls the timezone for time pickers, recurring anchors, and POC labels. |
| `Range Mode` | Selects between Absolute (From Time, Between Times) and Recurring (Daily Anchor, Daily Session). |
| `Fixed Anchors` | Standard time inputs for absolute range boundaries. |
| `Recurring Anchors` | Session windows and daily reset triggers, along with historical retention counts. |
| `Profile` | Resolution, value area size, a master visual toggle, and visual style (Boxes vs. Polylines). |
| `Appearance` | Color pickers, gradients, POC line settings, and range box highlights. |
| `Debug` | Toggles an on-chart statistics table. |

## Phases

1. **Setup / Cache update**: Initializes library state (`VP.createState()`) and resolves user inputs into `ProfileConfig` and `ProfileStyle`.
2. **Data Fetching**: Requests intrabar data (1-minute timeframe by default) via `request.security_lower_tf()`.
3. **Core Engine**: Passes the arrays to `VP.update()` to distribute volume and manage drawings.
4. **Data Exposure**: Fetches the latest POC and Value Area boundaries, plotting them invisibly so the Data Window and downstream consumers can access them.
5. **Rendering**: Optionally draws a debug statistics table on the last bar.

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Ownership boundary | The indicator is responsible for declaring `max_boxes_count` and `max_polylines_count` in the `indicator()` declaration because it delegates drawing to the library. |
| Intrabar Fetching | The indicator must manually resolve the intrabar timeframe and call `request.security_lower_tf()` at the global scope, since libraries cannot safely loop or branch on `request.*` calls. |
| Invisible Plots | The `plot()` calls at the end use `display = display.data_window` so external consumers can read the POC/VA levels externally without cluttering the chart. |
