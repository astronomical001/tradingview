## Liquidity Detection Indicator – Technical Specification

### Vision & Goals
- Provide a robust TradingView indicator for commodities and futures that automatically locates liquidity pools formed by equal highs or equal lows on multiple timeframes.
- Surface actionable liquidity zones with visual emphasis, grading their relative strength, and supplying optional alerts for potential sweep events.
- Ship a codebase that is production-ready for public GitHub release with documentation, configurability, and testability (via built-in validation utilities in Pine Script).

### Core Concepts
- **Liquidity Pool**: A cluster of resting stop orders that forms near swing highs/lows with very similar prices. We detect these as occurrences of equal or nearly equal extremes within a user-defined lookback and tolerance.
- **Equal vs Relative Equal Extremes**:
  - *Equal High/Low*: Two or more swing extremes with absolute difference less than a tick-size derived tolerance.
  - *Relative Equal High/Low*: Extremes within a configurable percentage/ATR/tick threshold that still indicate liquidity.
- **Sweep Signal**: When the most recent bar wicks through an existing liquidity pool and closes back within range (optional alert condition).

### Detection Algorithm
1. **Source Selection**: Work on aggregations of OHLC data for each selected timeframe (default: chart TF, plus 1H, 4H, Daily, Weekly).
2. **Swing Identification**: Use a configurable `left` / `right` bar fractal definition to mark confirmed swing highs and lows.
3. **Clustering Equal Extremes**:
   - Maintain rolling buffers of detected swing highs and lows (price, time, strength) per timeframe.
   - Compare newest swing to previous swings within the lookback window.
   - If absolute difference ≤ tolerance, merge into a liquidity pool; update pool metadata (count, freshness, max deviation).
4. **Tolerance Calculation**:
   - Default: `max(syminfo.mintick * multiplier, atr(tf) * percentage)`, user-configurable weights.
   - Allow manual override (fixed ticks) for markets with non-standard tick sizes.
5. **Scoring**: Rank pools by number of touches, time span, and deviation to derive visual intensity (opacity) and display priority in tables/alerts.
6. **Visualization**:
   - Draw horizontal boxes (`line.new` + `box.new`) extending until invalidation (price trades cleanly through with close beyond tolerance).
   - Labels for each pool showing timeframe and type (EQH/EQL, REQH/REQL).
7. **Alerting**:
   - Trigger when price approaches within tolerance, or when a sweep occurs (bar high/low pierces zone and closes opposite).
   - Payload includes timeframe, pool type, score, and price/percentage deviation.

### Configurable Parameters (Initial Set)
- Timeframe list (string array) for multi-timeframe scanning.
- Swing detection lookback (`swingLeft`, `swingRight`).
- Tolerance strategy: `"ticks"`, `"percent"`, `"atr"`, with multipliers.
- Lookback bars per timeframe for pool maintenance.
- Maximum pools per timeframe to render.
- Alert toggles (approach, sweep) and minimum score threshold.
- Styling: colors for highs vs lows, transparency scaling, label text format, table visibility.

### Data Structures
- `var` arrays for high pools and low pools per timeframe; each pool as a `record` (struct) with fields: `price`, `top`, `bottom`, `strength`, `touches`, `createdAt`, `updatedAt`, `active`.
- Lightweight state machine per pool to manage invalidation and highlight transitions.

### Performance Considerations
- Limit iterations by trimming arrays beyond `maxPools` and skipping heavy math when pools inactive.
- Use `request.security()` judiciously: only on chosen timeframes, reuse results for multiple calculations.
- Avoid drawing thousands of objects by pruning invalid pools and using `max_labels_count`, `max_lines_count` guarding.

### Testing & Validation
- Built-in debug table summarizing pools per timeframe.
- Optional `debug` input to print detection logs to the console (`label`/`plotchar`).
- Backtest harness via TV's `indicator()` with `calc_on_every_tick` toggles for verifying intra-bar behavior.

### Deliverables
- Pine Script v5 indicator (`src/liquidity_pool_detector.pine`).
- README outlining usage, configuration, and screenshots/GIFs (placeholder instructions).
- Example alert message JSON schema.
- Roadmap section for future enhancements (automated trade execution, broker integration, machine-learning tolerance calibration).

### Non-Goals (for v1)
- Automated order execution or brokerage integration.
- Statistical backtesting module (beyond manual TradingView playback).
- Integration with non-TV platforms (NinjaTrader, ThinkorSwim) — can be explored later.
