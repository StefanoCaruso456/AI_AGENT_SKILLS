# Chart Mapping Rules

Rules for selecting chart type and axis defaults based on the intent object.

## Intent → Chart Type

| intent_type | time_granularity | group_by | Default chart_type |
|---|---|---|---|
| `time_series` | any non-null | null | `line` |
| `time_series` | any non-null | non-null | `area` (stacked) |
| `aggregation` | null | non-null, ≤6 categories | `pie` |
| `aggregation` | null | non-null, >6 categories | `bar` |
| `aggregation` | non-null | non-null | `stacked_bar` |
| `comparison` | null | non-null | `bar` |
| `comparison` | non-null | non-null | `stacked_bar` |
| `detail` | any | any | `table` |
| `anomaly` | non-null | any | `line` (with threshold markers) |
| `anomaly` | null | any | `table` |

## Override Rules

- If the SQL builder explicitly set `chart_type`, use it unless it conflicts with the data shape.
- `pie` requires exactly one numeric y_field and one categorical x_field with ≤12 slices. If more, downgrade to `bar`.
- `table` is always safe as a fallback.

## Axis Defaults

### x_field mapping

| intent_type | Default x_field |
|---|---|
| `time_series` | The `date_trunc()` alias (e.g. `period`) |
| `aggregation` | The `GROUP BY` column (e.g. `asset_class`, `currency`) |
| `detail` | First column in result schema |

### y_field mapping

| intent_type | Default y_field |
|---|---|
| `time_series` | The `SUM()` / aggregate alias (e.g. `total_value`) |
| `aggregation` | The `SUM()` / aggregate alias (e.g. `activity_value`) |
| `detail` | Not applicable (table shows all columns) |

## Formatting Defaults

| Data type | decimals | prefix | suffix |
|---|---|---|---|
| Monetary (USD) | 2 | `$` | null |
| Monetary (EUR) | 2 | `€` | null |
| Monetary (GBP) | 2 | `£` | null |
| Monetary (other/null) | 2 | null | null |
| Percentage | 2 | null | `%` |
| Count | 0 | null | null |
| Quantity (shares) | 4 | null | null |

If `intent.currency` is set, look up the symbol from this map:

| Currency | Prefix |
|---|---|
| USD | `$` |
| EUR | `€` |
| GBP | `£` |
| CHF | `CHF ` |
| JPY | `¥` |
| CAD | `CA$` |
| AUD | `A$` |

For unlisted currencies, use the ISO code followed by a space (e.g. `SEK `).

## Sorting Defaults

| chart_type | Default sort |
|---|---|
| `line`, `area` | `{ "field": x_field, "direction": "asc" }` (chronological) |
| `bar`, `stacked_bar` | `{ "field": y_field, "direction": "desc" }` (largest first) |
| `pie` | `{ "field": y_field, "direction": "desc" }` |
| `table` | `{ "field": first_column, "direction": "asc" }` |
