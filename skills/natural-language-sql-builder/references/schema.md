# Schema Reference

Ghostfolio Postgres schema — verified against `prisma/schema.prisma`. Analytics-relevant tables and all enums.

## Primary Analytics Tables

### Order (transactions)

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| userId | text FK → User.id | **Required scope** |
| date | timestamp(3) | UTC, NOT NULL |
| type | enum Type | BUY, SELL, DIVIDEND, FEE, INTEREST, LIABILITY |
| quantity | Float | double precision |
| unitPrice | Float | double precision |
| fee | Float | double precision |
| currency | text, nullable | ISO code, plain text |
| isDraft | boolean | Default false. Exclude drafts: `WHERE "isDraft" = false` |
| symbolProfileId | text FK → SymbolProfile.id | NOT NULL |
| accountId | text, nullable | Part of composite FK to Account |
| accountUserId | text, nullable | Part of composite FK to Account |
| comment | text, nullable | User note on the order |

Indexes: `accountId`, `date`, `isDraft`, `userId`

### Account (composite PK)

| Column | Type | Notes |
|---|---|---|
| id | text | `@default(uuid())` |
| userId | text FK → User.id | |
| name | text, nullable | Account display name |
| balance | Float | Default 0 |
| currency | text, nullable | |
| isExcluded | boolean | Default false. Filter: `WHERE NOT "isExcluded"` |
| comment | text, nullable | |
| platformId | text FK → Platform.id, nullable | Brokerage platform |

**PK = (id, userId)** — composite primary key.

Indexes: `currency`, `id`, `isExcluded`, `name`, `userId`

### AccountBalance (time series)

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| accountId | text | Composite FK to Account (id, userId) |
| userId | text | Composite FK to Account (id, userId) |
| date | timestamp(3) | Default `now()` |
| value | Float | double precision |

Unique constraint: `(accountId, date)`
Indexes: `accountId`, `date`

### SymbolProfile (asset metadata)

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| symbol | text | Ticker |
| name | text, nullable | Display name |
| dataSource | enum DataSource | Provider identifier |
| currency | text | Asset's native currency, NOT NULL |
| assetClass | enum AssetClass, nullable | |
| assetSubClass | enum AssetSubClass, nullable | |
| isActive | boolean | Default true |
| sectors | jsonb, nullable | Array of sector objects |
| countries | jsonb, nullable | Array of country objects |
| holdings | jsonb | Default `[]` |
| isin | text, nullable | International Securities ID |
| cusip | text, nullable | |
| figi | text, nullable | Bloomberg FIGI |
| figiComposite | text, nullable | |
| figiShareClass | text, nullable | |
| url | text, nullable | |
| symbolMapping | jsonb, nullable | |
| scraperConfiguration | jsonb, nullable | |
| comment | text, nullable | |
| userId | text FK → User.id, nullable | null = system-managed profile |

Unique constraint: `(dataSource, symbol)`
Indexes: `assetClass`, `currency`, `cusip`, `dataSource`, `isActive`, `isin`, `name`, `symbol`

### SymbolProfileOverrides

| Column | Type | Notes |
|---|---|---|
| symbolProfileId | text PK, FK → SymbolProfile.id | 1:1 override |
| assetClass | enum AssetClass, nullable | Override |
| assetSubClass | enum AssetSubClass, nullable | Override |
| name | text, nullable | Override |
| sectors | jsonb | Default `[]` |
| countries | jsonb | Default `[]` |
| holdings | jsonb | Default `[]` |
| url | text, nullable | Override |

When querying asset metadata, check SymbolProfileOverrides first:

```sql
COALESCE(spo."assetClass", sp."assetClass") AS asset_class
```

### MarketData (price history)

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| dataSource | enum DataSource | |
| symbol | text | |
| date | timestamp(3) | |
| marketPrice | Float | double precision |
| state | enum MarketDataState | Default CLOSE |

Unique constraint: `(dataSource, date, symbol)`
Indexes: `dataSource`, `(dataSource, symbol)`, `date`, `marketPrice`, `state`, `symbol`

**No FK to SymbolProfile.** Join by convention only.

### Tag

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| name | text | |
| userId | text FK → User.id, nullable | null = system tag |

Unique constraint: `(name, userId)`
Index: `name`

### Platform

| Column | Type | Notes |
|---|---|---|
| id | text PK | `@default(uuid())` |
| name | text, nullable | |
| url | text, unique | |

Referenced by Account.platformId.

## Join Tables (implicit Prisma many-to-many)

| Table | Column A | Column B |
|---|---|---|
| `_OrderToTag` | Order.id | Tag.id |
| `_UserWatchlist` | SymbolProfile.id | User.id |

## Enums

### Type (Order.type)

```
BUY | SELL | DIVIDEND | FEE | INTEREST | LIABILITY
```

### AssetClass

```
ALTERNATIVE_INVESTMENT | COMMODITY | EQUITY | FIXED_INCOME | LIQUIDITY | REAL_ESTATE
```

### AssetSubClass

```
BOND | CASH | COLLECTIBLE | COMMODITY | CRYPTOCURRENCY | ETF | MUTUALFUND | PRECIOUS_METAL | PRIVATE_EQUITY | STOCK
```

Note: `MUTUALFUND` — no underscore.

### DataSource

```
ALPHA_VANTAGE | COINGECKO | EOD_HISTORICAL_DATA | FINANCIAL_MODELING_PREP | GHOSTFOLIO | GOOGLE_SHEETS | MANUAL | RAPID_API | YAHOO
```

### MarketDataState

```
CLOSE | INTRADAY
```

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
-- SymbolProfile → MarketData (no FK exists)
JOIN "MarketData" md
  ON md."dataSource" = sp."dataSource"
  AND md.symbol = sp.symbol
```

### 3. Watchlist Join

```sql
JOIN "_UserWatchlist" uw ON uw."A" = sp.id
WHERE uw."B" = :userId
```

### 4. SymbolProfileOverrides Join

```sql
LEFT JOIN "SymbolProfileOverrides" spo
  ON spo."symbolProfileId" = sp.id
```

Use `COALESCE(spo.column, sp.column)` to apply overrides.

## Gotchas

1. **Money is double precision (Float).** Always `ROUND(x::numeric, 2)` for totals, `ROUND(x::numeric, 4)` for unit prices.
2. **Currency is nullable text on Order and Account.** Multi-currency portfolios exist. No FX rate table — if conversion is needed, return `sql: null`.
3. **Timestamps are UTC `TIMESTAMP(3)`.** Use `date_trunc('month', date)` for grouping, `CAST(date AS date)` for daily.
4. **Exclude drafts** by default: `WHERE o."isDraft" = false`.
5. **Exclude hidden accounts** by default: `WHERE NOT a."isExcluded"`.
6. **Prisma column quoting.** Columns use camelCase in Postgres — always double-quote: `"userId"`, `"unitPrice"`, `"symbolProfileId"`, `"assetClass"`, `"assetSubClass"`, `"isExcluded"`, `"isDraft"`, `"marketPrice"`, `"accountUserId"`.
7. **AssetSubClass is `MUTUALFUND`** (no underscore), not `MUTUAL_FUND`.
8. **SymbolProfileOverrides** may override assetClass, assetSubClass, name, sectors, countries, holdings. Always COALESCE when accuracy matters.
9. **MarketData.state** — filter to `state = 'CLOSE'` for end-of-day prices. `INTRADAY` rows may appear for live data.
10. **Order.currency is nullable.** Use `COALESCE(o.currency, sp.currency)` when currency is needed.
