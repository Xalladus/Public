# Pine Script Utilities Guide

## Introduction

Pine Script Utilities is a Pine v6 library for common representation, price-unit, time, drawing-maintenance and session-state tasks. Its session engine connects these utilities into a reusable lifecycle: resolve a schedule, collect qualifying prices, preserve session identity and expose state to the consumer.

This guide explains why and when to use the components. [AGENTS.md](AGENTS.md) contains the complete signatures, enum values, field definitions and argument reference. [session-example.pine](session-example.pine) demonstrates the larger indicator architecture; its styling, alerts, round levels and retention choices are consumer features.

## Core Philosophy & Data Flow

**Configure -> Resolve boundaries -> Acquire necessary data -> Update state -> Retain records -> Render and consume**

The host declares inputs and owns persistent storage. The library receives configuration and mutates the supplied session states. The host then decides how those states appear, which events matter, and how long records remain available. Each configured session needs its own array; sharing one array between unrelated schedules mixes their lifecycles.

Most utilities are independent of the session engine. You can use enum conversions or price-to-tick arithmetic without creating session state or requesting minute data. The library imports no other libraries. It does expose methods that update or delete existing drawings, but it does not create them.

Two time concepts remain separate throughout the pipeline: **the price-collection interval** and **the lifetime of its displayed levels**. Extending a high line must never extend the period used to calculate that high.

## Stage 1: Configure Meaning and Units

Primary components: `SessionConfig`, the exported enums and their conversion methods, `pipSize`, `priceToTicks`, `roundQuantity`.

Use enums when the host should present a constrained choice, such as a timezone, line style, anchor or clock component. Convert an enum at the boundary where Pine requires its native string or integer. Keep the `input.*` declarations in the host; the library supplies vocabulary rather than a settings panel.

For a custom session, configure an explicit IANA timezone and a string such as `"1600-0400"`. A preset supplies its own schedule and native timezone, so changing the custom timezone field does not move a preset. Select Custom when you want a different local schedule.

Price utilities distinguish price distance, ticks and pips. Tick conversion follows `syminfo.mintick`. The default forex pip is ten minimum ticks, which is a convention rather than a guarantee across feeds. Supply a positive pip-size override when that convention does not describe the symbol. The same override should accompany both directions of conversion.

Quantity rounding and notional conversion remain general utilities. Check point value and quantity units, and supply the broker quantity step when rounding. Default quantity steps are conveniences, not broker specifications. Account-risk sizing and trade risk/reward calculations belong in the separate risk library.

## Stage 2: Resolve the Session Clock

Primary functions: `resolveSessionInfo`, `sessionToParts`, `getSessionStartTime`, `isSessionFirstBar`, `isSessionBoundaryInBar`.

Resolution produces a `SessionInfo` descriptor with parsed opening/closing components, timezone and nominal duration. Cache this descriptor when the configuration is stable so the host can use it for boundary checks.

The engine accepts exactly `HHMM-HHMM`, with four decimal digits per time and valid hour/minute ranges. Invalid strings return `na`. An end earlier than the start denotes an overnight session; equal endpoints denote a 24-hour session. The engine parser does not accept weekday masks or comma-separated periods. `isInSession` and `isInAnySession` instead delegate membership to Pine's native session handling and are a separate interface.

Calendar timestamps preserve the configured local clock across date changes. A nominal session duration is not necessarily its elapsed duration across a daylight-saving transition. Use the resolved session timestamps for lifecycle comparisons instead of adding a fixed 24-hour duration to every date.

Presets describe recurring clock intervals. They do not predict exchange holidays, future closures or split trading periods. For example, the Japan and Hong Kong equity presets describe continuous envelopes that include their midday breaks.

## Stage 3: Acquire Only Necessary Intrabars

Primary functions: `needsSessionIntrabars`, `requestIntrabarData`, `scanIntrabarRange`.

If a session boundary aligns with the chart candle boundary, whole-candle OHLC is sufficient. If the session begins at 09:10 inside a 09:00-09:15 candle, the 09:00-09:09 prices must be excluded. This is why the engine can use one-minute arrays on a boundary candle.

The host checks every enabled session and combines the results with OR. It calls `requestIntrabarData` once and passes the returned arrays to each session engine. **Calling the wrapper every bar does not mean performing the underlying request every bar.** The wrapper requests on eligible intraday charts above one minute when data is required, and also on the last confirmed historical bar to prepare the request context for realtime operation. Otherwise it returns empty arrays.

The scan includes minute opening timestamps at or after the session start and before the exclusive end. Realtime scanning is also limited by the evaluation time. Before 09:10 there is no qualifying data for a 09:10 start; a pending state can therefore have `na` prices without being erroneous.

Missing historical intrabars are a different case. An empty required scan leaves prices unchanged instead of taking a whole candle that contains out-of-session prices. This preserves the boundary rule, but cannot recover unavailable data. A later full candle can produce a usable yet incomplete range. There is no state-level completeness flag; consumers requiring fully verified OHLC must add their own availability policy.

The request's history argument becomes a minute-bar budget, approximately `(historyDays + 3) * 1440`, bounded below by one day's minutes. It is not a promise of that many calendar days or unlimited provider history. Increasing it cannot guarantee missing data becomes available. Custom arrays supplied to the scanner must be chronological and aligned across all five fields.

## Stage 4: Maintain Identity and Prices

Primary functions: `runSessionEngine`, `getActiveSession`, `getCompletedSession`.

Run the engine for each enabled session on every chart bar. It inserts new states at index zero, updates the newest state and retains older records for the consumer. A non-daily session can initialize when loaded history begins midway through its interval; its prices then describe only the available observations. Daily sessions do not use that late-start initialization path.

The scheduled opening timestamp is the record's identity. Duplicate detection prevents a late-start check from adding another record for the same opening. An active-state accessor returns the record whose scheduled interval covers the supplied time; check its prices for `na` before drawing or calculating with them. The completed accessor counts back from the most recently completed record, with zero meaning that record.

### Session close versus drawing end

`closeTime` represents the exclusive session close. `sessEndTime` stores that close minus one millisecond, preserving the inclusive end convention. A session ending at 12:00 excludes the candle opening at 12:00 from its prices. The example uses stored containing-bar timestamps to place its box edges.

An explicit line end at 12:00 represents that exact clock time, allowing the level to reach the 12:00 candle. `SessionAnchor` chooses whether level starts follow the session opening, session end or extreme timestamps. These drawing choices do not change the OHLC collection interval.

### Observed closures and overnight continuation

The engine detects a long gap when a new candle opens after a sufficiently long interval since the previous candle closed. The default threshold is one day. It then adjusts eligible endpoints using the gap's local calendar-date span while preserving their wall-clock time. It does not forecast a weekend before the reopening bar exists.

For example, an endpoint on Saturday inside a Friday-to-Sunday closure shifts by the observed two-date span to Monday at the same clock time. This differs from merely moving it to the first available Sunday timestamp. A daylight-saving change is handled using calendar dates rather than rounding elapsed hours into a day count.

If a Friday session was still underway when trading stopped and its close fell inside the gap, the engine extends that session's close and resumes the same record after reopening. Its Friday prices remain part of the range. A scheduled Sunday opening covered by the resumed interval does not create a competing record. Sessions already completed before the closure remain separate.

The three adjusted endpoint fields use field-level `varip` so first-update changes survive subsequent realtime updates. Price fields retain normal rollback behavior. Preserve that distinction when changing the state type; blanket intrabar persistence would change how OHLC evolves.

## Stage 5: Retain Observations, Then Draw

Primary functions: `trimSessionStates`; example helpers `pruneExpiredSession`, `renderRetainedSessionHistory`, `syncSessionVisuals`, `renderSessionStates`.

**Calendar processing limits and record counts solve different problems.** The engine's default days limit gates how far back it processes. It does not promise a number of retained sessions. For count-based history, pass `int(na)` to process loaded bars and let the host trim by record count.

The basic trim helper keeps at least one record and removes array references. It does not delete linked drawings or remove empty records. A consumer that owns drawing IDs must delete those objects when their corresponding record is removed.

The example performs that extra cleanup and skips older records with no prices. Its regional history budget allows N previous sessions plus the current record. Daily storage allows an additional record because previous-day levels are presented alongside the developing day. Chart-object budgets can reduce the requested retention amount, and insufficient loaded history can also leave fewer observations.

For speed, the example processes historical state without creating every intermediate session drawing. On the last confirmed historical bar it renders the retained snapshot. Live execution then updates current visuals and handles rollover. This avoids creating objects that would immediately be deleted, while preserving historical calculations needed by later state and alerts.

Deferred rendering must use timestamps stored when the session was processed. Using the current rendering bar's `time` for historical box anchors collapses old drawings onto the newest bar. Likewise, when a gap extends a line's life, the host must allow a previously frozen visual tracker to continue updating.

Daily identifiers follow a separate daily-bar counter. They count available daily bars instead of calendar weekdays, but are not tied to the exact oldest record of every individual session. Trade-window shading retains its separate calendar-history gate. Changing session retention does not automatically change those other features.

## Stage 6: Consume State and Maintain Objects

Primary functions: `updateLine`, `updateLabel`, `updateBox`, `updateTableCell`, `clearDrawings`, `trimPool`, `toChartPoints`, `planTradeWindow`.

Create drawings in the host with the intended coordinate mode. Timestamp coordinates require `xloc.bar_time`; the future-bar restriction associated with bar-index coordinates should not be applied to these timestamps. Setter methods update only supplied, non-`na` properties. Use a native setter when your intention is to clear a property to `na`.

Cleanup functions delete the supplied drawing objects and empty their arrays. Pool trimming assumes newest-first storage and pops older objects. These helpers support object ownership; they do not decide which session should remain visible.

Trade-window planning returns boundaries and flags for host rendering. It neither places trades nor emits alerts. The example's round-number support comes from its separate RoundNumberLibrary import, and its breakout logic and visual preferences remain in the indicator.

## Critical Rules / Execution Patterns

### Minimal persistent consumer

The following uses the current checked-in test import. Confirm that the selected publication version includes the local source changes. Replace it with the final publication import when releasing a consumer.

```pine
//@version=6
indicator("Utility Session Consumer", overlay = true)
import OneCleverGuy/UtilityLibrary1CGTESTA/12 as UTIL

var UTIL.SessionConfig cfg = UTIL.SessionConfig.new(
     preset = UTIL.SessionPreset.Custom, customSession = "0930-1600",
     customLabel = "Cash", timezone = "America/New_York")
var array<UTIL.SessionState> states = array.new<UTIL.SessionState>()
var UTIL.SessionInfo descriptor = UTIL.resolveSessionInfo(
     cfg.preset, cfg.customSession, cfg.customLabel, cfg.timezone)

bool replayClock = barstate.isrealtime and not na(time_close) and timenow > time_close + UTIL.durationMs(UTIL.Duration.Day)
int nowTime = (barstate.isrealtime and not replayClock ? timenow : nz(time_close, time))
bool needsMinutes = UTIL.needsSessionIntrabars(descriptor)
UTIL.IntrabarData minutes = UTIL.requestIntrabarData(20, needsMinutes)
UTIL.SessionInfo resolved = UTIL.runSessionEngine(cfg, states, minutes, nowTime, int(na))
int retainedCount = UTIL.trimSessionStates(states, 8)
UTIL.SessionState active = UTIL.getActiveSession(states, nowTime)
float sessionHigh = (not na(active) ? active.highPrice : float(na))
plot(sessionHigh, "Active session high", style = plot.style_linebr)
```

This retains eight total records and plots the active high; it does not reproduce the example's full historical boxes. The clock fallback mirrors the implementation's replay heuristic. It is not an official replay-mode detector.

### Troubleshooting by pipeline stage

| Symptom | Check |
|---|---|
| No state appears | Enabled config, valid custom string, correct timezone and intraday timeframe. |
| Active state has no price | Whether its opening has occurred and qualifying intrabars are available. |
| Boundary high includes earlier prices | Request condition, shared arrays and scanner interval; avoid whole-candle fallbacks. |
| Requested history is short | Distinct record starts, loaded data, calendar processing gate, object budget and cleanup policy. |
| Old boxes appear on the newest bar | Deferred rendering must use recorded session bar timestamps. |
| Friday state splits on reopening | Gap adjustment must precede new-state creation; preserve overlap suppression. |
| Line stays frozen after a gap | Verify both library endpoint adjustment and host tracker reactivation. |
| Identifiers differ from session history | Daily-bar retention is separate from each session's record list. |

### Validation before release

Compile the library and both the minimal consumer and full example against the intended import version in TradingView. Compare aligned boundaries with a non-aligned start/end on a 15-minute chart. Test realtime before and after a boundary, limited minute history, Friday-to-Sunday replay, an already-completed Friday session, and a daylight-saving weekend. Compare record identities and OHLC as well as visual endpoints.

Check count-based history on a market without weekend candles and on a continuously traded symbol. Confirm the historical snapshot and live rollover agree. Profile the same symbol, timeframe, history depth and enabled features when comparing performance. The observed improvement in one chart setup is not a universal loading-time guarantee.

Use standard time-based intraday charts for this integration. Daily/higher timeframes do not execute the session-state engine; nonstandard chart behavior requires separate validation. Time helpers with nominal durations or date-based offsets should not be treated as universal substitutes for actual bar intervals and timezone-aware timestamps.

## Conclusion

1. **Configuration belongs to the host.** Explicit schedules, units and independent persistent arrays make the utilities reusable across consumers.
2. **State follows qualifying observations.** Boundary scans and observed-gap continuation preserve session identity without allowing line expiry to change prices.
3. **Presentation follows retained state.** Count records explicitly, render the surviving history once and let the host maintain live drawings and alerts.
