# Bollinger Band Regression (Pine Script v6)

## Overview
- Scalping strategy combining fast/slow moving averages for trend with Bollinger Bands for entries and ATR for risk sizing.
- Trend must persist for a set number of bars (default 6) before it is considered confirmed.
- Entries occur on closes outside the bands, aligned with the confirmed trend.
- Stops use ATR × coefficient (default 2.0); take-profit is risk × profit ratio (default 2.0).
- Cooldown: after an exit, price must close back inside the bands before another trade is allowed.
- Strategy overlay plots MAs, Bollinger Bands, entry markers, and fixed SL/TP levels; optional debug table/labels included.

## Inputs
- **Moving Average Settings**  
  - MA Type: `SMA` or `EMA` (default `EMA`)  
  - Fast MA Length: `30`  
  - Slow MA Length: `50`  
  - Trend Confirmation Bars: `6`
- **Bollinger Band Settings**  
  - Length: `15`  
  - Std Dev: `1.5`
- **ATR Settings**  
  - ATR Length: `7`
- **Trade Management**  
  - Stop Loss Coefficient: `2.0` (min `0.1`, step `0.1`)  
  - Profit Ratio: `2.0` (min `0.1`, max `20`, step `0.1`)
- **Debug**  
  - Enable Debug: toggle to show diagnostic table and SL/TP label.

## Signals & Logic
- **Trend detection:** Raw trend from fast vs slow MA; counters track consecutive bars. Trend becomes confirmed when raw direction persists for the required bars; otherwise treated as Sideways.
- **Entry rules:**  
  - Uptrend + close below lower BB → Long signal.  
  - Downtrend + close above upper BB → Short signal.  
  - Entry only if not already in a trade and not in the post-exit waiting state.
- **Risk/targets:**  
  - Long SL = entry − (ATR × coefficient); TP = entry + (SL distance × profit ratio).  
  - Short SL = entry + (ATR × coefficient); TP = entry − (SL distance × profit ratio).  
  - SL/TP fixed at entry (not recalculated).
- **Exit rules:**  
  - Long exits if low ≤ SL or high ≥ TP.  
  - Short exits if high ≥ SL or low ≤ TP.
- **Re-entry guard:** After any exit, strategy waits for a close back inside the Bollinger Bands before allowing a new trade.

## Visuals & Debug
- Plots: fast MA (blue), slow MA (orange), BB upper/lower with fill and dotted middle, entry shapes, and SL/TP linebreak plots when in a trade.
- Debug mode: top-right table shows trend, bar counts, ATR, signal, trade state, waiting flag, SL/TP; optional on-chart label with SL/TP at entry.

## Files
- Script implementation: `Strategy/Double-MA-BB/Double-MA-BB.ps`
