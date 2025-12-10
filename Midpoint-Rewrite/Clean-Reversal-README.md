# Clean Reversal Zones – How It Works

This document explains how the **Reversal zones 2 [ProJ]** Pine Script (v6) operates, what each section does, and how to interpret the output on your chart.

> File: `Clean Reversal - Sept.ps`  
> Pine: `//@version=6`  
> Indicator: `indicator("Reversal zones 2 [ProJ]", overlay = true, max_boxes_count = 500, max_labels_count = 500, max_lines_count = 500)`

---

## 1) What the indicator does (high‑level)

The script searches for **price relationships between candles** by computing **midpoints** between a *base price* on an earlier candle and each OHLC on the current/nearby candles. When it finds **exact midpoint matches** across specified lookback windows, it:
- Draws **reversal zones** (semi‑transparent boxes) spanning from the detected midpoint’s top to bottom.
- Draws a **dashed center line** through each box.
- Emits **structured log messages** that summarize all matches found on the current bar.

The logic runs for **two comparison containers**:
- **“One Candle”** (container “One”): compares the current candle against values one bar back.
- **“Two Candles”** (container “Two”): compares the current candle against values two bars back.

> In short: it’s a **pattern‑matching tool** that highlights zones where a midpoint derived from one candle’s price equals the midpoint to another candle’s OHLC, potentially signaling reversal areas.

---

## 2) Inputs

- `i_array_size (int)` – “How many zones to look back ?”  
  The number of historical rows kept for each container’s arrays (rolling buffers).
- `i_zone_color (color)` – Fill color for drawn boxes (zones).
- `i_zone_color2 (color)` – Color for the dashed **midline** drawn inside each box.
- `i_border_color (color)` – Border color for the boxes.

---

## 3) Data structures (UDTs)

### `type midpoints`
Holds midpoints from a **base value** to each of the OHLC prices on a target candle:
- `float vs_open`
- `float vs_high`
- `float vs_low`
- `float vs_close`

**Midpoint formula:**  
`(base + target_price) / 2`

### `type mid_container`
Rolling storage for each container (“One Candle”, “Two Candles”):
- `array<int> _bar`                 – the bar index backing each row.
- `array<midpoints> _open`          – midpoints from the base value to each **Open**.
- `array<midpoints> _high`          – midpoints from the base value to each **High**.
- `array<midpoints> _low`           – midpoints from the base value to each **Low**.
- `array<midpoints> _close`         – midpoints from the base value to each **Close**.
- `array<bool> _matched`            – per‑row flag that suppresses repeated work within a bar.
- `string _txt`                     – label for the container (“One Candle”, “Two Candles”).

Each container maintains **`i_array_size`** rows; new rows are appended each confirmed bar and oldest rows are shifted out.

---

## 4) Core functions (what they do)

### Midpoint creation
- `calc_midpoint(value1, value2)`: returns `(value1 + value2) / 2`.
- `create_mid_OHLC(base, bars_back)`: builds a `midpoints` record using `base` and the OHLC of `bars_back` candles ago.

### Price accessors
- `get_base_price(key, index)`: returns `open/high/low/close` for the **left‑hand** candle referenced by stored `_bar` (historical bar index).
- `get_alt_price(key, index, c)`: same idea but offsets to the **right‑hand** side of the comparison using a `container` offset (`c` = 1 for “One Candle”, `c` = 2 for “Two Candles”).

### Matching
- `get_candle_matches(value, candle_mids, container_text, part)`: checks if `value` equals any of `candle_mids.vs_open|high|low|close`. Returns a list of textual matches like `"Two Candles: High vs Close"`.
- `check_matches(value, mid_con, idx, live_con, live_p1, live_p2)`: aggregates matches across the stored **Open/High/Low/Close** arrays for a given row `idx`, building a “value text” header such as `"Container One: High vs Open"`.
- `check_mid_OHLC(...)`: for one **live** `midpoints` record, compare its `vs_open|high|low|close` to all stored `mid_con` rows.
- `check_con_midpoints(...)`: runs `check_mid_OHLC` against a single container.
- `check_all_midpoints(...)`: runs comparisons against **both** containers as long as their `_matched[idx]` flags are `false`.

### Drawing and alert text
- `show_matches_get_alert_text(value_text, matches, mid_con, n)`:
  - Uses parsed match strings to compute **top/bottom** for each zone.
  - Draws a **box** and a **dashed line** for every match.
  - Marks the row as matched: `mid_con._matched[n] = true`.
  - Returns a clean, multi‑line **alert text block**.
- `draw_matches_text(alerts_list, matches_arr, value_text, mid_con, n)`:
  - If matches exist, pushes the resulting alert block to `alerts_list`.

### Alert list management
- `manage_alert_list(active_alerts, master_list)`:
  - **Removes** stale alerts (present in `master_list` but not in `active_alerts`).
  - **Adds** new alerts (present in `active_alerts` but not yet in `master_list`).  
  - Returns: `(should_alert, new_match_locations)` so the caller can emit only **new** blocks this bar.

> Note: There are helper utilities (`array_logger`, `smart_remove_arr`, `parse_comparison_string`) that support formatting and deduping. Alerts are currently logged via `log.info(...)`; the `alert(...)` call is present but commented out.

---

## 5) Execution flow (per bar)

1. **Initialization (once)**  
   Two `mid_container` instances are created:
   - `one_candle_values` → label: `"One Candle"`
   - `two_candle_values` → label: `"Two Candles"`

2. **Live midpoints for this bar**  
   - **Box 1 (One Candle)**: `create_mid_OHLC(open/high/low/close, 1)`  
   - **Box 2 (Two Candles)**: `create_mid_OHLC(open/high/low/close, 2)`

3. **Reset per‑bar state**  
   - On `barstate.isnew`, `active_alerts` and `saved_alerts` are cleared.  
   - `_matched` flags for each container row are set to `false` (suppresses duplicate processing within the bar).

4. **Matching pass**  
   - Loop `n` from `0` to `i_array_size - 1`.  
   - For each `n`, call `check_all_midpoints(...)` twice: once with **One** (box 1 live mids), once with **Two** (box 2 live mids).  
   - Any matches draw **boxes + dashed midlines** immediately and contribute formatted text blocks to `active_alerts`.

5. **Alert aggregation & emission**  
   - `manage_alert_list(active_alerts, saved_alerts)` returns whether there are **new** blocks.  
   - If `should_alert`, the script concatenates the **new blocks** with blank lines between them and logs via `log.info("\n" + alert_text)`.

6. **Historical buffer update**  
   - On `barstate.isconfirmed`, both containers shift in the latest bar’s data via `update_mid_container_arrays(...)`:
     - `_bar.push(bar_index)`  
     - `_open/_high/_low/_close.push(<midpoints>)`  
     - `_matched.push(false)`  
     (Oldest rows are shifted out so arrays remain fixed length.)

---

## 6) Visual output

For each match found on the current bar:
- A **box** is drawn from `top = max(base, alt)` to `bottom = min(base, alt)` spanning from the **left‑hand bar** to the **current bar**.
- A **dashed line** is drawn at the midpoint `(top + bottom)/2` across the same span.
- Colors come from the Inputs:
  - `i_zone_color` for box fill,
  - `i_zone_color2` for dashed line,
  - `i_border_color` for box border.

These visuals are **ephemeral per bar**—they appear as the logic finds matches. The historical containers ensure past midpoints are available for new matches as time progresses.

---

## 7) Alerts and logging

- **`active_alerts`**: rebuilt on each bar; contains all alert blocks detected **this bar**.  
- **`saved_alerts`**: a per‑session “master list” of alert blocks used to detect **new vs stale** items between bars.  
- When there are new items, the script logs a clean, multi‑line block.  
- The TradingView `alert()` call is present but commented—uncomment to enable platform alerts:
  ```pine
  // alert(alert_text, alert.freq_once_per_bar)
  ```

---

## 8) Parameters that affect results

- **`i_array_size`**: Larger sizes keep more rows of historical midpoints per container. This increases both **coverage** and **compute**.  
- **Bar confirmation**: New rows are only added on `barstate.isconfirmed`, so **intrabar repainting** is minimized and history is stable.  
- **Bar timing**: Some checks are gated (e.g., “Open” on `barstate.isnew`, “Close” on `barstate.isconfirmed`) to match how OHLC values are finalized.

---

## 9) Known behaviors / notes

- The code comment `//container two values arent appearing.` suggests you may not see Two‑Candle matches under some conditions. Typical causes:
  - The exact midpoint equality is rare—small float differences prevent equality (`==`).  
    - **Mitigation:** introduce a **tolerance** check (e.g., `math.abs(a - b) <= syminfo.mintick * K`).
  - The `_matched[n]` flag can suppress repeated checks within a bar once **any** match was recorded for that row; review whether `_matched` should reset more granularly.
  - Ensure that `get_alt_price`’s right‑hand offset logic `(bar_index - index) + container` matches the intended candle pairing.

- Alert logs currently print only **new** blocks; if you expect persistent repeats, adjust `manage_alert_list` policy.

---

## 10) How to use

1. **Add to chart**: paste the script into a new Pine editor tab and click **Add to chart**.  
2. **Choose lookback**: set `i_array_size` (e.g., 10–50).  
3. **Style**: tweak colors to your liking.  
4. **Watch the logs**: open the **Pine Log** panel to see match blocks.  
5. **(Optional)** enable TradingView alerts by uncommenting the `alert(...)` line.

---

## 11) Troubleshooting tips

- **No boxes appear**: try increasing `i_array_size`, zoom out, and test on higher‑volatility symbols/timeframes. Consider a tolerance match.  
- **Performance concerns**: reduce `i_array_size` and/or gate drawing to `barstate.islast`.  
- **Duplicate / missing logs**: review `manage_alert_list` logic and `_matched` flags; they determine when new blocks are emitted.

---

## 12) Glossary

- **Midpoint**: `(base + target) / 2`.  
- **Container**: a group of rolling arrays representing comparisons to **1‑bar** or **2‑bar** back data.  
- **Match**: an exact equality between a live midpoint and a stored midpoint component (open/high/low/close).  
- **Zone**: a drawn rectangle between the base and alt prices for a match.

---

### Final note
This indicator is **event‑driven** each bar: it detects exact midpoint coincidences and visualizes them as potential reversal zones. For more robust matching, consider adding a **tick‑sized tolerance** and/or normalizing floating points before comparison.
