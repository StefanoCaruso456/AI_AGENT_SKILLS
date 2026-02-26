---
name: financial-dashboard-builder
description: Convert SQL builder output (intent + dataset metadata) into a renderable dashboard configuration JSON. Use after natural-language-sql-builder has produced a query and intent. Does not generate SQL or execute queries.
---

# Financial Dashboard Builder

Convert a structured intent object (from `natural-language-sql-builder`) into a dashboard configuration spec ready for UI rendering.

## Inputs

- `intent` — the intent object from the SQL builder output
- `sql_result_schema` — column names and types from the query result
- `dashboard_template_id` — optional override; if omitted, derived from `intent.dashboard_id`

## Output Contract

Return **strict JSON only** matching [dashboard_output_contract.json](references/dashboard_output_contract.json). No prose, no markdown, no explanations.

## Core Rules

1. **Never generate SQL.** This skill only produces dashboard layout configuration.
2. **Never execute queries.** It maps intent metadata to visual specifications.
3. **Chart type selection** follows [chart_mapping_rules.md](references/chart_mapping_rules.md).
4. **Use dashboard templates** from [dashboard_templates.md](references/dashboard_templates.md) as the starting point. Customize tile fields based on the actual `sql_result_schema`.
5. **Formatting defaults:** monetary values get `{ "decimals": 2, "prefix": "$" }`, percentages get `{ "decimals": 2, "suffix": "%" }`.
6. **Currency-aware formatting.** If `intent.currency` is set, use that currency's symbol as prefix. If null, omit currency prefix.

## Failure Modes

| Condition | Action |
|---|---|
| Unknown `dashboard_template_id` | Fall back to single-tile layout using intent fields directly |
| `sql_result_schema` is empty | Return config with placeholder tile, `error: "no_schema"` |
| `intent.requires_external_data` is true | Return config with warning tile explaining the data gap |

## References

- [references/dashboard_output_contract.json](references/dashboard_output_contract.json) — output JSON schema
- [references/chart_mapping_rules.md](references/chart_mapping_rules.md) — intent → chart type mapping
- [references/dashboard_templates.md](references/dashboard_templates.md) — pre-built dashboard layouts
