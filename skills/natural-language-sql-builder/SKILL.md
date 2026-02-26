---
name: natural-language-sql-builder
description: Convert portfolio analytics requests into safe, scoped, SELECT-only Postgres SQL and structured intent JSON for dashboard rendering. Use when the user asks about portfolio value, allocations, fees, dividends, activity, symbols, watchlists, or any dashboard data need.
---

# Natural Language SQL Builder

Translate a user's portfolio analytics request into a safe Postgres SELECT query, named parameters, and a rendering intent object.

## Inputs

- `user_request` — natural language question
- `userId` — authenticated user ID (always required)
- Optional filters: `date_from`, `date_to`, `accountId`, `currency`, `symbol`, `type`

## Output Contract

Return **strict JSON only** matching [sql_output_contract.json](references/sql_output_contract.json). No prose, no markdown, no explanations.

```json
{ "sql": "SELECT …" | null, "params": {}, "intent": {} }
```

## Core Rules

1. **SELECT only.** WITH (CTE) is allowed. No INSERT/UPDATE/DELETE/DROP/ALTER/TRUNCATE.
2. **Always scope by `:userId`.** Every query must filter on the userId column of the primary table.
3. **Composite Account join.** When joining Account: `Order.accountId = Account.id AND Order.accountUserId = Account.userId`. Same pattern for AccountBalance.
4. **MarketData convention join.** No FK exists — join by `MarketData.dataSource = SymbolProfile.dataSource AND MarketData.symbol = SymbolProfile.symbol`.
5. **Round monetary values.** Totals: `ROUND(x::numeric, 2)`. Unit prices: `ROUND(x::numeric, 4)`.
6. **Use `date_trunc()`** for time-based grouping. Cast to `date` for daily granularity.
7. **Detail queries** must include `ORDER BY` and `LIMIT 200`.
8. **Prefer canonical templates** from [query_catalog.md](references/query_catalog.md) before writing freeform SQL.

## Failure Modes

| Condition | Action |
|---|---|
| FX conversion needed, no rate table | `sql: null`, set `requires_external_data: true` |
| Request targets auth/secret fields | `sql: null`, empty `missing_requirements` |
| Ambiguous request | Choose the safest aggregate interpretation |
| Table/column not in schema | `sql: null`, list gap in `missing_requirements` |

## References

- [references/schema.md](references/schema.md) — table definitions, joins, gotchas
- [references/safety_rules.md](references/safety_rules.md) — hard constraints
- [references/sql_output_contract.json](references/sql_output_contract.json) — output JSON schema
- [references/query_catalog.md](references/query_catalog.md) — canonical query templates
