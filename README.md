# Liquidity Pool Detector – TradingView Indicator

## Overview
The Liquidity Pool Detector is a Pine Script v5 indicator engineered for commodities and futures markets. It identifies and highlights liquidity pools that form when prices create equal or relatively equal highs and lows across multiple timeframes. The script overlays smart liquidity zones, tracks touch counts, and surfaces sweep/approach alerts to support discretionary or systematic execution models.

Key capabilities include:
- Multi-timeframe swing analysis (chart TF + configurable higher TFs)
- Dynamic tolerance modelling via ticks, percentage, ATR, or hybrid rules
- Auto-generated zones with live updating boxes/labels and lifecycle management
- Sweep and approach alerts with contextual messaging
- In-chart summary table of active liquidity pools

## Repository Layout
- `src/liquidity_pool_detector.pine` – primary indicator implementation
- `docs/liquidity-detection-spec.md` – architecture and detection logic specification

## Installation
1. Open TradingView and create a new Pine Script indicator (version 5).
2. Copy the contents of `src/liquidity_pool_detector.pine` into the Pine Editor.
3. Click **Add to chart**.
4. Adjust inputs (timeframes, tolerances, alerts) to match your trading workflow.

## Configuration Highlights
- **Timeframes:** Toggle inclusion of the current chart timeframe plus up to four higher aggregation periods (defaults: 1H, 4H, Daily, Weekly).
- **Swing Detection:** Configure `Swing Left`/`Swing Right` to tune how far back/forward pivots are confirmed. Higher values reduce noise but increase latency.
- **Tolerance Model:** Choose between tick-based, percent-based, ATR-based, or hybrid tolerance bands. Hybrid takes the maximum of percent and ATR while respecting the tick floor.
- **Zone Lifecycle:** Limit active pools per timeframe/side and prune zones once they age beyond the `Lookback Bars` window or invalidate via decisive closes.
- **Alerts:** Opt into approach (price enters tolerance band) and sweep (wick through and close back) alerts. Messages include timeframe and liquidity side.

## Using the Indicator
1. Apply the indicator to your commodity or futures chart (supports continuous futures such as `CME:CL1!`).
2. Review the summary table (top-right) for active equal-high (EQH) or equal-low (EQL) pools, touch counts, and age.
3. Monitor shaded zones on-chart; opacity conveys relative strength. Labels display timeframe, touches, and tolerance band.
4. Enable alerts to receive notifications when price approaches or sweeps a zone. Combine with discretionary confirmation (order flow, market structure) before execution.

## Development Notes
- The script uses persistent arrays to track pool metadata and TradingView drawing objects (`box`, `label`).
- Zones self-manage their lifecycle: they expand with new touches, extend forward, and deactivate on violation or staleness.
- The hybrid tolerance model defaults to the strictest tick floor, preventing underestimation on thinly traded contracts.
- Alert throttling ensures a single alert per bar per zone, reducing notification spam.

## Roadmap Ideas
- Expand timeframe selection via CSV parsing for more than four custom frames.
- Add statistics panel summarizing historic sweep outcomes.
- Integrate volume/position data (e.g., Commitment of Traders) for context weighting.
- Package companion Python notebooks for batch signal analysis using exported TradingView data.

---
For questions or contributions, please open an issue or submit a pull request in the repository hosting this code.
