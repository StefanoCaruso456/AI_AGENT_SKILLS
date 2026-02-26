---
name: dashboard-recommender
description: Select the best dashboard template for a natural language portfolio analytics request. Use as the entry point before natural-language-sql-builder and financial-dashboard-builder. Does not generate SQL or UI — outputs a dashboard selection with confidence and required filters.
---

# Dashboard Recommender

Select the best-fit dashboard template for a user's natural language request. This is the first skill in the pipeline:

```
dashboard-recommender → natural-language-sql-builder → financial-dashboard-builder
```

## Inputs

- `user_request` — natural language question (e.g. "Show me my fees this year")
- Optional context: `available_accounts`, `date_range_hint`

## Output Contract

Return **strict JSON only** matching [recommender_output_contract.json](references/recommender_output_contract.json). No prose, no markdown, no explanations.

```json
{
  "selected_dashboard_id": "fee_analysis",
  "alternates": ["portfolio_overview"],
  "confidence": 0.92,
  "reason_codes": ["keyword:fees", "time_hint:ytd"],
  "required_filters": { "date_from": "2026-01-01", "date_to": null, ... }
}
```

## Core Rules

1. **Never generate SQL.** Output is a dashboard selection, not a query.
2. **Never produce UI config.** That is the job of `financial-dashboard-builder`.
3. **Match against the dashboard catalog** in [dashboard_catalog.md](references/dashboard_catalog.md).
4. **Apply routing rules** from [routing_rules.md](references/routing_rules.md) to map keywords/intent to dashboard IDs.
5. **Always return `required_filters`** with date defaults populated when a time window is implied.
6. **Confidence scoring:** 1.0 = exact keyword match + clear time scope. 0.5 = partial match, ambiguous scope. Below 0.5 = include alternates.
7. **Ambiguity resolution:** When multiple dashboards could match, pick the most general one as `selected_dashboard_id` and list others in `alternates`.

## Failure Modes

| Condition | Action |
|---|---|
| No keyword match | Select `portfolio_overview` as safe default, confidence ≤ 0.3 |
| FX conversion implied | Select the dashboard but add `reason_codes: ["fx_required"]` |
| Request outside portfolio domain | `selected_dashboard_id: null`, `reason_codes: ["out_of_domain"]` |

## Date Defaults

When the user implies a time window but doesn't specify exact dates:

| Phrase | `date_from` | `date_to` |
|---|---|---|
| "this month" / "MTD" | 1st of current month | null (now) |
| "this year" / "YTD" | Jan 1 of current year | null |
| "last 3 months" | NOW - 3 months | null |
| "last 6 months" | NOW - 6 months | null |
| "last year" / "12 months" | NOW - 12 months | null |
| No time hint | NOW - 6 months (default) | null |

## References

- [references/recommender_output_contract.json](references/recommender_output_contract.json) — output JSON schema
- [references/dashboard_catalog.md](references/dashboard_catalog.md) — available dashboards
- [references/routing_rules.md](references/routing_rules.md) — keyword → dashboard mapping
