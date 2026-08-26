---
name: duplicate-detection
description: Scan for potential double-payments by comparing vendor, amount, and currency across documents. Investigate suspicious matches.
---

# Duplicate Detection

Help users find invoices or receipts that may have been paid twice. Booksmate identifies matches by vendor name, total amount, and currency within a date range.

## Workflow

1. **Confirm organisation** — call `list_organizations` if needed.

2. **Set the scope** — ask the user what time period to check. Common defaults: "this year", "last quarter", "since January". Compute ISO dates (YYYY-MM-DD) for `date_start` and `date_end`.

3. **Run duplicate scan** — call `find_duplicates(org_id, date_start, date_end)`. The response groups potential duplicates by vendor name, total, and currency, with the list of matching document IDs.

4. **Present findings** — for each duplicate group, show:
   - Vendor name
   - Amount and currency
   - Number of matching documents
   - Document IDs

5. **Investigate suspicious matches** — offer to pull details on the documents in each group with `get_document`. Look for:
   - Same vendor, same amount, different dates (possible double-payment)
   - Same vendor, same amount, same date (likely a true duplicate)
   - Same vendor, same amount, different document types (invoice vs receipt for same transaction — may be legitimate)

6. **Recommend next steps** — Booksmate does not support editing or merging documents through this server. If duplicates are found, tell the user the document IDs so they can resolve them in the Booksmate app or check their accounting system.

## Key rules

- `find_duplicates` is read-only — it does not modify or delete anything.
- The tool matches on exact vendor name, exact total, and exact currency. Near-matches or different currencies will not appear.
- Duplicates are suggestions, not definitive judgments — always investigate before concluding a double-payment occurred.
- If no duplicates are found, confirm the scan ran successfully and suggest a wider date range if the user expected results.

## Example exchanges

**User:** "Did I pay any vendor twice this year?"
1. `find_duplicates(org_id, date_start="2026-01-01", date_end="2026-12-31")`
2. Present groups: "Found 3 possible duplicates. The most suspicious is 2 payments to Amazon for $129.99. Want me to check those?"
3. `get_document` on each matching document.
4. Report: "Both are invoices from Amazon dated Jan 15 and Feb 3 for $129.99 each. These look like separate purchases, not a double-payment."

**User:** "Check for duplicates from last quarter."
1. `find_duplicates(org_id, date_start="2026-04-01", date_end="2026-06-30")`
2. Report findings or "No duplicates found in Q2 2026."
