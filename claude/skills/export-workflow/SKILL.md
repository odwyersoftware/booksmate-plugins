---
name: export-workflow
description: Export invoices and receipts to connected accounting targets (Xero) or cloud storage (Google Drive). Review what will be exported before sending.
---

# Export Workflow

Help users push their processed documents from Booksmate to Xero or Google Drive for accounting and record-keeping.

## Workflow

1. **Check export targets** — call `list_exporters(org_id)` to see what is connected. The user needs at least one active exporter (Xero or Google Drive) before exporting.

2. **Identify documents to export** — there are two modes:
   - **Specific documents:** The user names particular documents or a search query. Use `search_documents` with `exported=false` to find candidates.
   - **Bulk export:** The user wants to export everything unexported. Skip the search and call `export_now` without `document_ids` — this exports all unexported documents for the org.

3. **Confirm before exporting** — always present a summary of what will be exported (document count, total value) and ask the user to confirm. Mention the target (Xero, Google Drive). Only call `export_now` after the user confirms.

4. **Execute export** — call `export_now(org_id, document_ids=[...])` with the confirmed list, or `export_now(org_id)` to export all unexported.

5. **Report results** — the response includes a success flag and message. Report any errors and offer to investigate failed documents with `get_document`.

## Key rules

- Always confirm with the user before calling `export_now`. Exporting creates records in external systems that may be difficult to undo.
- Filter by `exported=false` when searching for documents to export — avoid re-exporting.
- If no exporters are connected (`list_exporters` returns empty), tell the user to connect Xero or Google Drive in the Booksmate app first.
- `export_now` without `document_ids` exports all unexported documents — warn the user about the scope before proceeding.
- Exports go to all connected targets simultaneously — the user does not pick one.
- `export_now` queues a background task and returns a generic success or error message; it does not report per-document status, task IDs, or completion counts. Tell the user the export was queued and direct them to verify results in Booksmate or their accounting software.

## Example exchanges

**User:** "Export my invoices from this week to Xero."
1. `search_documents(org_id, document_type="invoice", date_start="2026-07-27", date_end="2026-07-31", exported=false)`
2. Present summary: "Found 4 unexported invoices totalling $3,450. Export to Xero and Google Drive?"
3. After confirmation: `export_now(org_id, document_ids=[101, 102, 103, 104])`
4. Report: "Export queued. The documents are being sent to your connected accounting targets. Check Booksmate or your accounting software to confirm they arrived."

**User:** "Push everything unexported to my accounting."
1. `search_documents(org_id, exported=false)` → "Found 23 unexported documents. This will export all of them. Confirm?"
2. After confirmation: `export_now(org_id)`
3. Report: "Export queued for all 23 documents. Processing may take a moment — verify in Booksmate or your accounting software."
