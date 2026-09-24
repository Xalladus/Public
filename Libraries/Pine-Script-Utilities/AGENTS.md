# AGENTS.md - Libraries/Pine-Script-Utilities

Pine Script v6 utility and session-state library. The library owns enums, validation, conversions, time calculations, conditional intrabar retrieval, and session mutation. The host owns inputs, persistent arrays, drawing creation, retention policy, alerts, and cross-library composition. Existing drawing setters and cleanup exports mutate or delete host-owned objects; they never create them. The advanced indicator consumer is [Ultra-Sessions-2026.pine](../../../Trading-View-Scripts/Indicators/Ultra-Sessions/Ultra-Sessions-2026.pine) in the sibling Trading-View-Scripts project.

Current documented import (version 1), also used by `Ultra-Sessions-2026.pine`:

```pine
import OneCleverGuy/UtilityLibrary1CG/<version> as UTIL
```

The library imports nothing. The Ultra Sessions consumer also imports `OneCleverGuy/RoundNumberLibrary/3 as RNL`. Keep consumer imports aligned with the published version; do not invent version numbers. Shared standards live in the sibling Trading-View-Scripts project: [coding style](../../../Trading-View-Scripts/docs/style-guide.md), [AGENTS](../../../Trading-View-Scripts/docs/agents-file-standard.md), [guide](../../../Trading-View-Scripts/docs/guide-standards.md), [publication markup](../../../Trading-View-Scripts/docs/tradingview-publication-markup.md).

Keep this file compact: retain the callable surface and non-obvious runtime contracts; document shared arguments once and summarize regular numeric enum ranges. Use grouped references under the shared AGENTS standard. Internal imports use the latest verified numeric version; publication examples use `<version>` and TradingView markup. Keep complete integrations and edge cases in the guide, and a concise copyable consumer example in publication copy.

## Public API

Source-order signatures. `->` is reference notation, not executable Pine syntax. Method receivers are explicit below; consumers call them as `i_style.toLineStyle()` or `myLine.updateLine(...)`.

```pine
UTIL.toTimezone(Timezones this) -> string
UTIL.toHourInt(Hours this) -> int
UTIL.toMinuteInt(Minutes this) -> int
UTIL.toHhmm(QuarterHours this) -> string
UTIL.toLineStyle(LineStyle this) -> string
UTIL.toLineWidth(LineSize this) -> int
UTIL.toTextSizeString(TextSize this) -> string
UTIL.toHorizontalAlign(HorizontalAlign this) -> string
UTIL.toVerticalAlign(VerticalAlign this) -> string
UTIL.toLineExtend(LineExtend this) -> string
UTIL.toLabelStyle(LabelStyle this) -> string
UTIL.updateLine(line this, int _x1 = na, float _y1 = na, int _x2 = na, float _y2 = na, string _extend = na, color _color = na, string _style = na, int _width = na) -> line
UTIL.updateLabel(label this, int _x = na, float _y = na, string _text = na, string _yloc = na, color _color = na, string _style = na, color _textColor = na, string _textSize = na, string _textAlign = na, string _tooltip = na, string _fontFamily = na) -> label
UTIL.updateBox(box this, int _left = na, float _top = na, int _right = na, float _bottom = na, color _borderColor = na, int _borderWidth = na, string _borderStyle = na, string _extend = na, color _bgColor = na, string _text = na, string _textSize = na, color _textColor = na, string _textHAlign = na, string _textVAlign = na, string _textWrap = na, string _fontFamily = na) -> box
UTIL.updateTableCell(table this, int _column, int _row, string _text = na, color _textColor = na, string _textSize = na, color _bgColor = na, string _textHAlign = na, string _textVAlign = na, string _tooltip = na, string _fontFamily = na) -> table
UTIL.detectAssetClass() -> AssetClass
UTIL.priceDecimals() -> int
UTIL.tickValue() -> float
UTIL.priceToTicks(float _priceDistance) -> float
UTIL.ticksToPrice(float _ticks) -> float
UTIL.pipSize(float _pipSizeOverride = na) -> float
UTIL.priceToPips(float _priceDistance, float _pipSizeOverride = na) -> float
UTIL.pipsToPrice(float _pips, float _pipSizeOverride = na) -> float
UTIL.defaultQuantityStep() -> float
UTIL.roundQuantity(float _quantity, float _step = 1.0) -> float
UTIL.positionNotional(float _quantity, float _price) -> float
UTIL.durationMs(Duration _duration) -> int
UTIL.timezoneOffsetMs(string _timezone) -> int
UTIL.hhmmFromParts(Hours _hour, Minutes _minute) -> string
UTIL.hhmmToParts(string _hhmm) -> [int, int]
UTIL.hhmmToMinutes(string _hhmm) -> int
UTIL.minutesToHhmm(int _minutesOfDay) -> string
UTIL.barDurationMs() -> int
UTIL.barDayBoundaryOffsetMs(string _timezone, int _hour, int _minute) -> int
UTIL.clearDrawings(array<line> lines = na, array<label> labels = na, array<box> boxes = na, array<polyline> polylines = na) -> int
UTIL.trimPool(int maxSize, array<line> lines = na, array<label> labels = na, array<box> boxes = na, array<polyline> polylines = na) -> int
UTIL.toChartPoints(array<int> _times, array<float> _prices) -> array<chart.point>
UTIL.sessionToParts(string _session) -> [int, int, int, int]
UTIL.sessionDurationMs(string _session) -> int
UTIL.getObservedLongGap(int _minimumGapMs = 86400000) -> [bool, int, int, int]
UTIL.historyCutoffTime(int _calendarDays, int _referenceTime) -> int
UTIL.isWithinHistoryWindow(int _calendarDays, int _referenceTime, int _extraDays = 0) -> bool
UTIL.resolveSessionInfo(SessionPreset _preset, string _customSession = na, string _customLabel = na, string _customTimezone = na) -> SessionInfo
UTIL.isInSession(string _session, string _timezone) -> bool
UTIL.isInAnySession(array<string> _sessions, string _timezone) -> bool
UTIL.getSessionStartTime(SessionInfo _session, int _dayOffsetMs = 0) -> int
UTIL.isSessionFirstBar(SessionInfo _session, int _dayOffsetMs = 0) -> bool
UTIL.isSessionBoundaryInBar(bool _isStart, SessionInfo _session, int _barStartTime, int _barEndTime, int _dayOffsetMs = 0) -> bool
UTIL.needsSessionIntrabars(SessionInfo _session) -> bool
UTIL.requestIntrabarData(simple int _historyDays, bool _required = true) -> IntrabarData
UTIL.scanIntrabarRange(IntrabarData _intrabarData, int _startTime, int _endTime, float _seedHigh, float _seedLow, int _seedHighTime, int _seedLowTime) -> IntrabarScan
UTIL.runSessionEngine(SessionConfig _config, array<SessionState> _states, IntrabarData _intrabarData, int _timeNow, int _daysLimit = 5) -> SessionInfo
UTIL.getActiveSession(array<SessionState> _states, int _timeNow) -> SessionState
UTIL.getCompletedSession(array<SessionState> _states, int _sessionsBack, int _timeNow) -> SessionState
UTIL.trimSessionStates(array<SessionState> _states, int _maxSessions) -> int
UTIL.planTradeWindow(string _session, string _timezone, int _daysLimit, int _timeNow, int _lastDrawnStart) -> TradeWindowPlan
```

## Exported Enums

Regular numeric ranges are summarized below. Timezone and preset members are listed explicitly after the overview; [source declarations](1CG-PS-Utilities.pine) remain authoritative when changing them.

| Enum | Values / use |
|---|---|
| `Timezones` | `utc`, `exch`, and named city selections; `toTimezone()` resolves an IANA string or the exchange timezone. |
| `Hours`, `Minutes`, `QuarterHours` | `h0`â€“`h23`, `m0`â€“`m59`, and `t0000`â€“`t2345` in 15-minute steps; convert to integers or HHMM tokens. |
| `Duration` | `Minute`, `QuarterHour`, `HalfHour`, `Hour`, `FourHours`, `EightHours`, `TwelveHours`, `Day`, `Week`; fixed millisecond durations. |
| `AssetClass` | `Forex`, `Crypto`, `Futures`, `Stock`, `Index`, `CFD`, `Fund`, `Bond`, `Economic`, `Other`. |
| `LineStyle`, `LineSize` | `solid`, `dotted`, `dashed`, `lArrow`, `rArrow`, `bArrow`; widths `thin`, `normal`, `heavy`, `thick`, `wide` map to 1â€“5. |
| `TextSize` | `auto`, `tiny`, `small`, `normal`, `large`, `huge`. |
| `HorizontalAlign`, `VerticalAlign` | `left`/`center`/`right`; `top`/`center`/`bottom`. |
| `LineExtend`, `LabelStyle` | Extension: `none`, `right`, `left`, `both`. Labels: `center`, `down`, `left`, `right`, `up`, `lowLeft`, `lowRight`, `upperLeft`, `upperRight`. |
| `SessionPreset` | `Custom`; regional `Fx*`, exchange `Eq*`, futures `Fut*`, and daily `Daily*` schedules. Resolve through `resolveSessionInfo`; do not duplicate preset lookup tables in consumers. |
| `SessionAnchor` | `SessionEnd` (default; current chart bar while active), `SessionOpen`, `ExtremeTime`; controls level anchors, not OHLC collection. |

### `Timezones` members

| Member | Display / meaning |
|---|---|
| `utc` | UTC |
| `exch` | Brokers/Exchange |
| `lon` | Europe/London(+0/+1) |
| `ber` | Europe/Berlin(+1/+2) |
| `jnb` | Africa/Johannesburg(+2) |
| `nbo` | Africa/Nairobi(+3) |
| `ath` | Europe/Athens(+2/+3) |
| `cai` | Africa/Cairo(+2/+3) |
| `msk` | Europe/Moscow(+3) |
| `ruh` | Asia/Riyadh(+3) |
| `doha` | Asia/Qatar(+3) |
| `dxb` | Asia/Dubai(+4) |
| `bom` | Asia/Kolkata(+5.5) |
| `rgn` | Asia/Yangon(+6.5) |
| `bkk` | Asia/Bangkok(+7) |
| `hkg` | Asia/Hong_Kong(+8) |
| `bjs` | Asia/Shanghai(+8) |
| `sgp` | Asia/Singapore(+8) |
| `sel` | Asia/Seoul(+9) |
| `tyo` | Asia/Tokyo(+9) |
| `adl` | Australia/Adelaide(+9.5/+10.5) |
| `drw` | Australia/Darwin(+9.5) |
| `syd` | Australia/Sydney(+10/+11) |
| `lhi` | Australia/Lord_Howe(+10.5/+11) |
| `vvo` | Asia/Vladivostok(+10) |
| `nou` | Pacific/Noumea(+11) |
| `akl` | Pacific/Auckland(+12/+13) |
| `tonu` | Pacific/Tongatapu(+13) |
| `kir` | Pacific/Kiritimati(+14) |
| `ppgo` | Pacific/Pago_Pago(-11) |
| `adk` | America/Adak(-10/-9) |
| `hnl` | Pacific/Honolulu(-10) |
| `anc` | America/Anchorage(-9/-8) |
| `gam` | Pacific/Gambier(-9) |
| `yvr` | America/Vancouver(-8/-7) |
| `lax` | America/Los_Angeles(-8/-7) |
| `pit` | Pacific/Pitcairn(-8) |
| `den` | America/Denver(-7/-6) |
| `phx` | America/Phoenix(-7) |
| `edm` | America/Edmonton(-7/-6) |
| `chi` | America/Chicago(-6/-5) |
| `mex` | America/Mexico_City(-6) |
| `win` | America/Winnipeg(-6/-5) |
| `bz` | America/Belize(-6) |
| `ny` | America/New_York(-5/-4) |
| `tor` | America/Toronto(-5/-4) |
| `lim` | America/Lima(-5) |
| `bog` | America/Bogota(-5) |
| `ccs` | America/Caracas(-4) |
| `scl` | America/Santiago(-4/-3) |
| `lpb` | America/La_Paz(-4) |
| `sp` | America/Sao_Paulo(-3) |
| `bsb` | America/Araguaina(-3) |
| `eze` | America/Argentina/Buenos_Aires(-3) |
| `yyt` | America/St_Johns(-3.5/-2.5) |
| `nor` | America/Noronha(-2) |
| `sg` | Atlantic/South_Georgia(-2) |

### `SessionPreset` members

| Member | Display / meaning |
|---|---|
| `FxSydney` | FX: Sydney (0700-1600 Sydney) |
| `FxTokyo` | FX: Tokyo (0900-1800 Tokyo) |
| `FxLondon` | FX: London (0800-1700 London) |
| `FxNewYork` | FX: New York (0800-1700 New York) |
| `EqUnitedStates` | Equities: US RTH (0930-1600 New York) |
| `EqUnitedKingdom` | Equities: UK LSE (0800-1630 London) |
| `EqGermany` | Equities: Germany Xetra (0900-1730 Berlin) |
| `EqJapan` | Equities: Japan TSE (0900-1530 Tokyo) |
| `EqHongKong` | Equities: Hong Kong HKEX (0930-1600 Hong Kong) |
| `EqIndia` | Equities: India NSE (0915-1530 Kolkata) |
| `EqAustralia` | Equities: Australia ASX (1000-1600 Sydney) |
| `FutCmeGlobex` | Futures: CME Globex (1700-1600 Chicago) |
| `FutCmeEquityDay` | Futures: CME Equity Day (0830-1515 Chicago) |
| `FutEurexCore` | Futures: Eurex Core (0800-2200 Berlin) |
| `DailyFxClose` | Daily: FX Close (1700-1700 New York) |
| `DailyUtc` | Daily: UTC (0000-0000 UTC) |
| `DailyExchange` | Daily: Exchange (0000-0000 Exchange) |
| `DailyNewYork` | Daily: New York (0000-0000 New York) |
| `Custom` | Custom |

## Exported Types

Persist configuration and one newest-first state array per session with `var`. Accessors return references; the engine mutates session records in place. Fields without explicit array defaults must receive initialized arrays before scanning. Timestamps are UNIX milliseconds.

### `IntrabarScan`

Aggregated one-minute data for a time range inside the current bar.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `openPrice` | `float` | `na` | First one-minute open inside the range. |
| `highPrice` | `float` | `na` | Highest one-minute high inside the range, seeded by the caller. |
| `lowPrice` | `float` | `na` | Lowest one-minute low inside the range, seeded by the caller. |
| `closePrice` | `float` | `na` | Last one-minute close inside the range. |
| `highTime` | `int` | `na` | Stamp time written when a new high was found. |
| `lowTime` | `int` | `na` | Stamp time written when a new low was found. |
| `hasData` | `bool` | `false` | True when at least one one-minute bar fell inside the range. |

### `IntrabarData`

Shared one-minute arrays for the current chart bar.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `times` | `array<int>` | `implicit na; supply array` | One-minute bar opening timestamps. |
| `opens` | `array<float>` | `implicit na; supply array` | One-minute opening prices. |
| `highs` | `array<float>` | `implicit na; supply array` | One-minute high prices. |
| `lows` | `array<float>` | `implicit na; supply array` | One-minute low prices. |
| `closes` | `array<float>` | `implicit na; supply array` | One-minute closing prices. |

### `SessionInfo`

Static session descriptor including the timezone it is defined in.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `labelText` | `string` | `na` | Display label for the session. |
| `session` | `string` | `na` | Session string in "HHMM-HHMM" format. |
| `timezone` | `string` | `na` | IANA timezone the session times are expressed in. |
| `openHour` | `int` | `na` | Session open hour. |
| `openMinute` | `int` | `na` | Session open minute. |
| `closeHour` | `int` | `na` | Session close hour. |
| `closeMinute` | `int` | `na` | Session close minute. |
| `durationMs` | `int` | `na` | Nominal session length in milliseconds. Calendar boundaries are built separately so DST remains correct. |
| `isDaily` | `bool` | `false` | True when open and close are identical, meaning a 24-hour session. |

### `SessionState`

Lifecycle state for one tracked session instance.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `openPrice` | `float` | `na` | First available qualifying session open. na until session data is available. |
| `openTime` | `int` | `na` | UNIX time when the session opened. |
| `highPrice` | `float` | `na` | Current session high. |
| `highTime` | `int` | `na` | UNIX time when the session high was made. |
| `lowPrice` | `float` | `na` | Current session low. |
| `lowTime` | `int` | `na` | UNIX time when the session low was made. |
| `closePrice` | `float` | `na` | Latest session close, finalized when the session ends. |
| `closeTime` | `varip int` | `na` | Session close boundary, extended across an observed closure that interrupted the session. |
| `labelText` | `string` | `na` | Resolved session label text. |
| `sessStartTime` | `int` | `na` | Session start UNIX time. |
| `sessEndTime` | `varip int` | `na` | Inclusive session end UNIX time. Interrupted sessions resume after observed long gaps. |
| `highLineStartTime` | `int` | `na` | High-line anchor UNIX time. |
| `lowLineStartTime` | `int` | `na` | Low-line anchor UNIX time. |
| `lineEndTime` | `varip int` | `na` | UNIX time when the line lifecycle ends. Persists intrabar so first-update gap adjustments survive rollback. |
| `sessStartBarTime` | `int` | `na` | Bar time of the chart bar containing the session start. |
| `sessEndBarTime` | `int` | `na` | Bar time of the chart bar containing the session end. |

### `SessionConfig`

Consumer configuration for the session engine.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `isEnabled` | `bool` | `true` | Master processing toggle. |
| `preset` | `SessionPreset` | `SessionPreset.Custom` | Preset session, or SessionPreset.Custom. |
| `customLabel` | `string` | `na` | Label used when preset is Custom. |
| `customSession` | `string` | `na` | Session string used when preset is Custom. |
| `timezone` | `string` | `na` | IANA timezone for a Custom session. na uses the exchange timezone. Presets ignore this field. |
| `anchor` | `SessionAnchor` | `SessionAnchor.SessionEnd` | Controls high and low line anchor timestamps. |
| `lineEndHour` | `int` | `na` | Hour at which the line lifecycle ends. na ends at session close. |
| `lineEndMinute` | `int` | `na` | Minute at which the line lifecycle ends. Treated as 0 when na. |
| `lineEndTimezone` | `string` | `na` | Timezone for lineEndHour and lineEndMinute. na uses the session timezone. |
| `addExtraLineDay` | `bool` | `false` | Extends the line lifecycle by one additional day. |

### `TradeWindowPlan`

Planning output for drawing trade-window boundaries.

| Field | Type | Default | Meaning |
|---|---|---|---|
| `startTime` | `int` | `na` | Window start UNIX time. |
| `endTime` | `int` | `na` | Scheduled window end UNIX time. |
| `isInWindow` | `bool` | `false` | True when the current bar is inside the window. |
| `alreadyDrawn` | `bool` | `false` | True when this window start was already drawn by the host. |
| `shouldDraw` | `bool` | `false` | True when the host should draw the window boundaries now. |

## Function Reference

Public API gives exact argument order, types, and defaults. This grouped reference pairs argument intent and return shapes with behavior and side effects; argument-heavy operations have dedicated tables below.

| Callable group | Arguments | Returns | Meaning / side effects |
|---|---|---|---|
| Enum conversion methods | `this`: enum selection to convert | `string` or `int`, as listed per method in Public API | Convert the explicit receiver to the corresponding Pine string or integer; do not gather inputs. |
| `detectAssetClass`, `priceDecimals`, `tickValue` | None; uses current chart metadata | `AssetClass`, `int` decimals, `float` tick value respectively | Read chart symbol metadata. Tick value is `mintick * pointvalue`. |
| `priceToTicks`, `ticksToPrice`, `pipSize`, `priceToPips`, `pipsToPrice` | `_priceDistance` or `_ticks`/`_pips`: distance to convert; `_pipSizeOverride`: optional pip size | `float` converted distance or pip size | Convert price distances. Default pip size is ten ticks on forex, one elsewhere; positive override replaces it, nonpositive override returns `na`. |
| `defaultQuantityStep`, `roundQuantity`, `positionNotional` | No arguments for default; `_quantity`, `_step`: quantity and rounding increment; `_quantity`, `_price`: notional inputs | `float` step, rounded quantity, or notional | Step heuristic: 0.01 crypto, 1 elsewhere. Round down and clamp quantity at zero; invalid step falls back to 1. Notional is quantity Ã— price Ã— pointvalue. |
| `durationMs`, `timezoneOffsetMs`, `barDurationMs` | `_duration`: named interval; `_timezone`: target timezone; no arguments for bar duration | `int` milliseconds | Millisecond helpers; named/bar durations are nominal. Timezone offset uses date-based midnight construction, not a universal instantaneous DST calculation. |
| `hhmmFromParts`, `hhmmToParts`, `hhmmToMinutes`, `minutesToHhmm`, `sessionToParts`, `sessionDurationMs` | `_hour`, `_minute`: enum selections; `_hhmm`: clock token; `_minutesOfDay`: minute count; `_session`: session token | Clock `string`, `[int, int]` clock parts, `int` minutes, `[int, int, int, int]` session parts, or `int` duration respectively | Parse/format clock values. Invalid tokens return `na` values; minute-of-day input wraps. Equal valid session endpoints mean 24 hours. |
| `barDayBoundaryOffsetMs`, `getSessionStartTime`, `isSessionFirstBar`, `isSessionBoundaryInBar` | `_timezone`, `_hour`, `_minute`: clock boundary; `_session`: descriptor; `_dayOffsetMs`: day token; `_isStart`, `_barStartTime`, `_barEndTime`: boundary test | `int` offset/start timestamp or `bool` first-bar/boundary test | Build/test session boundaries in the descriptor timezone. Day offset is a nominal-day token. Boundary-in-bar test is strict; first-bar test includes a boundary at bar open. |
| `getObservedLongGap` | `_minimumGapMs`: minimum interruption length | `[bool, int, int, int]`: detected, previous close, current open, duration | Returns detection flag, previous close, current open, and duration. Evaluates on `barstate.isnew`; otherwise timestamps are `na` and duration 0. Nonpositive threshold clamps to 1 ms. |
| `historyCutoffTime`, `isWithinHistoryWindow` | `_calendarDays`: history length; `_referenceTime`: anchor; `_extraDays`: membership slack | `int` cutoff or `bool` membership | Exchange-calendar cutoff preserving reference hour/minute; cutoff days clamp at 0, membership days plus slack at 1. Membership compares current bar open. |
| `resolveSessionInfo`, `isInSession`, `isInAnySession` | `_preset` and custom descriptor fields; `_session` or `_sessions`: membership schedules; `_timezone`: membership timezone | `SessionInfo` or na; `bool` for membership wrappers | Resolve a strict engine descriptor, or test native TradingView session membership. Empty membership array returns false; the native wrappers and engine parser have different syntax contracts. |
| `needsSessionIntrabars`, `requestIntrabarData`, `scanIntrabarRange` | `_session`: descriptor; `_historyDays`: minute request budget; `_required`: request gate; scan range and seeds detailed below | `bool` need, `IntrabarData` arrays, or `IntrabarScan` aggregation | Detect partial boundaries, request shared minute arrays, and scan `[startTime, endTime)` using supplied extreme seeds. Request budget: `min((max(nz(_historyDays), 0) + 1) * 1440, 100000)` minute bars. |
| `runSessionEngine` | `_config`, `_states`, `_intrabarData`, `_timeNow`, `_daysLimit`: configuration, mutable storage, shared data, clock, processing window | `SessionInfo` or na | Mutates the supplied state array and records on eligible intraday bars. Disabled/invalid config returns `na`; disabling does not clear records. A valid returned descriptor does not prove that a record was created. |
| `getActiveSession`, `getCompletedSession` | `_states`: newest-first records; `_timeNow`: clock; `_sessionsBack`: completed-record offset | `SessionState` reference or na | Return references or `na`. Active checks index 0 against inclusive session bounds. Completed counts records ending before the supplied clock; offset 0 is newest, negative offsets clamp to 0. Missing-price records still count. |
| `trimSessionStates` | `_states`: mutable records; `_maxSessions`: retention limit | `int` remaining count | Pops oldest references in place, retaining at least 1; returns remaining count. No drawing cleanup. |
| `planTradeWindow` | `_session`, `_timezone`: schedule; `_daysLimit`: history; `_timeNow`: clock; `_lastDrawnStart`: host deduplication marker | `TradeWindowPlan` | Returns scheduled boundaries and rendering flags, gated by calendar history, realtime readiness, and the last-drawn start. No orders, alerts, or drawings. |
| `updateLine`, `updateLabel`, `updateBox`, `updateTableCell` | `this`: existing object; optional properties below; table also needs `_column`, `_row` | Same `line`, `label`, `box`, or `table` reference | Mutate existing host objects and return the same ID. Optional `na` properties are unchanged; coordinate mode is unchanged. |
| `clearDrawings`, `trimPool`, `toChartPoints` | Optional object pools; `maxSize`: per-pool limit for trim; `_times`, `_prices`: point coordinates | `int` deleted count, `int` remaining count, or `array<chart.point>` | Clear deletes objects/empties arrays and returns deleted count. Trim pops oldest objects from newest-first pools and returns total remaining count. Points uses the shorter time/price array. |

### Arguments for multi-parameter operations

Defaults are listed in Public API; parameters with defaults are optional. Shared drawing options are documented once below. Each method requires its existing object receiver; table updates also require zero-based `_column` and `_row`. All optional drawing properties use `na` to leave the property unchanged.

| Drawing argument | Type | Meaning / applies to |
|---|---|---|
| `_x1` | `int` | New first point x value. |
| `_y1` | `float` | New first point price. |
| `_x2` | `int` | New second point x value. |
| `_y2` | `float` | New second point price. |
| `_extend` | `string` | New extend constant. |
| `_color` | `color` | Line color or label background color. |
| `_style` | `string` | Line or label style constant for the receiving object. |
| `_width` | `int` | New line width in pixels. |
| `_x` | `int` | New x value. |
| `_y` | `float` | New price, used when yloc is yloc.price. |
| `_text` | `string` | Displayed label, box, or table-cell text. |
| `_yloc` | `string` | New yloc constant. |
| `_textColor` | `color` | New text color. |
| `_textSize` | `string` | New text size constant. |
| `_textAlign` | `string` | New text alignment constant. |
| `_tooltip` | `string` | New tooltip text. |
| `_fontFamily` | `string` | New font family constant. |
| `_left` | `int` | New left edge x value. |
| `_top` | `float` | New top edge price. |
| `_right` | `int` | New right edge x value. |
| `_bottom` | `float` | New bottom edge price. |
| `_borderColor` | `color` | New border color. |
| `_borderWidth` | `int` | New border width in pixels. |
| `_borderStyle` | `string` | New border style constant. |
| `_bgColor` | `color` | New background color. |
| `_textHAlign` | `string` | New horizontal text alignment constant. |
| `_textVAlign` | `string` | New vertical text alignment constant. |
| `_textWrap` | `string` | New text wrap constant. |
| `_column` | `int` | Zero-based column index. |
| `_row` | `int` | Zero-based row index. |

#### `clearDrawings`

| Argument | Type | Meaning |
|---|---|---|
| `lines` | `array<line>` | Optional line pool to clear. |
| `labels` | `array<label>` | Optional label pool to clear. |
| `boxes` | `array<box>` | Optional box pool to clear. |
| `polylines` | `array<polyline>` | Optional polyline pool to clear. |

#### `trimPool`

| Argument | Type | Meaning |
|---|---|---|
| `maxSize` | `int` | Maximum number of objects to retain in each supplied pool. |
| `lines` | `array<line>` | Optional line pool to trim (newest first). |
| `labels` | `array<label>` | Optional label pool to trim (newest first). |
| `boxes` | `array<box>` | Optional box pool to trim (newest first). |
| `polylines` | `array<polyline>` | Optional polyline pool to trim (newest first). |

#### `resolveSessionInfo`

| Argument | Type | Meaning |
|---|---|---|
| `_preset` | `SessionPreset` | Preset selection, or SessionPreset.Custom. |
| `_customSession` | `string` | Session string in "HHMM-HHMM" format, required when the preset is Custom. |
| `_customLabel` | `string` | Label applied when the preset is Custom. Defaults to "Custom". |
| `_customTimezone` | `string` | IANA timezone for a Custom session. na uses the exchange timezone. |

#### `isSessionBoundaryInBar`

| Argument | Type | Meaning |
|---|---|---|
| `_isStart` | `bool` | True tests the session open, false tests the session close. |
| `_session` | `SessionInfo` | Session descriptor. |
| `_barStartTime` | `int` | Bar start UNIX time. |
| `_barEndTime` | `int` | Bar end UNIX time. |
| `_dayOffsetMs` | `int` | Nominal-day token from barDayBoundaryOffsetMs() or a manual whole-day shift. |

#### `scanIntrabarRange`

| Argument | Type | Meaning |
|---|---|---|
| `_intrabarData` | `IntrabarData` | Shared one-minute arrays returned by requestIntrabarData(). |
| `_startTime` | `int` | Inclusive range start UNIX time. |
| `_endTime` | `int` | Exclusive range end UNIX time. |
| `_seedHigh` | `float` | Existing high to beat. Pass na to take the first value found. |
| `_seedLow` | `float` | Existing low to beat. Pass na to take the first value found. |
| `_seedHighTime` | `int` | High timestamp kept when no new high is found. |
| `_seedLowTime` | `int` | Low timestamp kept when no new low is found. |

#### `runSessionEngine`

| Argument | Type | Meaning |
|---|---|---|
| `_config` | `SessionConfig` | Session configuration supplied by the host script. |
| `_states` | `array<SessionState>` | State storage for this session. Newest state is index 0. |
| `_intrabarData` | `IntrabarData` | Shared one-minute arrays returned by requestIntrabarData(). |
| `_timeNow` | `int` | Current UNIX timestamp supplied by the host. |
| `_daysLimit` | `int` | Calendar days of history to process. Pass na to process all loaded bars and let the host retain records by count. |

#### `planTradeWindow`

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `string` | Window session string in "HHMM-HHMM" format. |
| `_timezone` | `string` | IANA timezone the window is expressed in. |
| `_daysLimit` | `int` | Calendar days of history the host may draw. |
| `_timeNow` | `int` | Current UNIX timestamp. |
| `_lastDrawnStart` | `int` | Start timestamp the host most recently drew, or na. |

## Function Hierarchy

```text
UTIL
|-- Configuration and representation
|   |-- Enum conversion methods
|   |-- Price, tick, pip and quantity helpers
|   +-- HHMM parsing and calendar helpers
|-- Session pipeline (host calls each bar)
|   |-- resolveSessionInfo -> presetToSessionInfo / buildSessionInfo
|   |-- needsSessionIntrabars -> barDayBoundaryOffsetMs / wallClockTimestamp
|   |-- requestIntrabarData -> conditional security_lower_tf
|   +-- runSessionEngine
|       |-- resolveSessionInfo
|       |-- getObservedLongGap -> adjustSessionStateForGap
|       |   +-- advanceWallClockPastGap / resolveLineEndTime
|       |-- isSessionFirstBar / isLateSessionStart
|       |-- createSessionState (newest first)
|       +-- updateSessionState -> scanIntrabarRange / applySessionAnchors
|-- State consumption
|   |-- getActiveSession / getCompletedSession
|   |-- trimSessionStates (references only)
|   +-- planTradeWindow (separate calendar-gated planner)
|-- Host-owned drawing support
|   |-- updateLine / updateLabel / updateBox / updateTableCell
|   +-- clearDrawings / trimPool / toChartPoints
+-- Ultra-Sessions-2026.pine (sibling project consumer, not imported by library)
    |-- Persistent configurations, state arrays and visual trackers
    |-- Shared intrabar request -> engine per enabled session
    |-- pruneExpiredSession (record count and drawing cleanup)
    |-- renderRetainedSessionHistory (historical snapshot)
    +-- syncSessionVisuals / renderSessionStates (live drawings)
```

## Standard Integration Pattern

This uses the published version 1 import. Verify that the imported publication contains any future local changes before testing. Eight retained records here means at most eight total records, including a developing session; it is not the Ultra Sessions consumer's separate previous-session input.

```pine
//@version=6
indicator("Utility Session Consumer", overlay = true)
import OneCleverGuy/UtilityLibrary1CG/1 as UTIL

string i_session = input.session("0930-1600", "Session")
string i_timezone = input.string("America/New_York", "Timezone")
int i_retainedSessions = input.int(8, "Retained sessions", minval = 1)

var UTIL.SessionConfig cfg = UTIL.SessionConfig.new(
     preset = UTIL.SessionPreset.Custom, customSession = i_session,
     customLabel = "Cash", timezone = i_timezone)
var array<UTIL.SessionState> states = array.new<UTIL.SessionState>()
var UTIL.SessionInfo descriptor = UTIL.resolveSessionInfo(
     cfg.preset, cfg.customSession, cfg.customLabel, cfg.timezone)

bool replayClock = barstate.isrealtime and not na(time_close) and timenow > time_close + UTIL.durationMs(UTIL.Duration.Day)
int nowTime = (barstate.isrealtime and not replayClock ? timenow : nz(time_close, time))
bool needsMinutes = UTIL.needsSessionIntrabars(descriptor)
UTIL.IntrabarData minutes = UTIL.requestIntrabarData(20, needsMinutes)
UTIL.SessionInfo resolved = UTIL.runSessionEngine(cfg, states, minutes, nowTime, int(na))
int retainedCount = UTIL.trimSessionStates(states, i_retainedSessions)
UTIL.SessionState active = UTIL.getActiveSession(states, nowTime)
bool hasActiveHigh = not na(resolved) and not na(active) and not na(active.highPrice)
float sessionHigh = (hasActiveHigh ? active.highPrice : float(na))
plot(sessionHigh, "Active session high", style = plot.style_linebr)
```

For multiple sessions, OR the enabled sessions' boundary checks, request once, and pass the same arrays to each engine call. Persist a separate state array per session. If drawings are attached to records, replace simple trimming with host cleanup before dropping each record.

## Execution Ownership

| Phase | Owner and behavior |
|---|---|
| Initialization | Host declares inputs, persistent configurations, descriptors and arrays. Library has no imports or input declarations. |
| Every chart bar | Host checks boundaries and calls the shared request wrapper, then the engine for each enabled session. Engine mutates arrays and UDTs in place. |
| Historical request initialization | Wrapper also requests on `barstate.islastconfirmedhistory` on eligible timeframes, even when no boundary needs scanning. Preserve this preparation for realtime requests. |
| First update after long gap | Engine adjusts retained endpoints when the gap is observed. Three `varip` endpoint fields preserve these first-update changes. Price fields keep ordinary rollback behavior. |
| Retention | Host chooses record counts or a calendar processing window. Engine does not trim storage automatically. |
| Historical rendering | Ultra Sessions computes history first, then creates retained drawings on the last confirmed historical bar. |
| Live rendering | Ultra Sessions updates current drawings and handles rollover. Stored bar timestamps anchor deferred boxes. |
| Alerts and other libraries | Host owns signal decisions, alert calls, round levels and visual composition. |

The existing API includes drawing setters/deleters and a conditional request wrapper. These are specific exceptions to the shared compute-only/global-request conventions, as implemented in the source. Preserve their documented contracts; do not move drawing creation, inputs, alerts or consumer-specific policy into the library during unrelated changes.

## Rules And Pitfalls

| Rule | Detail |
|---|---|
| Persist independent state | Use `var` configurations and one newest-first array per session. UDT accessors return references, not copies. |
| Pass configuration explicitly | Construct descriptors from arguments. Do not restore mutable global preset objects accessed by exports. |
| Strict engine session syntax | Use exactly `HHMM-HHMM`, valid hours and minutes. Invalid parsing returns `na`; equal endpoints mean 24 hours. Weekday masks and split periods are not supported by this parser. Native membership wrappers have a separate contract. |
| Preset timezone ownership | Presets use their native timezone; custom timezone applies to Custom. IANA identifiers handle seasonal offsets. Presets are schedule envelopes, not holiday or lunch-break calendars. |
| Session prices and line expiry are independent | Session OHLC uses session boundaries only. `closeTime` is exclusive; `sessEndTime` is close minus 1 ms. Explicit line endpoints use the requested clock time. Preserve the Ultra Sessions consumer's bar-edge rendering convention. |
| React to observed gaps | Default long-gap threshold is one day between previous close and current open. No future market-calendar prediction. Endpoints strictly inside a gap shift by the gap's local calendar-date span, preserving wall-clock time. |
| Resume interrupted sessions | A session started before the gap and closing inside it retains its identity and prices with an extended close. Suppress overlapping scheduled starts until it ends. A session already finished before the gap is not resumed. |
| Preserve field-level `varip` | `closeTime`, `sessEndTime`, and `lineEndTime` survive first-tick gap adjustment. Do not make all price state intrabar-persistent as a blanket rollback fix. |
| Request selectively, execute consistently | Call the wrapper each bar; the actual one-minute request is conditional. Boundary detection applies to intraday charts above one minute. Share requests between sessions. |
| Missing minute data stays missing | Empty scans leave prices unchanged, possibly `na`. Never substitute a whole partial candle. Partial available history may yield incomplete OHLC; there is no completeness flag. |
| Intrabar arrays must align | Custom `IntrabarData` must contain chronological, equally sized time/OHLC arrays. Scanner does not validate alignment. Request history is a minute-bar budget, not guaranteed calendar coverage. |
| Standard chart assumptions | Intended for time-based intraday charts. Daily and higher charts do not run the state engine. Nonstandard chart behavior is not certified. Replay-clock detection is a heuristic. |
| Count records explicitly | Pass `int(na)` as engine days limit to process loaded history. `trimSessionStates` keeps at least one record and removes references only. It neither filters empty records nor deletes drawings. |
| Ultra Sessions history has separate policies | Regional storage budgets N+1; daily N+2 supports previous-day levels. Object budgets may cap N. Daily identifiers count daily bars; trade-window history remains calendar based. |
| Partial initial history | Non-daily sessions can initialize mid-session from available bars. Daily sessions do not use that late-start path. Loaded data cannot reconstruct absent prices. |
| Timestamp coordinates | Host creates time-based objects with `xloc.bar_time`. Do not apply the bar-index 500-future-bar limit to timestamps. Setters do not change coordinate mode. |
| Setter omissions | `na` means leave a property unchanged. Use native setters if the desired operation is clearing a value to `na`. Arrow line styles are not box-border styles. |
| Cleanup side effects | `clearDrawings` deletes objects and empties supplied arrays. `trimPool` pops oldest entries from newest-first pools. `toChartPoints` uses the shorter input length. |
| Units and heuristics | Timestamps/durations are milliseconds; named day/minute counts are counts. Forex pip default is mintick times ten; use a feed-specific positive override. Quantity-step defaults are heuristics. |
| Risk ownership | Account-risk sizing and trade risk/reward calculations belong in the separate risk library. Quantity rounding and notional conversion remain general utilities. |
| Time helper limits | `sessionDurationMs` is nominal duration. `barDurationMs` is nominal where timeframe seconds exist. `timezoneOffsetMs` uses date-based midnight construction, not a universal instantaneous DST-offset calculation. |
| Publication verification | Match local code to the final import version, compile consumers in TradingView and replay boundaries/gaps. Local documentation checks do not certify Pine runtime behavior. |

See [the guide](Pine-Script-Utilities-Guide.md) for lifecycle reasoning and [publication copy](publication-document.txt) for TradingView-formatted text.
