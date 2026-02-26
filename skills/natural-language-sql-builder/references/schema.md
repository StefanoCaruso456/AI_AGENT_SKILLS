# Schema Reference

Ghostfolio Postgres schema — analytics-relevant tables only.

## Primary Analytics Tables

### Order (transactions)

| Column | Type | Notes |
|---|---|---|
| id | text uuid PK | |
| userId | text FK → User.id | **Required scope** |
| date | timestamp(3) UTC | |
| type | enum | BUY, SELL, DIVIDEND, FEE, INTEREST, LIABILITY |
| quantity | double precision | |
| unitPrice | double precision | |
| fee | double precision | |
| currency | text | ISO code, plain text |
| isDraft | boolean | Exclude drafts: `WHERE isDraft = false` |
| symbolProfileId | text FK → SymbolProfile.id | NOT NULL |
| accountId | text, nullable | Part of composite FK to Account |
| accountUserId | text, nullable | Part of composite FK to Account |

### Account (composite PK)

| Column | Type | Notes |
|---|---|---|
| id | text uuid | |
| userId | text uuid | |
| balance | double precision | |
| currency | text | |
| isExcluded | boolean | Filter with `WHERE NOT isExcluded` |

**PK = (id, userId)**

### AccountBalance (time series)

| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| accountId | text | Composite FK to Account |
| userId | text | Composite FK to Account |
| date | timestamp(3) | |
| value | double precision | |

Unique constraint: `(accountId, date)`

### SymbolProfile (asset metadata)

| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| symbol | text | Ticker |
| dataSource | enum | Provider identifier |
| currency | text | Asset's native currency |
| assetClass | enum | EQUITY, FIXED_INCOME, REAL_ESTATE, COMMODITY, LIQUIDITY |
| assetSubClass | enum | STOCK, ETF, CRYPTOCURRENCY, BOND, MUTUAL_FUND, PRECIOUS_METAL |
| sectors | jsonb | Optional array |
| countries | jsonb | Optional array |
| holdings | jsonb | Optional array |
| userId | text, nullable | null = system-managed profile |

Unique constraint: `(dataSource, symbol)`

### MarketData (price history)

| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| dataSource | enum | |
| symbol | text | |
| date | timestamp(3) | |
| marketPrice | double precision | |

Unique constraint: `(dataSource, date, symbol)`

**No FK to SymbolProfile.** Join by convention only.

### Tag

| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| name | text | |
| userId | text, nullable | null = system tag |

Unique constraint: `(name, userId)`

## Join Tables (implicit Prisma many-to-many)

| Table | Column A | Column B |
|---|---|---|
| `_OrderToTag` | Order.id | Tag.id |
| `_UserWatchlist` | SymbolProfile.id | User.id |

## Critical Join Rules

### 1. Composite Account Join

```sql
-- Order → Account
JOIN "Account" a
  ON a.id = o."accountId"
  AND a."userId" = o."accountUserId"

-- AccountBalance → Account
JOIN "Account" a
  ON a.id = ab."accountId"
  AND a."userId" = ab."userId"
```

### 2. MarketData Convention Join

```sql
-- SymbolProfile → MarketData (no FK)
JOIN "MarketData" md
  ON md."dataSource" = sp."dataSource"
  AND md.symbol = sp.symbol
```

### 3. Watchlist Join

```sql
JOIN "_UserWatchlist" uw ON uw."A" = sp.id
WHERE uw."B" = :userId
```

## Gotchas

1. **Money is double precision.** Always `ROUND(x::numeric, 2)` for totals, `ROUND(x::numeric, 4)` for unit prices.
2. **Currency is plain text.** Multi-currency portfolios exist. No FX rate table in schema — if conversion is needed, return `sql: null`.
3. **Timestamps are UTC `TIMESTAMP(3)`.** Use `date_trunc('month', date)` for grouping, `CAST(date AS date)` for daily.
4. **Exclude drafts** by default: `WHERE o."isDraft" = false`.
5. **Exclude hidden accounts** by default: `WHERE NOT a."isExcluded"`.
6. **Prisma column quoting.** Columns use camelCase in Postgres — always double-quote: `"userId"`, `"unitPrice"`, `"symbolProfileId"`.
