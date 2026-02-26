# Routing Rules

Map user keywords and phrases to dashboard IDs. Rules are evaluated top-down; first strong match wins.

## Keyword Routes

| Keywords / Phrases | dashboard_id | reason_code |
|---|---|---|
| value over time, portfolio value, balance history, net worth trend, performance | `portfolio_value_detail` | `keyword:value` |
| fees, expense, commission, cost of investing, fee drag | `fee_analysis` | `keyword:fees` |
| dividend, income, yield, payout, distribution | `dividend_tracker` | `keyword:dividend` |
| allocation, breakdown, diversification, asset class, split by class | `allocation_breakdown` | `keyword:allocation` |
| currency exposure, currency breakdown, split by currency | `allocation_breakdown` | `keyword:currency` |
| top holdings, top symbols, biggest positions, largest investments | `top_holdings` | `keyword:top_holdings` |
| watchlist, watching, tracked symbols, followed | `watchlist_view` | `keyword:watchlist` |
| activity, transactions, buys, sells, trades, orders, recent activity | `activity_summary` | `keyword:activity` |
| overview, summary, snapshot, how is my portfolio, dashboard | `portfolio_overview` | `keyword:overview` |
| health, health check, diversified, risk | `portfolio_health` | `keyword:health` |
| income and fees, dividends and costs, net income | `income_overview` | `keyword:income_overview` |

## Compound Routes

When multiple keywords appear, prefer the more specific dashboard:

| Combination | dashboard_id | reason_code |
|---|---|---|
| "fees" + "dividends" | `income_overview` | `compound:income_and_fees` |
| "allocation" + "value" | `portfolio_overview` | `compound:alloc_plus_value` |
| "top" + "asset class" | `allocation_breakdown` | `compound:top_by_class` |
| "value" + "fees" + "allocation" | `portfolio_health` | `compound:health_check` |

## Time Hint Extraction

| Pattern | time_hint reason_code | date_from logic |
|---|---|---|
| this month, MTD, month to date | `time_hint:mtd` | 1st of current month |
| this year, YTD, year to date | `time_hint:ytd` | Jan 1 of current year |
| last 3 months, past 3 months, 3m, Q1/Q2/Q3/Q4 | `time_hint:3m` | NOW - 3 months |
| last 6 months, past 6 months, 6m, half year | `time_hint:6m` | NOW - 6 months |
| last year, past year, 12 months, 1y, annual | `time_hint:12m` | NOW - 12 months |
| last 2 years, 2y, 24 months | `time_hint:24m` | NOW - 24 months |
| all time, since inception, everything | `time_hint:all` | null (no lower bound) |

## Worked Examples

| # | User phrase | selected_dashboard_id | confidence | reason_codes |
|---|---|---|---|---|
| 1 | "Show me my portfolio value over the last 6 months" | `portfolio_value_detail` | 0.95 | `["keyword:value", "time_hint:6m"]` |
| 2 | "How much have I paid in fees?" | `fee_analysis` | 0.90 | `["keyword:fees"]` |
| 3 | "What dividends did I receive this year?" | `dividend_tracker` | 0.92 | `["keyword:dividend", "time_hint:ytd"]` |
| 4 | "Break down my portfolio by asset class" | `allocation_breakdown` | 0.95 | `["keyword:allocation"]` |
| 5 | "What are my top holdings?" | `top_holdings` | 0.90 | `["keyword:top_holdings"]` |
| 6 | "Show my watchlist" | `watchlist_view` | 0.98 | `["keyword:watchlist"]` |
| 7 | "Recent transactions this month" | `activity_summary` | 0.88 | `["keyword:activity", "time_hint:mtd"]` |
| 8 | "Give me an overview" | `portfolio_overview` | 0.85 | `["keyword:overview"]` |
| 9 | "How much did I earn vs pay in fees?" | `income_overview` | 0.88 | `["compound:income_and_fees"]` |
| 10 | "Is my portfolio well diversified?" | `portfolio_health` | 0.80 | `["keyword:health"]` |
| 11 | "What's my currency exposure?" | `allocation_breakdown` | 0.90 | `["keyword:currency"]` |
| 12 | "Show my Tesla trades" | `activity_summary` | 0.75 | `["keyword:activity", "filter:symbol"]` |
| 13 | "How is my crypto doing?" | `allocation_breakdown` | 0.60 | `["keyword:allocation", "filter:asset_sub_class"]` |
| 14 | "Compare my ETFs to stocks" | `allocation_breakdown` | 0.55 | `["keyword:allocation", "intent:comparison"]` |
| 15 | "What's the weather like?" | null | 0.00 | `["out_of_domain"]` |
