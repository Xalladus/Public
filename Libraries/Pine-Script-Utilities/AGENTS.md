# AGENTS.md - Libraries/Pine-Script-Utilities

Pine Script v6 utility and session-state library. The library owns enums, validation, conversions, time calculations, conditional intrabar retrieval, and session mutation. The host owns inputs, persistent arrays, drawing creation, retention policy, alerts, and cross-library composition. Existing drawing setters and cleanup exports mutate or delete host-owned objects; they never create them. `session-example.pine` is the advanced indicator consumer.

Current checked-in test import, not a claim that every local edit is published:

```pine
import OneCleverGuy/UtilityLibrary1CGTESTA/12 as UTIL
```

The library imports nothing. The example also imports `OneCleverGuy/RoundNumberLibrary/3 as RNL`. Reconcile the permanent title and published version before release; do not invent version numbers. Standards: [coding style](../../docs/style-guide.md), [AGENTS](../../docs/agents-file-standard.md), [guide](../../docs/guide-standards.md), [publication markup](../../docs/tradingview-publication-markup.md).

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

### `Timezones`

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

### `Hours`

| Member | Display / meaning |
|---|---|
| `h0` | 00 |
| `h1` | 01 |
| `h2` | 02 |
| `h3` | 03 |
| `h4` | 04 |
| `h5` | 05 |
| `h6` | 06 |
| `h7` | 07 |
| `h8` | 08 |
| `h9` | 09 |
| `h10` | 10 |
| `h11` | 11 |
| `h12` | 12 |
| `h13` | 13 |
| `h14` | 14 |
| `h15` | 15 |
| `h16` | 16 |
| `h17` | 17 |
| `h18` | 18 |
| `h19` | 19 |
| `h20` | 20 |
| `h21` | 21 |
| `h22` | 22 |
| `h23` | 23 |

### `Minutes`

| Member | Display / meaning |
|---|---|
| `m0` | 00 |
| `m1` | 01 |
| `m2` | 02 |
| `m3` | 03 |
| `m4` | 04 |
| `m5` | 05 |
| `m6` | 06 |
| `m7` | 07 |
| `m8` | 08 |
| `m9` | 09 |
| `m10` | 10 |
| `m11` | 11 |
| `m12` | 12 |
| `m13` | 13 |
| `m14` | 14 |
| `m15` | 15 |
| `m16` | 16 |
| `m17` | 17 |
| `m18` | 18 |
| `m19` | 19 |
| `m20` | 20 |
| `m21` | 21 |
| `m22` | 22 |
| `m23` | 23 |
| `m24` | 24 |
| `m25` | 25 |
| `m26` | 26 |
| `m27` | 27 |
| `m28` | 28 |
| `m29` | 29 |
| `m30` | 30 |
| `m31` | 31 |
| `m32` | 32 |
| `m33` | 33 |
| `m34` | 34 |
| `m35` | 35 |
| `m36` | 36 |
| `m37` | 37 |
| `m38` | 38 |
| `m39` | 39 |
| `m40` | 40 |
| `m41` | 41 |
| `m42` | 42 |
| `m43` | 43 |
| `m44` | 44 |
| `m45` | 45 |
| `m46` | 46 |
| `m47` | 47 |
| `m48` | 48 |
| `m49` | 49 |
| `m50` | 50 |
| `m51` | 51 |
| `m52` | 52 |
| `m53` | 53 |
| `m54` | 54 |
| `m55` | 55 |
| `m56` | 56 |
| `m57` | 57 |
| `m58` | 58 |
| `m59` | 59 |

### `QuarterHours`

| Member | Display / meaning |
|---|---|
| `t0000` | 00:00 |
| `t0015` | 00:15 |
| `t0030` | 00:30 |
| `t0045` | 00:45 |
| `t0100` | 01:00 |
| `t0115` | 01:15 |
| `t0130` | 01:30 |
| `t0145` | 01:45 |
| `t0200` | 02:00 |
| `t0215` | 02:15 |
| `t0230` | 02:30 |
| `t0245` | 02:45 |
| `t0300` | 03:00 |
| `t0315` | 03:15 |
| `t0330` | 03:30 |
| `t0345` | 03:45 |
| `t0400` | 04:00 |
| `t0415` | 04:15 |
| `t0430` | 04:30 |
| `t0445` | 04:45 |
| `t0500` | 05:00 |
| `t0515` | 05:15 |
| `t0530` | 05:30 |
| `t0545` | 05:45 |
| `t0600` | 06:00 |
| `t0615` | 06:15 |
| `t0630` | 06:30 |
| `t0645` | 06:45 |
| `t0700` | 07:00 |
| `t0715` | 07:15 |
| `t0730` | 07:30 |
| `t0745` | 07:45 |
| `t0800` | 08:00 |
| `t0815` | 08:15 |
| `t0830` | 08:30 |
| `t0845` | 08:45 |
| `t0900` | 09:00 |
| `t0915` | 09:15 |
| `t0930` | 09:30 |
| `t0945` | 09:45 |
| `t1000` | 10:00 |
| `t1015` | 10:15 |
| `t1030` | 10:30 |
| `t1045` | 10:45 |
| `t1100` | 11:00 |
| `t1115` | 11:15 |
| `t1130` | 11:30 |
| `t1145` | 11:45 |
| `t1200` | 12:00 |
| `t1215` | 12:15 |
| `t1230` | 12:30 |
| `t1245` | 12:45 |
| `t1300` | 13:00 |
| `t1315` | 13:15 |
| `t1330` | 13:30 |
| `t1345` | 13:45 |
| `t1400` | 14:00 |
| `t1415` | 14:15 |
| `t1430` | 14:30 |
| `t1445` | 14:45 |
| `t1500` | 15:00 |
| `t1515` | 15:15 |
| `t1530` | 15:30 |
| `t1545` | 15:45 |
| `t1600` | 16:00 |
| `t1615` | 16:15 |
| `t1630` | 16:30 |
| `t1645` | 16:45 |
| `t1700` | 17:00 |
| `t1715` | 17:15 |
| `t1730` | 17:30 |
| `t1745` | 17:45 |
| `t1800` | 18:00 |
| `t1815` | 18:15 |
| `t1830` | 18:30 |
| `t1845` | 18:45 |
| `t1900` | 19:00 |
| `t1915` | 19:15 |
| `t1930` | 19:30 |
| `t1945` | 19:45 |
| `t2000` | 20:00 |
| `t2015` | 20:15 |
| `t2030` | 20:30 |
| `t2045` | 20:45 |
| `t2100` | 21:00 |
| `t2115` | 21:15 |
| `t2130` | 21:30 |
| `t2145` | 21:45 |
| `t2200` | 22:00 |
| `t2215` | 22:15 |
| `t2230` | 22:30 |
| `t2245` | 22:45 |
| `t2300` | 23:00 |
| `t2315` | 23:15 |
| `t2330` | 23:30 |
| `t2345` | 23:45 |

### `Duration`

| Member | Display / meaning |
|---|---|
| `Minute` | Minute |
| `QuarterHour` | 15 Minutes |
| `HalfHour` | 30 Minutes |
| `Hour` | Hour |
| `FourHours` | 4 Hours |
| `EightHours` | 8 Hours |
| `TwelveHours` | 12 Hours |
| `Day` | Day |
| `Week` | Week |

### `AssetClass`

| Member | Display / meaning |
|---|---|
| `Forex` | Forex |
| `Crypto` | Crypto |
| `Futures` | Futures |
| `Stock` | Stock |
| `Index` | Index |
| `CFD` | CFD |
| `Fund` | Fund |
| `Bond` | Bond |
| `Economic` | Economic |
| `Other` | Other |

### `LineStyle`

| Member | Display / meaning |
|---|---|
| `solid` | Solid (─) |
| `dotted` | Dotted (┈) |
| `dashed` | Dashed (╌) |
| `lArrow` | Left Arrow (<─) |
| `rArrow` | Right Arrow (─>) |
| `bArrow` | Both Arrows (<─>) |

### `LineSize`

| Member | Display / meaning |
|---|---|
| `thin` | 1px (thin) |
| `normal` | 2px (normal) |
| `heavy` | 3px (heavy) |
| `thick` | 4px (thick) |
| `wide` | 5px (wide) |

### `TextSize`

| Member | Display / meaning |
|---|---|
| `auto` | Auto |
| `tiny` | Tiny |
| `small` | Small |
| `normal` | Normal |
| `large` | Large |
| `huge` | Huge |

### `HorizontalAlign`

| Member | Display / meaning |
|---|---|
| `left` | Left |
| `center` | Center |
| `right` | Right |

### `VerticalAlign`

| Member | Display / meaning |
|---|---|
| `top` | Top |
| `center` | Center |
| `bottom` | Bottom |

### `LineExtend`

| Member | Display / meaning |
|---|---|
| `none` | None |
| `right` | Right |
| `left` | Left |
| `both` | Both |

### `LabelStyle`

| Member | Display / meaning |
|---|---|
| `center` | Center |
| `down` | Down |
| `left` | Left |
| `right` | Right |
| `up` | Up |
| `lowLeft` | Low Left |
| `lowRight` | Low Right |
| `upperLeft` | Upper Left |
| `upperRight` | Upper Right |

### `SessionPreset`

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

### `SessionAnchor`

| Member | Display / meaning |
|---|---|
| `SessionEnd` | Session End |
| `SessionOpen` | Session Open |
| `ExtremeTime` | Extreme Time |

Timezone offsets in display strings are descriptive. Use IANA timezone names for DST. Presets keep their native timezones; only Custom uses the supplied custom timezone. Japan and Hong Kong presets are continuous envelopes including midday recesses, not full exchange calendars.

## Exported Types

Persist SessionConfig and one array<SessionState> per configured session with var. Returned UDTs are references. Engine-owned session fields should be read by the host, not independently rewritten.

### `IntrabarScan`

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

| Field | Type | Default | Meaning |
|---|---|---|---|
| `times` | `array<int>` | `required` | One-minute bar opening timestamps. |
| `opens` | `array<float>` | `required` | One-minute opening prices. |
| `highs` | `array<float>` | `required` | One-minute high prices. |
| `lows` | `array<float>` | `required` | One-minute low prices. |
| `closes` | `array<float>` | `required` | One-minute closing prices. |

### `SessionInfo`

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

| Field | Type | Default | Meaning |
|---|---|---|---|
| `startTime` | `int` | `na` | Window start UNIX time. |
| `endTime` | `int` | `na` | Scheduled window end UNIX time. |
| `isInWindow` | `bool` | `false` | True when the current bar is inside the window. |
| `alreadyDrawn` | `bool` | `false` | True when this window start was already drawn by the host. |
| `shouldDraw` | `bool` | `false` | True when the host should draw the window boundaries now. |

## Function Reference

Exact argument types and defaults appear in Public API. Optional arguments are those with defaults. Drawing setters use na to mean unchanged; cleanup functions delete objects and mutate their arrays.

### `toTimezone`

Resolves a timezone selection into a string usable by time() and timestamp().

| Argument | Type | Meaning |
|---|---|---|
| `this` | `Timezones` | Timezone enum value. |

Returns: (string) - IANA timezone name, or the exchange timezone for Timezones.exch.

### `toHourInt`

Converts an hour selection into an integer hour.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `Hours` | Hour enum value. |

Returns: (int) - Hour in 24-hour format.

### `toMinuteInt`

Converts a minute selection into an integer minute.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `Minutes` | Minute enum value. |

Returns: (int) - Minute value from 0 to 59.

### `toHhmm`

Converts a quarter-hour selection into a compact time token.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `QuarterHours` | Quarter-hour enum value. |

Returns: (string) - Four-character time token. Example: "1430".

### `toLineStyle`

Converts a line style selection into a Pine line style constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `LineStyle` | Line style enum value. |

Returns: (string) - Pine line style constant.

### `toLineWidth`

Converts a line size selection into a pixel width.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `LineSize` | Line size enum value. |

Returns: (int) - Width in pixels.

### `toTextSizeString`

Converts a text size selection into a Pine size constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `TextSize` | Text size enum value. |

Returns: (string) - Pine size constant.

### `toHorizontalAlign`

Converts a horizontal alignment selection into a Pine text align constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `HorizontalAlign` | Horizontal alignment enum value. |

Returns: (string) - Pine text align constant.

### `toVerticalAlign`

Converts a vertical alignment selection into a Pine text align constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `VerticalAlign` | Vertical alignment enum value. |

Returns: (string) - Pine text align constant.

### `toLineExtend`

Converts a line extension selection into a Pine extend constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `LineExtend` | Line extension enum value. |

Returns: (string) - Pine extend constant.

### `toLabelStyle`

Converts a label style selection into a Pine label style constant.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `LabelStyle` | Label style enum value. |

Returns: (string) - Pine label style constant.

### `updateLine`

Updates any combination of line properties in one call.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `line` | Line being updated. |
| `_x1` | `int` | New first point x value. |
| `_y1` | `float` | New first point price. |
| `_x2` | `int` | New second point x value. |
| `_y2` | `float` | New second point price. |
| `_extend` | `string` | New extend constant. |
| `_color` | `color` | New line color. |
| `_style` | `string` | New line style constant. |
| `_width` | `int` | New line width in pixels. |

Returns: (line) - The same line, to allow chaining.

### `updateLabel`

Updates any combination of label properties in one call.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `label` | Label being updated. |
| `_x` | `int` | New x value. |
| `_y` | `float` | New price, used when yloc is yloc.price. |
| `_text` | `string` | New label text. |
| `_yloc` | `string` | New yloc constant. |
| `_color` | `color` | New label background color. |
| `_style` | `string` | New label style constant. |
| `_textColor` | `color` | New text color. |
| `_textSize` | `string` | New text size constant. |
| `_textAlign` | `string` | New text alignment constant. |
| `_tooltip` | `string` | New tooltip text. |
| `_fontFamily` | `string` | New font family constant. |

Returns: (label) - The same label, to allow chaining.

### `updateBox`

Updates any combination of box properties in one call.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `box` | Box being updated. |
| `_left` | `int` | New left edge x value. |
| `_top` | `float` | New top edge price. |
| `_right` | `int` | New right edge x value. |
| `_bottom` | `float` | New bottom edge price. |
| `_borderColor` | `color` | New border color. |
| `_borderWidth` | `int` | New border width in pixels. |
| `_borderStyle` | `string` | New border style constant. |
| `_extend` | `string` | New extend constant. |
| `_bgColor` | `color` | New background color. |
| `_text` | `string` | New box text. |
| `_textSize` | `string` | New text size constant. |
| `_textColor` | `color` | New text color. |
| `_textHAlign` | `string` | New horizontal text alignment constant. |
| `_textVAlign` | `string` | New vertical text alignment constant. |
| `_textWrap` | `string` | New text wrap constant. |
| `_fontFamily` | `string` | New font family constant. |

Returns: (box) - The same box, to allow chaining.

### `updateTableCell`

Updates any combination of properties on an existing table cell.

| Argument | Type | Meaning |
|---|---|---|
| `this` | `table` | Table that already contains the cell. |
| `_column` | `int` | Zero-based column index. |
| `_row` | `int` | Zero-based row index. |
| `_text` | `string` | New cell text. |
| `_textColor` | `color` | New text color. |
| `_textSize` | `string` | New text size constant. |
| `_bgColor` | `color` | New cell background color. |
| `_textHAlign` | `string` | New horizontal text alignment constant. |
| `_textVAlign` | `string` | New vertical text alignment constant. |
| `_tooltip` | `string` | New cell tooltip. |
| `_fontFamily` | `string` | New font family constant. |

Returns: (table) - The same table, to allow chaining.

### `detectAssetClass`

Resolves the chart symbol into a normalized asset class.

Returns: (AssetClass) - Normalized asset class for the current symbol.

### `priceDecimals`

Returns the number of decimal places implied by the symbol tick size.

Returns: (int) - Decimal places, clamped to zero or more.

### `tickValue`

Returns the money value of one tick for one contract or unit.

Returns: (float) - Tick value in the instrument's quote currency.

### `priceToTicks`

Converts a price distance into ticks.

| Argument | Type | Meaning |
|---|---|---|
| `_priceDistance` | `float` | Distance in price units. |

Returns: (float) - Distance in ticks.

### `ticksToPrice`

Converts a tick distance into price units.

| Argument | Type | Meaning |
|---|---|---|
| `_ticks` | `float` | Distance in ticks. |

Returns: (float) - Distance in price units.

### `pipSize`

Returns the price distance of one pip.

| Argument | Type | Meaning |
|---|---|---|
| `_pipSizeOverride` | `float` | Optional pip price distance. na uses the default; nonpositive values return na. |

Returns: (float) - Override when supplied; otherwise ten ticks on forex symbols, one tick elsewhere. Override for feeds without fractional pips.

### `priceToPips`

Converts a price distance into pips.

| Argument | Type | Meaning |
|---|---|---|
| `_priceDistance` | `float` | Distance in price units. |
| `_pipSizeOverride` | `float` | Optional pip price distance, passed to pipSize(). |

Returns: (float) - Distance in pips.

### `pipsToPrice`

Converts a pip distance into price units.

| Argument | Type | Meaning |
|---|---|---|
| `_pips` | `float` | Distance in pips. |
| `_pipSizeOverride` | `float` | Optional pip price distance, passed to pipSize(). |

Returns: (float) - Distance in price units.

### `defaultQuantityStep`

Returns a sensible quantity increment for the current asset class.

Returns: (float) - 1.0 for whole-unit markets, 0.01 for crypto. Override per venue when needed.

### `roundQuantity`

Rounds a position size down to a tradable increment.

| Argument | Type | Meaning |
|---|---|---|
| `_quantity` | `float` | Unrounded position size. |
| `_step` | `float` | Quantity increment. Futures and equities use 1.0. |

Returns: (float) - Nonnegative quantity rounded down to the increment.

### `positionNotional`

Returns the notional exposure of a position.

| Argument | Type | Meaning |
|---|---|---|
| `_quantity` | `float` | Position size in broker units. |
| `_price` | `float` | Price used for the valuation. |

Returns: (float) - Notional value in the instrument's quote currency.

### `durationMs`

Converts a named duration into milliseconds.

| Argument | Type | Meaning |
|---|---|---|
| `_duration` | `Duration` | Duration enum value. |

Returns: (int) - Length in milliseconds.

### `timezoneOffsetMs`

Measures the offset between a timezone and UTC on the current chart day.

| Argument | Type | Meaning |
|---|---|---|
| `_timezone` | `string` | IANA timezone name. |

Returns: (int) - Offset in milliseconds. Negative west of UTC. Divide by 3600000 for hours.

### `hhmmFromParts`

Combines hour and minute selections into a compact time token.

| Argument | Type | Meaning |
|---|---|---|
| `_hour` | `Hours` | Hour enum value. |
| `_minute` | `Minutes` | Minute enum value. |

Returns: (string) - Four-character time token. Example: "0930".

### `hhmmToParts`

Splits a compact time token into hour and minute integers.

| Argument | Type | Meaning |
|---|---|---|
| `_hhmm` | `string` | Four-character time token. |

Returns: ([int, int]) - Hour and minute, or [na, na] when the token is invalid.

### `hhmmToMinutes`

Converts a compact time token into minutes from midnight.

| Argument | Type | Meaning |
|---|---|---|
| `_hhmm` | `string` | Four-character time token. |

Returns: (int) - Minutes from midnight, or na when the token is invalid.

### `minutesToHhmm`

Converts minutes from midnight into a compact time token.

| Argument | Type | Meaning |
|---|---|---|
| `_minutesOfDay` | `int` | Minutes from midnight. Values outside one day wrap. |

Returns: (string) - Four-character time token, or na when the input is na.

### `barDurationMs`

Returns the length of one chart bar in milliseconds.

Returns: (int) - Bar length in milliseconds.

### `barDayBoundaryOffsetMs`

Returns the day offset needed when a wall-clock target occurs after local midnight inside this bar.

| Argument | Type | Meaning |
|---|---|---|
| `_timezone` | `string` | Timezone whose calendar boundary is tested. |
| `_hour` | `int` | Wall-clock target hour. |
| `_minute` | `int` | Wall-clock target minute. |

Returns: (int) - One nominal day when tomorrow's target occurs inside this bar, otherwise 0.

### `clearDrawings`

Deletes drawings and empties any combination of supplied drawing pools.

| Argument | Type | Meaning |
|---|---|---|
| `lines` | `array<line>` | Optional line pool to clear. |
| `labels` | `array<label>` | Optional label pool to clear. |
| `boxes` | `array<box>` | Optional box pool to clear. |
| `polylines` | `array<polyline>` | Optional polyline pool to clear. |

Returns: (int) - Total count of drawings deleted across all supplied pools.

### `trimPool`

Trims any combination of drawing pools down to a maximum size (newest first).

| Argument | Type | Meaning |
|---|---|---|
| `maxSize` | `int` | Maximum number of objects to retain in each supplied pool. |
| `lines` | `array<line>` | Optional line pool to trim. |
| `labels` | `array<label>` | Optional label pool to trim. |
| `boxes` | `array<box>` | Optional box pool to trim. |
| `polylines` | `array<polyline>` | Optional polyline pool to trim. |

Returns: (int) - Total count of drawings remaining across all supplied pools.

### `toChartPoints`

Builds a chart.point array for polyline and multi-point drawing calls.

| Argument | Type | Meaning |
|---|---|---|
| `_times` | `array<int>` | UNIX times, one per point. |
| `_prices` | `array<float>` | Prices, one per point. |

Returns: (array<chart.point>) - Points built from the shorter of the two input arrays.

### `sessionToParts`

Parses a session string into its hour and minute components.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `string` | Exactly one "HHMM-HHMM" window. Day suffixes and multiple windows are unsupported. |

Returns: ([int, int, int, int]) - Start hour, start minute, end hour, end minute; all na when invalid.

### `sessionDurationMs`

Calculates the length of a session string in milliseconds.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `string` | Session string in "HHMM-HHMM" format. |

Returns: (int) - Session length in milliseconds, or na when invalid. Identical valid start and end times return 24 hours.

### `getObservedLongGap`

Detects a long interruption immediately before the current chart bar.

| Argument | Type | Meaning |
|---|---|---|
| `_minimumGapMs` | `int` | Smallest qualifying interruption in milliseconds. Defaults to one day. |

Returns: ([bool, int, int, int]) - Detection flag, previous bar close, current bar open, and observed duration.

### `historyCutoffTime`

Returns the oldest timestamp a script should process, counted in calendar days.

| Argument | Type | Meaning |
|---|---|---|
| `_calendarDays` | `int` | Number of calendar days of history to allow. |
| `_referenceTime` | `int` | Newest chart timestamp used as the history anchor. |

Returns: (int) - Cutoff UNIX timestamp.

### `isWithinHistoryWindow`

Reports whether the current bar is inside the allowed calendar history window.

| Argument | Type | Meaning |
|---|---|---|
| `_calendarDays` | `int` | Number of calendar days of history to allow. |
| `_referenceTime` | `int` | Newest chart timestamp used as the history anchor. |
| `_extraDays` | `int` | Additional calendar days of slack for sessions that span days. |

Returns: (bool) - True when the bar should be processed.

### `resolveSessionInfo`

Resolves a preset, or builds a descriptor from a custom session string.

| Argument | Type | Meaning |
|---|---|---|
| `_preset` | `SessionPreset` | Preset selection, or SessionPreset.Custom. |
| `_customSession` | `string` | Session string in "HHMM-HHMM" format, required when the preset is Custom. |
| `_customLabel` | `string` | Label applied when the preset is Custom. Defaults to "Custom". |
| `_customTimezone` | `string` | IANA timezone for a Custom session. na uses the exchange timezone. |

Returns: (SessionInfo) - Resolved descriptor with timezone, or na when a Custom session string is missing or invalid.

### `isInSession`

Reports whether the current bar falls inside a session.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `string` | Session string in TradingView format. |
| `_timezone` | `string` | IANA timezone used for the session test. |

Returns: (bool) - True when the current bar is inside the session.

### `isInAnySession`

Reports whether the current bar falls inside any supplied session.

| Argument | Type | Meaning |
|---|---|---|
| `_sessions` | `array<string>` | Session strings to test. |
| `_timezone` | `string` | IANA timezone used for the session tests. |

Returns: (bool) - True when at least one session contains the current bar. False on an empty array.

### `getSessionStartTime`

Returns the session open timestamp for the current local session day.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `SessionInfo` | Session descriptor. Its timezone is used. |
| `_dayOffsetMs` | `int` | Nominal-day token from barDayBoundaryOffsetMs() or a manual whole-day shift. |

Returns: (int) - Session open UNIX time.

### `isSessionFirstBar`

Reports whether the current bar contains the session open.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `SessionInfo` | Session descriptor. |
| `_dayOffsetMs` | `int` | Nominal-day token from barDayBoundaryOffsetMs() or a manual whole-day shift. |

Returns: (bool) - True when the session opens inside this bar.

### `isSessionBoundaryInBar`

Reports whether a session open or close falls inside a bar interval.

| Argument | Type | Meaning |
|---|---|---|
| `_isStart` | `bool` | True tests the session open, false tests the session close. |
| `_session` | `SessionInfo` | Session descriptor. |
| `_barStartTime` | `int` | Bar start UNIX time. |
| `_barEndTime` | `int` | Bar end UNIX time. |
| `_dayOffsetMs` | `int` | Nominal-day token from barDayBoundaryOffsetMs() or a manual whole-day shift. |

Returns: (bool) - True when the requested boundary falls strictly inside the bar.

### `needsSessionIntrabars`

Tests whether either session boundary falls strictly inside the current chart candle.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `SessionInfo` | Resolved session descriptor. |

Returns: (bool) - True when one-minute filtering is needed. False on one-minute or smaller charts.

### `requestIntrabarData`

Requests one shared set of one-minute arrays for the current chart bar.

| Argument | Type | Meaning |
|---|---|---|
| `_historyDays` | `simple int` | Calendar days of one-minute history the host may need. |
| `_required` | `bool` | Whether this candle needs filtering. Call every bar; the request runs conditionally inside. |

Returns: (IntrabarData) - Shared one-minute OHLC arrays, empty when skipped. Initializes the dataset on the last confirmed historical bar for realtime use.

### `scanIntrabarRange`

Aggregates one-minute data inside a time range of the current chart bar.

| Argument | Type | Meaning |
|---|---|---|
| `_intrabarData` | `IntrabarData` | Shared one-minute arrays returned by requestIntrabarData(). |
| `_startTime` | `int` | Inclusive range start UNIX time. |
| `_endTime` | `int` | Exclusive range end UNIX time. |
| `_seedHigh` | `float` | Existing high to beat. Pass na to take the first value found. |
| `_seedLow` | `float` | Existing low to beat. Pass na to take the first value found. |
| `_seedHighTime` | `int` | High timestamp kept when no new high is found. |
| `_seedLowTime` | `int` | Low timestamp kept when no new low is found. |

Returns: (IntrabarScan) - Aggregated open, high, low, close, extreme times, and a data flag.

### `runSessionEngine`

Creates and maintains session lifecycle state for one configured session.

| Argument | Type | Meaning |
|---|---|---|
| `_config` | `SessionConfig` | Session configuration supplied by the host script. |
| `_states` | `array<SessionState>` | State storage for this session. Newest state is index 0. |
| `_intrabarData` | `IntrabarData` | Shared one-minute arrays returned by requestIntrabarData(). |
| `_timeNow` | `int` | Current UNIX timestamp supplied by the host. |
| `_daysLimit` | `int` | Calendar days of history to process. Pass na to process all loaded bars and let the host retain records by count. |

Returns: (SessionInfo) - Resolved descriptor including its timezone, or na when the configuration cannot be resolved.

### `getActiveSession`

Returns the session that is currently in progress.

| Argument | Type | Meaning |
|---|---|---|
| `_states` | `array<SessionState>` | Session state storage. |
| `_timeNow` | `int` | Current UNIX timestamp. |

Returns: (SessionState) - In-progress session state, or na when no session is open.

### `getCompletedSession`

Returns a completed session counted back from the most recent one.

| Argument | Type | Meaning |
|---|---|---|
| `_states` | `array<SessionState>` | Session state storage. |
| `_sessionsBack` | `int` | Zero-based offset. 0 is the most recently completed session. |
| `_timeNow` | `int` | Current UNIX timestamp used to exclude in-progress sessions. |

Returns: (SessionState) - Requested completed session state, or na when unavailable.

### `trimSessionStates`

Drops the oldest session states so storage stays bounded.

| Argument | Type | Meaning |
|---|---|---|
| `_states` | `array<SessionState>` | Session state storage. |
| `_maxSessions` | `int` | Maximum number of sessions to retain. |

Returns: (int) - Number of states remaining after trimming.

### `planTradeWindow`

Computes scheduled trade-window boundaries with realtime gating.

| Argument | Type | Meaning |
|---|---|---|
| `_session` | `string` | Window session string in "HHMM-HHMM" format. |
| `_timezone` | `string` | IANA timezone the window is expressed in. |
| `_daysLimit` | `int` | Calendar days of history the host may draw. |
| `_timeNow` | `int` | Current UNIX timestamp. |
| `_lastDrawnStart` | `int` | Start timestamp the host most recently drew, or na. |

Returns: (TradeWindowPlan) - Window boundaries plus in-window, already-drawn, and should-draw flags.

## Function Hierarchy

```text
UTIL
|-- Configuration and representation
|   |-- Enum conversion methods
|   |-- Price, tick, pip and quantity helpers
|   +-- HHMM parsing and calendar helpers
|-- Session pipeline (host calls each bar)
|   |-- resolveSessionInfo -> presetToSessionInfo / buildSessionInfo
|   |-- needsSessionIntrabars -> isSessionBoundaryInBar
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
+-- session-example.pine (consumer, not imported by library)
    |-- Persistent configurations, state arrays and visual trackers
    |-- Shared intrabar request -> engine per enabled session
    |-- pruneExpiredSession (record count and drawing cleanup)
    |-- renderRetainedSessionHistory (historical snapshot)
    +-- syncSessionVisuals / renderSessionStates (live drawings)
```

## Standard Integration Pattern

This uses the checked-in test import. Verify that the imported publication contains the local implementation before testing. Eight retained records here means at most eight total records, including a developing session; it is not the example's separate previous-session input.

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

For multiple sessions, OR the enabled sessions' boundary checks, request once, and pass the same arrays to each engine call. Persist a separate state array per session. If drawings are attached to records, replace simple trimming with host cleanup before dropping each record.

## Execution Ownership

| Phase | Owner and behavior |
|---|---|
| Initialization | Host declares inputs, persistent configurations, descriptors and arrays. Library has no imports or input declarations. |
| Every chart bar | Host checks boundaries and calls the shared request wrapper, then the engine for each enabled session. Engine mutates arrays and UDTs in place. |
| Historical request initialization | Wrapper also requests on `barstate.islastconfirmedhistory` on eligible timeframes, even when no boundary needs scanning. Preserve this preparation for realtime requests. |
| First update after long gap | Engine adjusts retained endpoints when the gap is observed. Three `varip` endpoint fields preserve these first-update changes. Price fields keep ordinary rollback behavior. |
| Retention | Host chooses record counts or a calendar processing window. Engine does not trim storage automatically. |
| Historical rendering | Example computes history first, then creates retained drawings on the last confirmed historical bar. |
| Live rendering | Example updates current drawings and handles rollover. Stored bar timestamps anchor deferred boxes. |
| Alerts and other libraries | Host owns signal decisions, alert calls, round levels and visual composition. |

The drawing setters/deleters and conditional request wrapper are existing, session-authorized exceptions to the broader repository compute-only/global-request conventions. Preserve their documented contracts; do not move drawing creation, inputs, alerts or example-specific policy into the library during unrelated changes.

## Rules And Pitfalls

| Rule | Detail |
|---|---|
| Persist independent state | Use `var` configurations and one newest-first array per session. UDT accessors return references, not copies. |
| Pass configuration explicitly | Construct descriptors from arguments. Do not restore mutable global preset objects accessed by exports. |
| Strict engine session syntax | Use exactly `HHMM-HHMM`, valid hours and minutes. Invalid parsing returns `na`; equal endpoints mean 24 hours. Weekday masks and split periods are not supported by this parser. Native membership wrappers have a separate contract. |
| Preset timezone ownership | Presets use their native timezone; custom timezone applies to Custom. IANA identifiers handle seasonal offsets. Presets are schedule envelopes, not holiday or lunch-break calendars. |
| Session prices and line expiry are independent | Session OHLC uses session boundaries only. `closeTime` is exclusive; `sessEndTime` is close minus 1 ms. Explicit line endpoints use the requested clock time. Preserve the example's bar-edge rendering convention. |
| React to observed gaps | Default long-gap threshold is one day between previous close and current open. No future market-calendar prediction. Endpoints strictly inside a gap shift by the gap's local calendar-date span, preserving wall-clock time. |
| Resume interrupted sessions | A session started before the gap and closing inside it retains its identity and prices with an extended close. Suppress overlapping scheduled starts until it ends. A session already finished before the gap is not resumed. |
| Preserve field-level `varip` | `closeTime`, `sessEndTime`, and `lineEndTime` survive first-tick gap adjustment. Do not make all price state intrabar-persistent as a blanket rollback fix. |
| Request selectively, execute consistently | Call the wrapper each bar; the actual one-minute request is conditional. Boundary detection applies to intraday charts above one minute. Share requests between sessions. |
| Missing minute data stays missing | Empty scans leave prices unchanged, possibly `na`. Never substitute a whole partial candle. Partial available history may yield incomplete OHLC; there is no completeness flag. |
| Intrabar arrays must align | Custom `IntrabarData` must contain chronological, equally sized time/OHLC arrays. Scanner does not validate alignment. Request history is a minute-bar budget, not guaranteed calendar coverage. |
| Standard chart assumptions | Intended for time-based intraday charts. Daily and higher charts do not run the state engine. Nonstandard chart behavior is not certified. Replay-clock detection is a heuristic. |
| Count records explicitly | Pass `int(na)` as engine days limit to process loaded history. `trimSessionStates` keeps at least one record and removes references only. It neither filters empty records nor deletes drawings. |
| Example history has separate policies | Regional storage budgets N+1; daily N+2 supports previous-day levels. Object budgets may cap N. Daily identifiers count daily bars; trade-window history remains calendar based. |
| Partial initial history | Non-daily sessions can initialize mid-session from available bars. Daily sessions do not use that late-start path. Loaded data cannot reconstruct absent prices. |
| Timestamp coordinates | Host creates time-based objects with `xloc.bar_time`. Do not apply the bar-index 500-future-bar limit to timestamps. Setters do not change coordinate mode. |
| Setter omissions | `na` means leave a property unchanged. Use native setters if the desired operation is clearing a value to `na`. Arrow line styles are not box-border styles. |
| Cleanup side effects | `clearDrawings` deletes objects and empties supplied arrays. `trimPool` pops oldest entries from newest-first pools. `toChartPoints` uses the shorter input length. |
| Units and heuristics | Timestamps/durations are milliseconds; named day/minute counts are counts. Forex pip default is mintick times ten; use a feed-specific positive override. Quantity-step defaults are heuristics. |
| Risk ownership | Account-risk sizing and trade risk/reward calculations belong in the separate risk library. Quantity rounding and notional conversion remain general utilities. |
| Time helper limits | `sessionDurationMs` is nominal duration. `barDurationMs` is nominal where timeframe seconds exist. `timezoneOffsetMs` uses date-based midnight construction, not a universal instantaneous DST-offset calculation. |
| Publication verification | Match local code to the final import version, compile consumers in TradingView and replay boundaries/gaps. Local documentation checks do not certify Pine runtime behavior. |

See [the guide](Pine-Script-Utilities-Guide.md) for lifecycle reasoning and [publication copy](publication-document.txt) for TradingView-formatted text.
