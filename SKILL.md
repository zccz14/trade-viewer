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
   - If the selected date/session has trades, default to highlighting the first trade when no trade is specified in the URL.
   - Add a URL parameter for the selected trade, such as `trade=1`, using the same human-facing numbering shown in the UI.
   - Loading a URL with a valid trade parameter should restore that trade highlight.
   - If the trade parameter is missing, invalid, or out of range, fall back predictably to the first available trade.
   - Use a clear visual treatment such as an entry-to-exit line, entry/exit price lines, active marker styling, or a highlighted holding interval.
   - Clicking a trade should not change the current zoom scale by default.
   - If the trade entry and exit are already inside the current viewport, only apply the highlight.
   - If the trade is outside the current viewport, pan or scroll the viewport just enough to bring the entry/exit segment into view while preserving the current viewport width.
   - Avoid aggressive local zoom. A short trade should not fill the whole chart because that creates a distorted, stretched review experience.

## Chart Structure

Use `lightweight-charts` v5 unless the project has a strong existing charting standard.

Prefer one replay chart with v5 panes for the trading day view:

- Main pane: candlesticks or bars.
- Main overlays: strategy-relevant technical indicators, levels, bands, VWAP, moving averages, opening range, stops, targets, or model states.
- Sub panes: strategy-specific diagnostics such as MACD histogram, volume, volatility, prediction score, position size, drawdown, or signal confidence.
- Order markers: signal, entry, exit, stop, target, add, reduce, cancel, or rebalance markers.
- Legend: a compact current-bar readout for OHLC, active overlays, pane values, and selected order context.

Use the v5 pane API to keep the main chart and sub panes on one shared time scale, crosshair, and zoom context. Do not implement main/sub panes as separate chart instances unless the host chart library forces that design.

Keep the equity curve as an independent chart instance. It should be clickable for date navigation, but it should not share the replay chart's time scale, crosshair, scroll, or zoom state. The equity curve summarizes strategy history; the replay panes inspect one trading day.

Add a legend to the replay chart. The legend should update on crosshair movement and show the values a human needs for the bar under inspection: time, OHLC, key overlays, important sub-pane values, and any selected order or marker context. Keep it compact and readable; do not force users to inspect raw tooltips or raw JSON to understand the current bar.

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
   - Use `lightweight-charts` v5.
   - Render candlesticks in the main pane.
   - Add sub panes with the v5 pane API so the day replay shares one time scale and crosshair.
   - Add order markers and a detail panel.
   - Add a crosshair-driven legend for OHLC, overlays, pane values, and selected order context.
   - Keep raw records secondary.

4. Add navigation
   - Add symbol/date selectors.
   - Add keyboard shortcuts: left/right for dates, up/down for symbols.
   - Add URL parameter read/write for symbol, date/session, and selected trade.
   - Default to the latest date.

5. Add equity navigation
   - Render a separate equity chart instance.
   - Click equity points to jump to the matching date/session.
   - Preserve symbol/date URL state after the jump.
   - Do not synchronize equity chart zoom, scroll, or crosshair with the replay chart.

6. Add trade highlighting
   - Make order cards or marker details clickable.
   - Highlight the selected order's entry and exit on the main chart.
   - Default to the first trade for a selected date/session.
   - Persist selected trade state in the URL and restore it on load.
   - Preserve the current zoom scale when a trade is clicked; pan only if needed to make entry and exit visible.

7. Verify behavior
   - Load with no URL parameters: latest date appears.
   - Load with valid URL parameters: requested symbol/date appears.
   - Load with invalid URL parameters: viewer falls back predictably.
   - Keyboard shortcuts update chart and URL.
   - Equity click jumps to the expected date.
   - Clicking a trade highlights the correct entry/exit without changing zoom scale unless the host charting library leaves no alternative.
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
- Share a URL that opens the same symbol/date and selected trade.
- Use arrow keys to move across symbols and dates.
- See candlesticks with strategy-relevant indicators in main/sub panes.
- See replay main/sub panes managed by `lightweight-charts` v5 panes with shared replay time scale and crosshair.
- Use a legend to read the current bar's OHLC, indicator values, pane values, and selected order context.
- Click the equity curve to jump into a day replay.
- Use the equity curve independently from the replay chart's pane synchronization.
- Click an order or trade card to highlight its entry and exit while preserving the current zoom scale and panning only when needed.
- Open a date with trades and see the first trade highlighted by default when no trade URL parameter is present.
- Read human-oriented order annotations explaining the strategy's reasons.
- Expand raw records only when they need audit detail.
