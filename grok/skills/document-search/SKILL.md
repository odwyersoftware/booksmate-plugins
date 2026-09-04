---
name: document-search
description: Find specific invoices and receipts by vendor, date, amount, status, or keyword. Inspect individual document details including extracted fields and line items.
---

# Document Search

Help users locate and inspect specific invoices and receipts in their Booksmate account.

## Workflow

1. **Confirm organisation** — if the user has not picked an org, call `list_organizations` first.

2. **Translate the user's request into filters** — `search_documents` supports many filters. Map natural language to the right parameters:

   | User says | Parameter |
   |---|---|
   | "from Amazon", "Amazon invoices" | `vendor_name="Amazon"` |
   | "last week", "March 2026" | `date_start` / `date_end` (ISO format) |
   | "invoices" | `document_type="invoice"` |
   | "receipts" | `document_type="receipt"` |
   | "unreviewed" / "pending" | `status` (ask or try common values) |
   | "over $100" | `amount_min=100` |
   | "exported" / "not exported yet" | `exported=true` / `exported=false` |
   | "failed to scan" | `failed_extraction=true` |
   | "largest first" | `sort_by="-total"` |

3. **Run the search** — call `search_documents` with `org_id` and the relevant filters. Keep `limit` reasonable (default 50). Use `offset` if the user asks for more.

4. **Present results** — summarise the matching documents with vendor, date, total, currency, and status. If the result set is large, offer to narrow with additional filters.

5. **Show detail on request** — when the user picks a specific document, call `get_document(document_id, org_id)` to show extracted fields (vendor, total, tax, category, subcategories, line items) and export status.

## Key rules

- Dates must be ISO (YYYY-MM-DD). Convert "last week", "March", "Q1 2026" before calling.
- `vendor_name` does partial matching — "Amazon" matches "Amazon Web Services" too.
- The `category` field must be an exact category name — use `get_taxonomy` first if the user uses a natural-language term.
- Prepend `-` to `sort_by` for descending order (e.g. `-date`, `-total`).
- If no results are returned, suggest broadening the date range or removing restrictive filters.

## Example exchanges

**User:** "Show me my Amazon invoices from last month."
1. Compute dates: last month = previous full calendar month in YYYY-MM-DD.
2. `search_documents(org_id, vendor_name="Amazon", document_type="invoice", date_start="2026-06-01", date_end="2026-06-30")`
3. Present list. Ask if they want to see details on any specific document.

**User:** "What's the biggest invoice I've received this year?"
1. `search_documents(org_id, document_type="invoice", date_start="2026-01-01", sort_by="-total", limit=1)`
2. Call `get_document` on the result to show full detail.
