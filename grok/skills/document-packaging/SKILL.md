---
name: document-packaging
description: Bundle documents into a ZIP archive with a signed download URL for sharing with accountants, auditors, or tax preparation.
---

# Document Packaging

Help users create downloadable ZIP archives of their documents for sharing with accountants, auditors, or for tax preparation.

## Workflow

1. **Confirm organisation** — call `list_organizations` if needed.

2. **Identify the documents to include** — use `search_documents` to find the relevant documents based on the user's criteria:
   - Tax preparation: filter by `date_start`/`date_end` for the tax year, `document_type` for invoices or receipts.
   - Accountant handoff: filter by status or date range as the user specifies.
   - Audit request: filter by vendor, date range, or specific document IDs.

3. **Summarise and confirm** — present the list of documents that will be packaged (count, total value, date range) and ask the user to confirm before proceeding.

4. **Package with naming patterns** — call `package_documents` with:
   - `org_id`: required.
   - `document_ids`: the list of document IDs to include.
   - `folder_pattern` (optional): use tokens like `{{vendor}}` to group by vendor, `{{year}}` for year.
   - `filename_pattern` (optional): use tokens like `{{vendor}}_{{documentId}}` for descriptive filenames.

5. **Share the download link** — the response includes a `download_url` and `expires_at`. Tell the user the URL and when it expires. The URL is a short-lived signed link — the user or their accountant must download it before expiry.

## Naming pattern examples

- No patterns: all files in a flat ZIP.
- `folder_pattern="{{vendor}}"`, `filename_pattern="{{vendor}}_{{documentId}}_{{date}}"` — groups by vendor folder, files named "Amazon_1042_2026-03-15.pdf".
- `folder_pattern="{{year}}/{{month}}"` — organises by year/month folders.

## Key rules

- The download URL is temporary — remind the user of the expiry time.
- Always confirm the document count before packaging — packaging large sets is fine, but the user should know what they are getting.
- Tokens like `{{vendor}}` and `{{documentId}}` come from the document metadata. If a document is missing a field (e.g. no vendor extracted), the token renders as empty.
- Do not include documents the user did not explicitly request.

## Example exchanges

**User:** "Package all my 2025 invoices for my accountant."
1. `search_documents(org_id, document_type="invoice", date_start="2025-01-01", date_end="2025-12-31")`
2. "Found 47 invoices totalling $89,230 across 2025. I'll organise them by vendor folder. Confirm?"
3. After confirmation: `package_documents(org_id, document_ids=[...], folder_pattern="{{vendor}}")`
4. "ZIP ready. Download: https://... Expires in 24 hours. Please share this with your accountant before then."

**User:** "Give me a zip of all Amazon receipts from Q1."
1. `search_documents(org_id, vendor_name="Amazon", document_type="receipt", date_start="2026-01-01", date_end="2026-03-31")`
2. `package_documents(org_id, document_ids=[...], filename_pattern="Amazon_{{date}}_{{documentId}}")`
3. Share the link.
