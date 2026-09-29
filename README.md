# Pine-Script-3-9min-Based-Indicator

Indicator based on multi-timeframe EMA ribbons, RSI, Ichimoku Kinko Hyo, Hull Suite, and higher-timeframe benchmark levels with a modular "Use" toggle architecture. Signals entry for long/short positions across 3-minute and 9-minute timeframes for any cryptocurrency on TradingView.

--

## Chart Preview

![Indicator Preview](3-9-conversion-ss.png)

--

## Motivation & Problem

- **Multi-Timeframe Misalignment & False Breakouts**: Lower-timeframe trading often falls victim to erratic price noise and false breakout signals when isolated from intermediate trend direction and multi-timeframe momentum confirmation.
- **The Core Goal**: To build a robust, modular indicator that synchronizes technical factors across 1m, 3m, and 9m timeframes—allowing users to selectively toggle EMA ribbon crossovers, multi-timeframe RSI momentum, and Ichimoku slope confirmation while anchoring trades against 1H and 1D price baselines.

--

## Strategy Logic & Architecture

- This indicator avoids false signals and wrong interpretation of the trend by utilizing a **rule-based, multi-factor filtering system**:

### Core Components:

1. **Trend & Ribbon Filters (EMA Ribbons & Hull Suite)**:
  - Deploys a 10-period EMA ribbon (lengths 3, 5, 7, 9, 12, 20, 30, 60, 100, 200) computed across multiple timeframes (3m and 9m) to detect trend transitions and crossover events (e.g., fast EMAs 1–4 crossing EMA 20 or SMA 20).
  - Includes an integrated Hull Suite band (HMA length 55) for smooth trend visualization and higher-timeframe (240m) macro bias confirmation.

2. **Ichimoku Slope & Multi-Timeframe RSI Confirmation**:
  - Monitors the directional slope of the 9-minute Ichimoku Conversion Line (Tenkan-sen) and Leading Span A (Senkou Span A) to ensure trades align with expanding cloud and trend momentum.
  - Pulls multi-timeframe RSI data (default lengths 7, 9, 11 across 3m and 9m) via 'request.security()' to confirm momentum alignment above 55 (bullish) or below 45 (bearish).
  - Implements dynamic boolean switches (UseEMA1to10, UseRSI, UseIchimoku) using ternary bypass logic to easily activate or deactivate individual filters.

3. **Execution Rules by Timeframe**:
  - **3-Minute Chart Signals (IN3)**:
    - **Bullish Signal**: Triggers when 3m EMA crossover up occurs (if enabled) and 3m RSI is above 55 (if enabled).
    - **Bearish Signal**: Triggers when 3m EMA crossover down occurs (if enabled) and 3m RSI is below 45 (if enabled).
  - **9-Minute Chart Signals (IN9)**:
    - **Bullish Signal**: Triggers when 9m EMA crossover up occurs (if enabled) and 9m RSI is above 55 (if enabled) and the 9m Ichimoku Conversion Line is sloping upward and 9m Leading Span A is sloping upward.
    - **Bearish Signal**: Triggers when 9m EMA crossover down occurs (if enabled) and 9m RSI is below 45 (if enabled) and the 9m Ichimoku Conversion Line is sloping downward and 9m Leading Span A is sloping downward.
  - **Exit Marker**: Automatically plots a "SELL" indicator mark on the bar immediately following a signal entry.

4. **Dynamic Higher-Timeframe Benchmarks (1H & 1D Open Levels)**:
  - Fetches the opening prices of the current 1-Hour and 1-Day candles via 'request.security()'.
  - Continuously projects clean, non-lagging horizontal reference lines to serve as key intraday support and resistance pivots.
    
--

## Configurable Parameters

Users can adjust the following parameters inside TradingView's settings panel:
- **Time Frame Inputs**: Default - 1m, 2m, 3m, 9m. Configurable timeframes for multi-timeframe analysis.
- **EMA Ribbon**: Default - Lengths 3, 5, 7, 9, 12, 20 (EMA & SMA), 30, 60, 100, 200. Toggle switches for filtering (UseEMA1to10), global ribbon plotting (PlotEMA1to10), and optional EMA 30 display.
- **RSI Time Frames & Lengths**: Default - 3m and 9m timeframes, lengths 7, 9, 11. Overbought/oversold momentum thresholds (55 / 45) with UseRSI toggle.
- **Ichimoku Settings**: Default - Conversion Line 9, Base Line 26, Leading Span B 52, Displacement 26. Toggles for visual plotting (PlotIchimoku) and signal confirmation (UseIchimoku).
- **Hull Suite**: Default - HMA length 55, 240m HTF. Customizable modes (HMA, EHMA, THMA), band transparency, and line thickness.
- **Time Mark (1H & 1D Anchors)**: Configurable line colors and widths for dynamic 1-Hour and 1-Day open price horizontal levels.
  
--

## How to Install & Use in TradingView

1. Open any crypto chart (e.g., `BTC/USDT`) on **[TradingView](https://www.tradingview.com/)**.
2. Open the **`Pine Editor`** console at the bottom of the page.
3. Open `3-9-conversion.txt` (or your Pine Script file), copy the source code, and paste it into the editor.
4. Click **`Save`** and then click **`Add to Chart`**.
5. Switch chart timeframes to **`3m`** or **`9m`** and click the gear icon (`Settings`) on the indicator to adjust parameters as needed.

--

## Key Learnings & Engineering Reflections

1. **Timeframe-Specific Conditional Strategy Execution (timeframe.period)**
  - I learned that conditioning entry logic on timeframe.period == "3" and timeframe.period == "9" allows a single unified script to deploy differentiated strategies. This enables responsive, fast momentum entries on the 3m chart while enforcing strict, higher-confluence Ichimoku cloud slope validation on the 9m chart.

2. **Clean Benchmark Line Rendering Using line.new and line.delete**
  - I learned how to track higher-timeframe open prices (1H and 1D) and draw single-segment horizontal rays that update dynamically with each bar (line.delete(line1H[1])). This prevents historical chart clutter while giving traders real-time visual reference to macro session anchors.
3. **Modular Filter Control Using the Ternary Operator (x ? y : true)**
  - I learned that using the ternary operator (x ? y : true) allows effortless toggling of individual conditions. If x (Use toggle) is true, the script evaluates y (the filter condition); if x is false, it returns true as a pass-through bypass. This keeps the multi-indicator architecture fully modular without disrupting compound boolean logic.
