# AGENTS.md - Indicators/Single-Session

Pine v6 overlay: `Ultra-Single-Session-2026-PUB.ps`, titled `Single Session`
(short title `SingleSess`). The indicator owns inputs, visual configuration,
drawing handles, retention and alert policy for one session and one trade
window. The utility library owns session computation and timing. This public
consumer no longer embeds the old session engine or imports InputLibrary.

## Dependencies

```pine
import OneCleverGuy/UtilityLibrary1CG/1 as UTIL
```

Local implementation: `../../Libraries/Pine-Script-Utilities/1CG-PS-Utilities.pine`.
See that library's `AGENTS.md` for its API. Follow `../../docs/style-guide.md`,
`../../docs/agents-file-standard.md` and `../../docs/tradingview-publication-markup.md`.
Keep published import versions verified; never add library-to-library imports.

## Input Groups

| Group | Contract |
| --- | --- |
| Timezone | Selected timezone applies to custom session, trade window and line end times; presets resolve in their native timezone. |
| SESSION | Enable, enum preset, custom label/hours, colors, box/text, high/low lines, history, anchor, Frontrun, expiry, labels, alerts. Custom fields are active only for Custom. |
| Trade Window | One window, enabled by default at 10:00-11:00; V-Lines and background are independent visual controls. |
| History | Default 5 previous occurrences, range 1-90; master off sets `daysLimit = 0`. |

## Phases

1. Assemble persistent configs and resolve `TIMENOW`, window gates and history.
2. Determine whether the enabled session needs intrabars above one minute;
   call `UTIL.requestIntrabarData(i_daysInput, needsIntrabarData)` once.
3. Call `UTIL.runSessionEngine()` unconditionally. For non-na session info,
   synchronize retained visuals, process alerts and expose newest-record prices.
4. Draw/clear trade-window lines and calculate its background independently.
5. On the last bar, prune stale alert trackers; emit global backgrounds,
   status-line high/low plots and break markers.

## Enums and UDTs

`AlertMode`: `off`, `onBreak`, `closeOut`, `closeIn`. `UTIL.SessionAnchor`:
`ExtremeTime` (input default), `SessionOpen`, `SessionEnd`. Presets cover FX,
regional equity sessions, futures, four daily boundaries and Custom. They are
session templates, not a complete exchange holiday/break calendar.
`UTIL` also supplies timezone, style, width, text, alignment, hour and minute enums.
Convert visual enums during config assembly, not in per-record render loops.

| Type / store | Fields and ownership |
| --- | --- |
| `UTIL.SessionConfig sessionConfig` | Persistent enabled state, preset/custom definition, timezone, anchor, line-end hour/minute/timezone and extra-day flag. |
| `array<UTIL.SessionState> sessionStates` | Persistent newest-first engine records: OHLC, session bounds, extreme/line timestamps and containing-bar anchors. No drawing IDs. |
| `SessionVisualConfig visualConfig` | Enable/color/background fields; line/box visibility, resolved style/width, labels, `aheadBars`, separate history flags, box-text style/alignment and anchor. |
| `SessionVisualState` | `highLine`, `lowLine` (line), `highLabel`, `lowLabel` (label), `square` (box); all initially na. |
| `BarTracker` | Integer `sessBar`, `hiBar`, `loBar`, `hiTime`, `loTime`, `endBar`, initially na; freeze state does not change the timestamp coordinate model. |
| `BreakTracker` | `sessStartTime` plus `highBroken`, `lowBroken`, `highAlerted`, `lowAlerted`, initially false. |
| `TradeWindowState windowState` | `lastDrawnStart` initially na and `endAdjusted` initially false; persistent line-pool bookkeeping. |

`SESSION_SLOT_KEY = "sess01"`. `sessionInstanceKey(slot, start)` returns
`slot:timestamp`. `visualStateMap`, `barTrackerMap` and `breakTrackerMap` are
persistent maps keyed by that string. Window start/end lines live in separate
persistent arrays. Do not put chart object handles back in engine state.

## Function Map

| Helper / group | Arguments / result / responsibility |
| --- | --- |
| `sessionInstanceKey`, `getOrCreateVisualState` | Slot + start -> string; instance key -> persistent `SessionVisualState`. |
| `renderSessionEnd` | `UTIL.SessionState` -> int, `sessEndTime - 1`. |
| `drawTradeWindow` | Start/end line arrays, `TradeWindowState`, session string, history flag, color/style/width, days limit and clock -> updated `TradeWindowState`; consumes `UTIL.planTradeWindow`, creates/reuses/prunes lines. |
| `clearTradeWindow` | Two line arrays -> fresh `TradeWindowState`; deletes both pools. |
| `createHiLoLine`, `createHiLoLabel`, `createSessionBox` | Side/style or box parameters -> new local drawing handle. |
| `clearVisualState`, `applyLabelOffsetStyle`, `syncSessionBox` | Mutate/delete selected drawings or align labels/box with current state and appearance. |
| `advanceHiLoLines` | State, visual state/config, daily flag, clock and instance key -> updates timestamp anchors and capped line/label projection. |
| `ensureCurrentSessionShapes`, `applyHistoryVisuals` | State, visual config, daily flag and key -> create/update current or completed-session drawings and visibility. |
| `isNewSessionInstance` | States, daily flag, first-bar flag, slot -> bool; identifies rollover or missing non-daily tracking. |
| `registerStateTracker`, `registerSessionTracker` | State or newest state array + slot -> establish local tracker/visual state without overwriting engine start-bar anchors. |
| `pruneExpiredSession`, `trimDailyHistoryVisuals` | Retention or daily visibility parameters -> remove empty/expired records and drawings, or hide older daily shapes. |
| `renderSessionStates`, `renderRetainedSessionHistory` | States, appearance, daily flag, clock and slot -> bool; live lifecycle updates or deferred historical draw pass. |
| `syncSessionVisuals` | Session config, visual config, resolved info, states, clock, history limit, slot -> bool enabled; central pruning/render orchestration. |
| `findAlertableSession` | States + clock -> newest completed, unexpired `SessionState`, or na. |
| `registerBreak` | Tracker, side, qualifying condition, window gate, message -> bool marker; consumes side and conditionally calls alert. |
| `processSessionBreakAlerts` | States, mode, label, clock, window gate, slot -> `[bool, bool]` high/low markers. |
| `pruneBreakTrackers` | States + slot -> removes map keys absent from retained records. |

## Function Hierarchy

```text
Execution
+-- Shared runtime -> resolveSessionInfo / needsSessionIntrabars / requestIntrabarData
+-- runSessionEngine (unconditional)
|   +-- syncSessionVisuals (valid SessionInfo only)
|   |   +-- pruneExpiredSession
|   |   +-- renderRetainedSessionHistory (last confirmed history)
|   |   +-- isNewSessionInstance / registerSessionTracker (live or last bar)
|   |   +-- applyHistoryVisuals / trimDailyHistoryVisuals
|   |   +-- renderSessionStates
|   |       +-- ensureCurrentSessionShapes / applyHistoryVisuals
|   |       +-- syncSessionBox / advanceHiLoLines
|   +-- processSessionBreakAlerts -> findAlertableSession -> registerBreak
+-- drawTradeWindow -> UTIL.planTradeWindow / UTIL.trimPool
|   or clearTradeWindow -> UTIL.clearDrawings
+-- pruneBreakTrackers (last bar)
+-- global backgrounds / status-line plots / markers
```

## Standard Integration Pattern

The source persists `sessionConfig`, `visualConfig`, `sessionStates` and all
maps with `var`. Within this indicator, preserve this handoff order (existing
inputs/configs and local helpers are required):

```pine
UTIL.IntrabarData intrabarData = UTIL.requestIntrabarData(i_daysInput, needsIntrabarData)
UTIL.SessionInfo sessionInfo = UTIL.runSessionEngine(
     sessionConfig, sessionStates, intrabarData, TIMENOW, int(na))
if not na(sessionInfo)
    syncSessionVisuals(sessionConfig, visualConfig, sessionInfo,
         sessionStates, TIMENOW, daysLimit, SESSION_SLOT_KEY)
    string alertLabel = sessionConfig.preset == UTIL.SessionPreset.Custom ? i_customLabel : sessionInfo.labelText
    [highBreakFlag, lowBreakFlag] = processSessionBreakAlerts(
         sessionStates, i_alertMode, alertLabel, TIMENOW, isAlertWindowOpen, SESSION_SLOT_KEY)
```

## Rules And Pitfalls

| Rule | Detail |
| --- | --- |
| Preserve engine history | Call the engine every bar, including when disabled; disabled configs return na. The final `int(na)` disables its calendar cutoff. Host retention counts actual occurrences. |
| Replay clock | Live uses wall time unless realtime time is behind by more than a bar (minimum one minute); replay/historical uses `nz(time_close, time)`. |
| Retention | Drop older all-OHLC-na records, preserve index 0. Retain `daysLimit + 1` non-daily or `daysLimit + 2` daily records. Gaps do not consume observed records; available data may limit count. |
| Daily rendering | Equal open/close defines 24 hours. Developing index 0 has no drawings; completed index 1 can show box, lines and PDH/PDL labels even with history off. Older daily records follow history switches. |
| Deferred drawings | Ordinary historical bars compute/prune only. Draw retained sessions on `barstate.islastconfirmedhistory`; live/last bars restore missing current shapes and update lifecycle. |
| Timestamp coordinates | Lines/labels use `xloc.bar_time`. Boxes use containing-bar timestamps with scheduled fallbacks. Preserve engine timestamps; do not convert anchors back to bar indexes. |
| Expiry and gaps | Frontrun projects by bar duration, capped at `lineEndTime`. Observed-gap extensions can clear a frozen end tracker. The utility engine owns gap timing; do not hardcode weekends. |
| Calendar visuals | Session/window backgrounds and trade-window planning still use calendar history windows. Window line pools are capped at `daysLimit + 1`. |
| Window gate | One window only. Disabling V-Lines or background does not disable alert filtering; disable the window itself. Disabled window allows all times. |
| Alert eligibility | Evaluate newest completed record with clock after `renderSessionEnd` and at/before `lineEndTime`. Session enable controls alerts, not high/low drawing visibility. |
| Exact alert conditions | On Break uses strict high/low penetration, not equal touch despite tooltip wording. Close Out uses confirmed close strictly beyond. Sweep uses confirmed penetration and close inside or on the level. |
| One-shot consumption | First qualifying event consumes each side even outside the window, with no marker/alert or later retry. `alert.freq_once_per_bar` can suppress simultaneous notifications. Ordinary state is not varip; do not promise tick-persistent deduplication. |
| Status line | High/Low plots expose index 0 even in daily mode, while daily chart drawings show the previous record. |
| Single-session scope | No daily dividers, round-number grids, multiple windows or history diagnostic switch are present. Do not copy those Ultra Sessions features into this guide. |
| Publication edits | Preserve existing `Ultra-Single-Session-2026-Description.txt` content and append only verified changes for this update. Do not rewrite earlier copy or describe unchanged alert modes as new. |

Validate future code changes in TradingView: intrabar boundaries, overnight and
daily presets, history off/1/5/90, reload/replay, long-gap reopen, all alert modes
inside/outside the window, and hidden window lines with filtering enabled.
No local Pine compiler is provided; documentation checks are not chart tests.
