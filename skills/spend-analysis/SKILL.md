---
name: spend-analysis
description: Analyze business spending by category, vendor, month, or type. Summarise totals, spot trends, and drill into details.
---

# Spend Analysis

Help users understand where their money goes using Booksmate spend data.

## Workflow

1. **Identify the organisation** — call `list_organizations` if the user has not already selected one. Ask which org if multiple are returned.

2. **Resolve natural-language categories** — when the user asks about spending on something like "software", "hotels", or "travel", call `get_taxonomy` first. The response maps natural-language phrases to internal enum keys via its `synonyms` field. The `categories` array lists every valid key and label pair. Use the **enum key** (snake_case, e.g. `it_technology`) as the `category` parameter — labels like "IT & Technology" will be rejected with a 400 error. If the user's phrase matches a subcategory synonym, also pass the corresponding `subcategory` to narrow results.

3. **Get the summary** — call `spend_summary` with the resolved filters. Always include `org_id`. For relative dates ("last month", "this quarter"), compute ISO dates (YYYY-MM-DD) before calling:
   - `group_by` is required — choose `category`, `vendor`, `month`, `subcategory`, `currency`, `document_type`, or `direction` based on the user's question.
   - Use `top_n` to limit results when the user asks for "top 5" or "biggest".

4. **Present the answer** — always lead with the overall total, then show the breakdown. Include currency, subtotal, tax, and document count for each group. Warn the user that `subcategory` grouping can double-count documents that have multiple subcategories, so subcategory totals may exceed the overall.

5. **Offer to drill in** — after showing the summary, ask if the user wants to see the actual documents behind a particular group. If yes, call `search_documents` with the same category/vendor/date filters.

## Key rules

- Never call `spend_summary` without `group_by`.
- Always call `get_taxonomy` before filtering by category or subcategory on a natural-language term.
- Pass enum keys (e.g. `it_technology`, `telecoms_internet`) to `category` and `subcategory` parameters, not display labels. The taxonomy response maps synonyms to the correct keys.
- Totals never cross currencies — each group carries its own currency.
- Pass `currency` filter only when the user explicitly asks for a specific currency.
- For time-based grouping (`month`), set `group_by=month` and narrow with `date_start`/`date_end`.

## Example exchanges

**User:** "How much did I spend on software last quarter?"
1. `get_taxonomy` — in the response, `synonyms.software` maps to `{category: "it_technology", subcategory: "software"}`
2. `spend_summary(org_id, group_by="category", category="it_technology", subcategory="software", date_start="2026-04-01", date_end="2026-06-30")`
3. Present total, then ask if they want a vendor breakdown or the individual invoices.

**User:** "What's my biggest expense category this year?"
1. `spend_summary(org_id, group_by="category", date_start="2026-01-01", date_end="2026-12-31", top_n=5)`
2. Present ranked breakdown, offer to drill into the top category.

**User:** "Show my travel spending for 2026."
1. `get_taxonomy` — `synonyms.travel` maps to `{category: "travel"}`
2. `spend_summary(org_id, group_by="month", category="travel", date_start="2026-01-01", date_end="2026-12-31")`
3. Present monthly breakdown.
