# Direct database queries

Source: Plugin Check 2.2.0, `prompts/ai-review-direct-db-queries.md`, rewritten.

A direct query issue is SQL run without being prepared, which can allow SQL injection.

## Real issue

- A query with variable data does not go through `$wpdb->prepare()`.
- An `IN ( ... )` list is built from a dynamic array without one placeholder per item.
- A column name comes from input with no allow list. Column names can't be prepared, so they must be checked against a fixed list.

## Not an issue

- The query uses only hard-coded values.
- `$wpdb->insert()`, `$wpdb->update()`, `$wpdb->delete()` or `$wpdb->replace()` with format arguments; they prepare their own values.
- A table name built with `$wpdb->prefix` or a core property such as `$wpdb->posts`. Table names can't be prepared, and these come from a trusted source.
- A variable interpolated into the SQL that you have traced to a source the user can't control. Name the source in the summary.
