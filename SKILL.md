---
name: trade-replay-viewer
description: Use this skill whenever the user wants an interactive trading replay, order annotation viewer, strategy visualization, trade chart, buy/sell point review, or equity-curve-linked chart UI. It guides building a lightweight-charts based viewer with symbol/date navigation, keyboard shortcuts, URL parameters, candlestick main/sub panes, clickable equity curve navigation, and human-readable order rationale. Do not bind the implementation to any particular CSV schema, database, broker export, or project-specific trade format.
---

# Trade Replay Viewer

Build interactive visualizations for reviewing trading strategies and orders. Preserve the interaction model and review workflow, but adapt data ingestion and field mapping to the project at hand.

## Purpose

The viewer is for human strategy review. A trader or researcher should be able to answer these questions quickly:

- What happened on this symbol and date?
- Where were the signal, entry, exit, stop, target, or adjustment points?
- Why did the strategy act there?
- How did the trade perform after entry?
- Where does this day sit on the broader equity curve?

Avoid showing raw records first. Raw data can be available behind a disclosure, but the primary surface should explain the decision in human terms.

## Core Interaction Model

Implement these interactions whenever feasible:

1. Symbol navigation
   - Provide a visible symbol selector.
   - Support keyboard up/down to switch symbols.
   - Keep `symbol` in the URL query string.
   - If a requested symbol is unavailable for the selected date, choose the nearest sensible available symbol/date pair rather than failing silently.

2. Date navigation
   - Provide a visible date/session selector.
   - Support keyboard left/right to move to previous/next date.
   - Keep `date`, `day`, or another clearly named date parameter in the URL query string.
   - Default to the latest available date unless the URL specifies a valid date.

3. URL state
   - Treat the URL as shareable state.
   - On every symbol/date change, update the URL with `history.replaceState` or the framework equivalent.
   - On load, read the URL first, then fall back to latest date and first available symbol.

4. Equity curve navigation
   - Show an independent equity curve below or near the replay chart.
   - Make equity curve points clickable.
   - Clicking a point jumps the replay chart to the corresponding date/session.
   - If the selected symbol has no trade that day, switch to a symbol that has data for that day or show the market/session context for the current symbol if available.

5. Trade highlight behavior
   - Clicking an order or trade card should highlight the corresponding entry and exit on the main chart.
   - Use a clear visual treatment such as an entry-to-exit line, entry/exit price lines, active marker styling, or a highlighted holding interval.
   - Do not zoom so tightly that a short trade fills the whole viewport. That creates a distorted, stretched review experience.
   - Preserve a minimum context window around the trade, such as 60 to 120 bars, centered on the holding interval.
   - For long holds, show the full holding interval plus buffer bars before entry and after exit.
   - Prefer moving the viewport to the trade while preserving usable context over aggressive local zoom.

## Chart Structure

Use `lightweight-charts` unless the project has a strong existing charting standard.

Prefer a multi-pane chart:

- Main pane: candlesticks or bars.
- Main overlays: strategy-relevant technical indicators, levels, bands, VWAP, moving averages, opening range, stops, targets, or model states.
- Sub panes: strategy-specific diagnostics such as MACD histogram, volume, volatility, prediction score, position size, drawdown, or signal confidence.
- Order markers: signal, entry, exit, stop, target, add, reduce, cancel, or rebalance markers.

Choose indicators from the user's strategy design. Do not add generic indicators just because they are common. The point is to explain the strategy, not decorate the chart.

## Order Annotation

Every order or decision marker should have a human-readable annotation.

The annotation should answer:

- Why did the strategy enter, exit, hold, reduce, or stop?
- Which strategy rule or model condition was active?
- Which market context mattered at that moment?
- What risk/target/path information mattered after the decision?

Good annotation examples:

- `Entry: long pullback. Opening range bias was bullish, pullback energy was close to prior impulse energy, price held above the structure level.`
- `Exit: structure stop. Price broke the support level that justified the long entry.`
- `Exit: time stop. Position did not move away from cost after the expected holding window.`
- `Hold: model exit probability stayed below threshold, drawdown remained inside the allowed path.`

Avoid raw-field dumps as the primary UI:

- Bad: `exit_reason=end_of_day_no_stop, area_ratio=1.24, pred=0.63`
- Better: `Exit: held until session end because no stop condition fired. Model exit probability stayed below the active threshold.`

Keep raw data available in a collapsed `details`, inspector, tooltip, or debug panel for auditability.

## Data Contract Guidance

Do not assume a fixed schema. Instead, map project data into a viewer-friendly contract.

Recommended normalized concepts:

- `symbols`: available instruments or strategy groups.
- `sessions`: available dates or trading sessions per symbol.
- `bars`: time, open, high, low, close, optional volume.
- `overlays`: named line/area series for main-pane indicators and levels.
- `panes`: named secondary series for diagnostics.
- `orders`: time, action, side, price, marker style, human annotation, optional raw record.
- `equity`: date/session, equity value, optional return, drawdown, trade count.

Adapt this contract to the framework and storage layer. The skill's intent is the interaction and explanatory style, not a rigid file format.

## Implementation Workflow

1. Discover the data sources
   - Locate market bars, order/trade records, and equity or daily return records.
   - Identify available symbols and dates.
   - Identify strategy-specific indicators or decision variables that explain entries/exits.

2. Normalize for the viewer
   - Convert timestamps into the format expected by `lightweight-charts`.
   - Group data by symbol and date/session.
   - Compute only the indicators needed to explain this strategy.
   - Build concise order annotations from strategy rules or model outputs.

3. Build the replay chart
   - Render candlesticks in the main pane.
   - Add main overlays and sub panes.
   - Add order markers and a detail panel.
   - Keep raw records secondary.

4. Add navigation
   - Add symbol/date selectors.
   - Add keyboard shortcuts: left/right for dates, up/down for symbols.
   - Add URL parameter read/write.
   - Default to the latest date.

5. Add equity navigation
   - Render a separate equity chart.
   - Click equity points to jump to the matching date/session.
   - Preserve symbol/date URL state after the jump.

6. Add trade highlighting
   - Make order cards or marker details clickable.
   - Highlight the selected order's entry and exit on the main chart.
   - Keep the viewport wide enough to show surrounding market context.

7. Verify behavior
   - Load with no URL parameters: latest date appears.
   - Load with valid URL parameters: requested symbol/date appears.
   - Load with invalid URL parameters: viewer falls back predictably.
   - Keyboard shortcuts update chart and URL.
   - Equity click jumps to the expected date.
   - Clicking a trade highlights the correct entry/exit without over-zooming.
   - Order cards explain why the strategy acted, not merely what field values were present.

## UI Style

Use a dense but readable research-tool style:

- Keep selectors visible and compact.
- Use dark or light theme according to the host product; do not force a theme if a design system exists.
- Place chart first, explanations beside or below it.
- Use color consistently for long/short, profit/loss, and active/disabled states.
- Keep keyboard hints visible.
- Use collapsed raw details so audit data is available without overwhelming the main review flow.

## Complexity Guidance

Keep the viewer straightforward:

- Prefer a small normalization step plus a simple static or client-rendered viewer.
- Do not build a backend unless the dataset size or security model requires it.
- Do not add compatibility branches for unknown schemas. Ask for the mapping or inspect the project data.
- Keep fallback behavior explicit and easy to verify.

## Done Criteria

The task is complete when a user can:

- Open the viewer and see the latest available date by default.
- Share a URL that opens the same symbol/date.
- Use arrow keys to move across symbols and dates.
- See candlesticks with strategy-relevant indicators in main/sub panes.
- Click the equity curve to jump into a day replay.
- Click an order or trade card to highlight its entry and exit while keeping enough chart context.
- Read human-oriented order annotations explaining the strategy's reasons.
- Expand raw records only when they need audit detail.
