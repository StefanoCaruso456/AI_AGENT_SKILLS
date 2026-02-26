# Safety Rules

Hard constraints enforced before any SQL generation.

## 1. Statement Type

- **Allowed:** `SELECT`, `WITH … SELECT`
- **Forbidden:** `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, `TRUNCATE`, `GRANT`, `REVOKE`, `COPY`, `CREATE`, `EXECUTE`
- If any forbidden keyword appears as a statement verb, return `sql: null`.

## 2. Mandatory User Scoping

Every query **must** filter by `:userId` on the primary table:

| Table | Required Clause |
|---|---|
| Order | `WHERE o."userId" = :userId` |
| Account | `WHERE a."userId" = :userId` |
| AccountBalance | `WHERE ab."userId" = :userId` |
| Tag | `WHERE t."userId" = :userId OR t."userId" IS NULL` |
| SymbolProfile | Scoped indirectly via Order or `_UserWatchlist` join |
| MarketData | Scoped indirectly via SymbolProfile join |

If a query cannot be scoped to a single user, return `sql: null`.

## 3. Detail Query Limits

Any query returning individual rows (intent_type = `detail`) must include:

```sql
ORDER BY <deterministic_column> DESC
LIMIT 200
```

Maximum allowed LIMIT value: **1000**.

## 4. Money Rounding

All monetary output columns must use `ROUND()`:

| Context | Rule |
|---|---|
| Totals, sums, balances | `ROUND(expr::numeric, 2)` |
| Unit prices, per-share | `ROUND(expr::numeric, 4)` |
| Percentages | `ROUND(expr::numeric, 2)` |

## 5. Forbidden Columns

Never select or filter on:

- `"ApiKey"."hashedKey"`
- `"AuthDevice"` credential columns
- `"User"."accessToken"`
- Any column containing passwords, tokens, or secrets

If a request targets these, return `sql: null` with empty `missing_requirements`.

## 6. FX / Currency Conversion

The schema has no FX rate table. If the user request implies conversion between currencies:

1. Set `sql: null`
2. Set `intent.requires_external_data: true`
3. Add `"FX rate data not available in schema"` to `intent.missing_requirements`

Single-currency queries (filtering to one currency) are fine.

## 7. Draft Exclusion

Always add `WHERE o."isDraft" = false` when querying the Order table unless the user explicitly asks about drafts.

## 8. Excluded Accounts

When joining Account, add `WHERE NOT a."isExcluded"` unless the user explicitly asks about excluded accounts.

## 9. Time Boundaries

- Default lookback when unspecified: 6 months (`date >= NOW() - INTERVAL '6 months'`).
- Always use parameterized dates: `:date_from`, `:date_to`.
- Use `date_trunc()` for grouped time series, never string formatting.

## 10. Parameter Safety

- All user-provided filter values must be named parameters (`:userId`, `:date_from`, etc.).
- Never interpolate values directly into SQL strings.
- Parameter names use snake_case.
