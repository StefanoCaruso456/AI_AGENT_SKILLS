# Query Catalog

Canonical query templates. Prefer these over freeform SQL. Each template is a proven-safe pattern that respects all safety rules.

Each template includes a `tool_equivalent` — the Ghostfolio AI service tool that provides equivalent data via the API layer. Use the SQL template when direct database access is needed; use the tool equivalent when going through the service layer.

**Canonical Ghostfolio tool names:** `getPortfolioSummary`, `listActivities`, `getAllocations`, `getPerformance`, `getQuote`, `getHistory`, `getFundamentals`, `getNews`, `computeRebalance`, `scenarioImpact`

---

## 1. portfolio_value_timeseries_6m

| Field | Value |
|---|---|
| dashboard_id | `portfolio_value_timeseries_6m` |
| tool_equivalent | `getPerformance` |
| intent_type | `time_series` |
| chart_type | `line` |
| Expected filters | `:userId`, `:date_from` (default 6mo ago) |
| Tables | AccountBalance, Account |
| x_field | `period` |
| y_field | `total_value` |

**SQL outline:**

```sql
SELECT date_trunc('month', ab.date) AS period,
       ROUND(SUM(ab.value)::numeric, 2) AS total_value
FROM "AccountBalance" ab
JOIN "Account" a ON a.id = ab."accountId" AND a."userId" = ab."userId"
WHERE ab."userId" = :userId
  AND NOT a."isExcluded"
  AND ab.date >= :date_from
GROUP BY period
ORDER BY period
```

**Notes:** Sums across all non-excluded accounts. Value is a point-in-time snapshot per AccountBalance row — it reflects the balance recorded at that date, not a live market value. The `getPerformance` tool provides a richer view with net performance %, returns, and net worth via the portfolio service.

---

## 2. activity_breakdown_mtd

| Field | Value |
|---|---|
| dashboard_id | `activity_breakdown_mtd` |
| tool_equivalent | `listActivities` |
| intent_type | `aggregation` |
| chart_type | `bar` |
| Expected filters | `:userId`, `:date_from` (1st of current month) |
| Tables | Order |
| x_field | `order_type` |
| y_field | `total_amount` |

**SQL outline:**

```sql
SELECT o.type AS order_type,
       COUNT(*) AS tx_count,
       ROUND(SUM(o.quantity * o."unitPrice")::numeric, 2) AS total_amount
FROM "Order" o
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.date >= :date_from
GROUP BY o.type
ORDER BY total_amount DESC
```

**Notes:** `quantity * unitPrice` is the activity value at time of transaction, not the current holding value. Each Order row is in its own `currency` (nullable) — this is a single-currency-per-row sum. Cross-currency totals require FX conversion. The `listActivities` tool returns the same data with date range and type filtering via the order service.

---

## 3. fees_over_time_6m

| Field | Value |
|---|---|
| dashboard_id | `fees_over_time_6m` |
| tool_equivalent | `listActivities` (filtered: type=FEE) |
| intent_type | `time_series` |
| chart_type | `bar` |
| Expected filters | `:userId`, `:date_from` (6mo ago) |
| Tables | Order |
| x_field | `period` |
| y_field | `total_fees` |

**SQL outline:**

```sql
SELECT date_trunc('month', o.date) AS period,
       ROUND(SUM(o.fee)::numeric, 2) AS total_fees
FROM "Order" o
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.type = 'FEE'
  AND o.date >= :date_from
GROUP BY period
ORDER BY period
```

**Notes:** Filters to `type = 'FEE'` rows only. The `fee` column on other order types (BUY/SELL) represents transaction fees attached to that trade — include those only if the user asks for "all fees including trading fees". Default is FEE-type orders only.

---

## 4. dividends_over_time_12m

| Field | Value |
|---|---|
| dashboard_id | `dividends_over_time_12m` |
| tool_equivalent | `listActivities` (filtered: type=DIVIDEND) |
| intent_type | `time_series` |
| chart_type | `bar` |
| Expected filters | `:userId`, `:date_from` (12mo ago) |
| Tables | Order |
| x_field | `period` |
| y_field | `total_dividends` |

**SQL outline:**

```sql
SELECT date_trunc('month', o.date) AS period,
       ROUND(SUM(o.quantity * o."unitPrice")::numeric, 2) AS total_dividends
FROM "Order" o
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.type = 'DIVIDEND'
  AND o.date >= :date_from
GROUP BY period
ORDER BY period
```

**Notes:** Dividend value = `quantity * unitPrice` per Ghostfolio convention. The `fee` column on dividend rows represents withholding tax if populated.

---

## 5. allocation_by_asset_class

| Field | Value |
|---|---|
| dashboard_id | `allocation_by_asset_class` |
| tool_equivalent | `getAllocations` |
| intent_type | `aggregation` |
| chart_type | `pie` |
| Expected filters | `:userId` |
| Tables | Order, SymbolProfile, SymbolProfileOverrides |
| x_field | `asset_class` |
| y_field | `activity_value` |

**SQL outline:**

```sql
SELECT COALESCE(spo."assetClass", sp."assetClass", 'UNKNOWN') AS asset_class,
       ROUND(SUM(o.quantity * o."unitPrice")::numeric, 2) AS activity_value
FROM "Order" o
JOIN "SymbolProfile" sp ON sp.id = o."symbolProfileId"
LEFT JOIN "SymbolProfileOverrides" spo ON spo."symbolProfileId" = sp.id
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.type IN ('BUY', 'SELL')
GROUP BY asset_class
ORDER BY activity_value DESC
```

**Notes:** Uses COALESCE to apply SymbolProfileOverrides. Shows allocation by historical activity value (sum of buy/sell amounts), **not** current market value. The `getAllocations` tool provides current allocation using live portfolio data with proper market valuations.

---

## 6. allocation_by_currency

| Field | Value |
|---|---|
| dashboard_id | `allocation_by_currency` |
| tool_equivalent | `getAllocations` |
| intent_type | `aggregation` |
| chart_type | `pie` |
| Expected filters | `:userId` |
| Tables | Order, SymbolProfile |
| x_field | `currency` |
| y_field | `activity_value` |

**SQL outline:**

```sql
SELECT COALESCE(o.currency, sp.currency) AS currency,
       ROUND(SUM(o.quantity * o."unitPrice")::numeric, 2) AS activity_value
FROM "Order" o
JOIN "SymbolProfile" sp ON sp.id = o."symbolProfileId"
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.type IN ('BUY', 'SELL')
GROUP BY currency
ORDER BY activity_value DESC
```

**Notes:** Falls back to SymbolProfile.currency when Order.currency is null. Groups by the order's native currency without FX conversion. Results show exposure per currency at transaction time. The `getAllocations` tool provides current currency exposure from the portfolio service.

---

## 7. top_symbols_by_value_3m

| Field | Value |
|---|---|
| dashboard_id | `top_symbols_by_value_3m` |
| tool_equivalent | `getPortfolioSummary` |
| intent_type | `aggregation` |
| chart_type | `bar` |
| Expected filters | `:userId`, `:date_from` (3mo ago) |
| Tables | Order, SymbolProfile |
| x_field | `symbol` |
| y_field | `activity_value` |

**SQL outline:**

```sql
SELECT sp.symbol,
       sp.name,
       ROUND(SUM(o.quantity * o."unitPrice")::numeric, 2) AS activity_value
FROM "Order" o
JOIN "SymbolProfile" sp ON sp.id = o."symbolProfileId"
WHERE o."userId" = :userId
  AND o."isDraft" = false
  AND o.type IN ('BUY', 'SELL')
  AND o.date >= :date_from
GROUP BY sp.symbol, sp.name
ORDER BY activity_value DESC
LIMIT 20
```

**Notes:** Ranked by activity value, not current holdings. LIMIT 20 for readability. The `getPortfolioSummary` tool returns top holdings by current allocation percentage (market-value weighted), which is more accurate for "what do I own now".

---

## 8. watchlist_symbols

| Field | Value |
|---|---|
| dashboard_id | `watchlist_symbols` |
| tool_equivalent | none (no direct tool) |
| intent_type | `detail` |
| chart_type | `table` |
| Expected filters | `:userId` |
| Tables | SymbolProfile, _UserWatchlist |
| x_field | `symbol` |
| y_field | `asset_class` |

**SQL outline:**

```sql
SELECT sp.symbol,
       sp.name,
       COALESCE(spo."assetClass", sp."assetClass") AS asset_class,
       COALESCE(spo."assetSubClass", sp."assetSubClass") AS asset_sub_class,
       sp.currency
FROM "SymbolProfile" sp
JOIN "_UserWatchlist" uw ON uw."A" = sp.id
LEFT JOIN "SymbolProfileOverrides" spo ON spo."symbolProfileId" = sp.id
WHERE uw."B" = :userId
ORDER BY sp.symbol
LIMIT 200
```

**Notes:** The `_UserWatchlist` join table uses `A` = SymbolProfile.id and `B` = User.id (Prisma implicit many-to-many convention). Does not include current prices — that would require a `getQuote` tool call or MarketData join filtered to the latest date per symbol. No direct tool equivalent exists; use `getQuote` for live prices on watchlist symbols.

---

## Tool Coverage Matrix

Shows which Ghostfolio tools map to which SQL templates, and which tools have no SQL equivalent (they require external APIs).

| Tool | SQL Template(s) | Data Source |
|---|---|---|
| `getPortfolioSummary` | `top_symbols_by_value_3m` (partial) | Portfolio service (computed) |
| `listActivities` | `activity_breakdown_mtd`, `fees_over_time_6m`, `dividends_over_time_12m` | Order table |
| `getAllocations` | `allocation_by_asset_class`, `allocation_by_currency` | Portfolio service (computed) |
| `getPerformance` | `portfolio_value_timeseries_6m` (partial) | Portfolio service (computed) |
| `getQuote` | none | External API (yahoo-finance2) |
| `getHistory` | none | External API (yahoo-finance2) |
| `getFundamentals` | none | External API (yahoo-finance2) |
| `getNews` | none | External API (yahoo-finance2) |
| `computeRebalance` | none | Portfolio service (computed) |
| `scenarioImpact` | none | Portfolio service (computed) |

**Key difference:** SQL templates query raw transaction/balance data. Service-layer tools (`getPortfolioSummary`, `getAllocations`, `getPerformance`) compute current market-weighted values using live prices. SQL-based allocation is "activity-based" (historical); tool-based allocation is "market-value-based" (current).
