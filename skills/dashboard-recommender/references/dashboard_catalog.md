# Dashboard Catalog

Available dashboard templates. IDs match `financial-dashboard-builder/references/dashboard_templates.md` and `natural-language-sql-builder/references/query_catalog.md`.

| dashboard_id | title | What it answers |
|---|---|---|
| `portfolio_overview` | Portfolio Overview | High-level snapshot: value trend + allocation by class and currency |
| `portfolio_value_detail` | Portfolio Value Over Time | How has my total portfolio value changed over the past N months? |
| `activity_summary` | Activity Summary | What transactions have I made this period, grouped by type? |
| `fee_analysis` | Fee Analysis | How much am I paying in fees, and how has it trended? |
| `dividend_tracker` | Dividend Tracker | How much dividend income have I received over the past 12 months? |
| `allocation_breakdown` | Allocation Breakdown | What is my portfolio split by asset class and currency? |
| `top_holdings` | Top Holdings | Which symbols have the highest activity value in the last 3 months? |
| `watchlist_view` | Watchlist | What symbols am I watching? |
| `income_overview` | Income Overview | Combined view of dividend income and fees paid |
| `portfolio_health` | Portfolio Health Check | Value trend + fee drag + asset class diversification |

## Choosing Between Similar Dashboards

| User intent | Preferred | Reason |
|---|---|---|
| General "how's my portfolio" | `portfolio_overview` | Broadest coverage |
| Specifically about value trend | `portfolio_value_detail` | Single focused chart |
| "Allocation" without qualifier | `allocation_breakdown` | Dedicated dual-pie layout |
| "Allocation" as part of overview | `portfolio_overview` | Includes allocation tile |
| "Fees and dividends together" | `income_overview` | Side-by-side comparison |
| "Is my portfolio healthy" | `portfolio_health` | Multi-dimension check |
