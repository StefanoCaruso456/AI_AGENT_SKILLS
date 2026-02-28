# AI Agent Skills

Composable Cursor agent skills for Ghostfolio portfolio analytics. Each skill produces strict JSON — no prose, no UI, no side effects.

## Pipeline

```
User request
  → dashboard-recommender     (selects best dashboard template)
  → natural-language-sql-builder  (generates safe SELECT query + intent)
  → financial-dashboard-builder   (produces renderable dashboard config)
```

## Skills

| Skill | Purpose | Output |
|---|---|---|
| [dashboard-recommender](skills/dashboard-recommender/SKILL.md) | Route a natural-language request to the best dashboard template | `{ selected_dashboard_id, confidence, reason_codes, required_filters }` |
| [natural-language-sql-builder](skills/natural-language-sql-builder/SKILL.md) | Convert request into a safe, user-scoped Postgres SELECT query | `{ sql, params, intent }` |
| [financial-dashboard-builder](skills/financial-dashboard-builder/SKILL.md) | Map SQL intent into a tile-based dashboard configuration | `{ title, tiles[], global_filters }` |

## Key Constraints

- **SELECT only** — no mutations, ever
- **User-scoped** — every query filters by `:userId`
- **Composite Account PK** — joins require both `id` and `userId`
- **Money rounding** — `ROUND(x::numeric, 2)` for totals, `ROUND(x::numeric, 4)` for unit prices
- **No FX conversion** — returns `sql: null` with `requires_external_data: true` when cross-currency math is needed

## Schema

Verified against Ghostfolio's `prisma/schema.prisma`. Full reference: [schema.md](skills/natural-language-sql-builder/references/schema.md)

Core tables: `Order`, `Account` (composite PK), `AccountBalance`, `SymbolProfile`, `MarketData` (convention join), `Tag`, `_UserWatchlist`, `_OrderToTag`

## Ghostfolio Tool Mapping

SQL templates cross-reference the 10 canonical AI service tools (`getPortfolioSummary`, `listActivities`, `getAllocations`, `getPerformance`, `getQuote`, `getHistory`, `getFundamentals`, `getNews`, `computeRebalance`, `scenarioImpact`). See the [Tool Coverage Matrix](skills/natural-language-sql-builder/references/query_catalog.md#tool-coverage-matrix).

## Structure

```
skills/
├── dashboard-recommender/
│   ├── SKILL.md
│   └── references/
├── financial-dashboard-builder/
│   ├── SKILL.md
│   └── references/
└── natural-language-sql-builder/
    ├── SKILL.md
    └── references/
```

Each `SKILL.md` stays lean (<70 lines). Heavy detail lives in `references/` (progressive disclosure).
