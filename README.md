# Trade Replay Viewer Skill

An agent skill for building interactive trading replay viewers.

It captures a reusable interaction pattern for strategy review:

- Symbol and date navigation with keyboard shortcuts and URL parameters.
- `lightweight-charts` candlestick replay with strategy-specific overlays and sub panes.
- Clickable equity curve navigation back into individual trading days.
- Human-readable order annotations that explain why a strategy acted.

The skill intentionally does not assume a specific CSV schema, database, broker export, or trade record format. It describes the interaction model and normalized concepts an agent should map project data into.

## Files

- `SKILL.md`: the skill definition.

## Use

Install or copy this repository as an agent skill named `trade-replay-viewer`.
