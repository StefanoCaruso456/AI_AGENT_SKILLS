# Dashboard Templates

Pre-built dashboard layouts. Each template maps to one or more `dashboard_id` values from the query catalog.

---

## 1. portfolio_overview

| Field | Value |
|---|---|
| template_id | `portfolio_overview` |
| title | Portfolio Overview |
| depends_on | `portfolio_value_timeseries_6m`, `allocation_by_asset_class`, `allocation_by_currency` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `pv_line` | Portfolio Value (6M) | line | `portfolio_value_timeseries_6m` | period | total_value |
| `alloc_class_pie` | Allocation by Asset Class | pie | `allocation_by_asset_class` | asset_class | activity_value |
| `alloc_currency_pie` | Currency Exposure | pie | `allocation_by_currency` | currency | activity_value |

---

## 2. portfolio_value_detail

| Field | Value |
|---|---|
| template_id | `portfolio_value_detail` |
| title | Portfolio Value Over Time |
| depends_on | `portfolio_value_timeseries_6m` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `pv_area` | Portfolio Value | area | `portfolio_value_timeseries_6m` | period | total_value |

---

## 3. activity_summary

| Field | Value |
|---|---|
| template_id | `activity_summary` |
| title | Activity Summary |
| depends_on | `activity_breakdown_mtd` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `activity_bar` | Activity by Type (MTD) | bar | `activity_breakdown_mtd` | order_type | total_amount |
| `activity_table` | Transaction Details | table | `activity_breakdown_mtd` | order_type | tx_count |

---

## 4. fee_analysis

| Field | Value |
|---|---|
| template_id | `fee_analysis` |
| title | Fee Analysis |
| depends_on | `fees_over_time_6m` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `fees_bar` | Fees Over Time (6M) | bar | `fees_over_time_6m` | period | total_fees |

---

## 5. dividend_tracker

| Field | Value |
|---|---|
| template_id | `dividend_tracker` |
| title | Dividend Tracker |
| depends_on | `dividends_over_time_12m` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `div_bar` | Dividends Over Time (12M) | bar | `dividends_over_time_12m` | period | total_dividends |

---

## 6. allocation_breakdown

| Field | Value |
|---|---|
| template_id | `allocation_breakdown` |
| title | Allocation Breakdown |
| depends_on | `allocation_by_asset_class`, `allocation_by_currency` |
| required_filters | none |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `alloc_class` | By Asset Class | pie | `allocation_by_asset_class` | asset_class | activity_value |
| `alloc_currency` | By Currency | pie | `allocation_by_currency` | currency | activity_value |

---

## 7. top_holdings

| Field | Value |
|---|---|
| template_id | `top_holdings` |
| title | Top Holdings |
| depends_on | `top_symbols_by_value_3m` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `top_bar` | Top 20 Symbols by Activity Value (3M) | bar | `top_symbols_by_value_3m` | symbol | activity_value |
| `top_table` | Symbol Details | table | `top_symbols_by_value_3m` | symbol | activity_value |

---

## 8. watchlist_view

| Field | Value |
|---|---|
| template_id | `watchlist_view` |
| title | Watchlist |
| depends_on | `watchlist_symbols` |
| required_filters | none |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `watchlist_table` | Watched Symbols | table | `watchlist_symbols` | symbol | asset_class |

---

## 9. income_overview

| Field | Value |
|---|---|
| template_id | `income_overview` |
| title | Income Overview |
| depends_on | `dividends_over_time_12m`, `fees_over_time_6m` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `income_bar` | Dividend Income (12M) | bar | `dividends_over_time_12m` | period | total_dividends |
| `fees_bar` | Fees Paid (6M) | bar | `fees_over_time_6m` | period | total_fees |

---

## 10. portfolio_health

| Field | Value |
|---|---|
| template_id | `portfolio_health` |
| title | Portfolio Health Check |
| depends_on | `portfolio_value_timeseries_6m`, `fees_over_time_6m`, `allocation_by_asset_class` |
| required_filters | `date_from` |

**Tiles:**

| tile_id | title | chart_type | sql_ref | x_field | y_field |
|---|---|---|---|---|---|
| `pv_line` | Value Trend (6M) | line | `portfolio_value_timeseries_6m` | period | total_value |
| `fees_bar` | Fee Drag (6M) | bar | `fees_over_time_6m` | period | total_fees |
| `alloc_pie` | Asset Class Mix | pie | `allocation_by_asset_class` | asset_class | activity_value |
