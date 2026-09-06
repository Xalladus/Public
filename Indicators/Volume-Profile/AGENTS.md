# AGENTS.md - Indicators/Volume-Profile

## Overview

Pine Script v6 example consumer for fixed and recurring volume profiles. The indicator owns user inputs, recurring-timezone conversion, fixed timestamp handoff, lower-timeframe requests, style/config assembly, missing-volume validation, Data Window plots, and the optional statistics table. `VolumeProfileLibrary` owns range lifecycle, volume/statistics calculations, history management, drawing-budget resolution, and chart-object rendering.

## Dependencies

| Library | Alias | Role |
|---------|-------|------|
| `OneCleverGuy/InputLibrary/4` | `iLib` | Timezone, quarter-hour, line-style, line-size, and text-size enums/conversions. |
| `OneCleverGuy/VolumeProfileLibraryTESTB/3` | `VP` | Profile state, accumulation, statistics, drawing plans, history, and rendering. |

## Input Groups

| Group | Purpose |
|-------|---------|
| `Timezone` | Recurring anchors, recurring sessions, and level-label dates. Fixed pickers remain absolute chart-timezone timestamps. |
| `Range Mode` | Selects `FromTime`, `BetweenTimes`, `DailyAnchor`, or `DailySession`. |
| `Fixed Anchors` | Absolute Range Start and Range End pickers for fixed modes. |
| `Recurring Anchors` | Daily anchor/session controls and requested previous sessions. |
| `Profile` | Value Area %, Rows, and Width %. Rows drives statistics and both renderers. |
| `Display` | Master visual toggle, renderer, box outside-VA dimming, and polyline fill. |
| `Appearance` | Buy/sell split, colors, range box, POC/VA lines and labels, and level extension. |
| `Debug` | Optional last-bar statistics table. |

## Phases

1. **Input resolution**: Resolve recurring timezone and daily-anchor HHMM values; pass fixed timestamps unchanged.
2. **Intrabar request**: Request 1-minute arrays above one minute and chart-timeframe arrays at or below one minute.
3. **Volume guard**: Track whether any usable volume has appeared and raise on the last bar when none exists.
4. **Config assembly**: Build persistent `ProfileStyle` and `ProfileConfig` objects from inputs.
5. **Core update**: Call `VP.update()` once per bar and read retained POC/VA levels.
6. **Data exposure**: Publish POC, VAH, VAL, POC Buy Volume, and POC Sell Volume only to the Data Window.
7. **Debug rendering**: Write retained statistics and the whole-session drawing plan on the last bar.

## Function Map

### Shared Helpers

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `formatPrice` | `_price` | `string` | Formats a price at symbol tick precision or returns the empty-table marker. |

### Debug Table

| Function | Arguments | Returns | Meaning |
|----------|-----------|---------|---------|
| `writeDebugRow` | `_table`, `_row`, `_label`, `_value` | `int` | Writes one styled label/value row. |
| `formatSessionPlan` | `_fullSessions`, `_requestedSessions` | `string` | Formats whole counts as `full/requested`. |
| `renderDebugTable` | `_table`, `_state`, `_cfg` | `int` | Writes Bars, Total Volume, POC Buy Volume, POC Sell Volume, POC, VAH, VAL, and Full Sessions / Requested. |

## Call Hierarchy

```text
FixedRangeVolumeProfile
|
+-- Input Resolution
|   +-- i_timezone.getOfficialTimezone()
|   +-- i_anchorQuarter.toHHMMString()
|
+-- Intrabar Context
|   +-- request.security_lower_tf(...)
|
+-- Config Assembly
|   +-- VP.ProfileStyle.new(...)
|   +-- VP.ProfileConfig.new(...)
|
+-- Core Orchestration
|   +-- VP.update(...)
|   +-- VP.getMostRecentLevels(...)
|   +-- VP.getMostRecentPocVolumes(...)
|   +-- plot(...) -> Data Window only
|
+-- Debug
    +-- renderDebugTable(...)
        +-- formatPrice(...)
        +-- formatSessionPlan(...)
        +-- writeDebugRow(...)
```

## Runtime State

| State | Type | Ownership |
|-------|------|-----------|
| `profileState` | `VP.ProfileState` | Persistent library state created once and reassigned from `VP.update()`. |
| `profileStyle` | `VP.ProfileStyle` | Persistent style assembled from Appearance inputs. |
| `profileConfig` | `VP.ProfileConfig` | Persistent engine config assembled from all input groups. |
| `hasSeenVolume` | `bool` | Indicator-owned guard retaining whether positive volume has appeared. |
| `debugTable` | `table` | Last-bar table created only when Debug is enabled. |

## Rules And Pitfalls

| Rule | Detail |
|------|--------|
| Ownership boundary | Keep accumulation, profile lifecycle, drawing-budget math, and chart-object rendering in the library. |
| Fixed timestamps | `input.time` returns absolute UNIX timestamps. Assign Range Start/End directly without timezone reinterpretation. |
| Recurring timezone | Apply `i_timezone` only to Daily Anchor, Daily Session, and level-label dates. |
| Global intrabar request | `request.security_lower_tf()` remains at indicator global scope and its four arrays pass directly to `VP.update()`. |
| Low-timeframe guard | Use the chart timeframe at or below one minute to avoid requesting a non-lower timeframe. |
| Drawing declarations | Keep the indicator declaration at 500 boxes, 100 polylines, 500 lines, and 500 labels. |
| Persistent config | `profileStyle` and `profileConfig` use `var`; input changes restart the script and rebuild them. |
| Input layout | Keep input calls compact. Align tooltip continuation operators at or beyond the tooltip declaration `=` column. Within a shared `inline` row, align each `group` argument and leave one blank line after the inline block. Name input-activity gates `a_` plus a three-letter abbreviation, align their declaration `=` with the surrounding input declarations, define them immediately after their parent inputs, and reuse them for dependent controls. Do not wrap only one or two trailing arguments onto a new line; when wrapping is necessary, indent continuation content beyond the declaration's `=`. |
| Input order | Profile is Value Area %, Rows, Width %; Display begins with Show Profile Visuals; Appearance begins with Split Buy/Sell. |
| Invisible outputs | Keep the latest-level and POC volume `plot()` calls at global scope with `display.data_window`. |
| Whole session diagnostics | Display `_state.effectiveHistoryCount/_cfg.historyCount`; do not expose fractional capacity or gradient-band counts in the table. |
| Publication preservation | Add release/update sections to `publication-document.txt`; do not replace its existing description. |
