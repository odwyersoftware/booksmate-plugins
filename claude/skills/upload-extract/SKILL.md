---
name: upload-extract
description: Upload invoice and receipt files for AI-powered data extraction. Supports PDF, images, and office documents.
---

# Upload & Extract

Help users upload new documents directly into Booksmate for automatic AI extraction. The extraction pulls vendor name, total, tax, line items, category, and currency.

## Workflow

1. **Confirm organisation** — call `list_organizations` if the user has not already selected an org.

2. **Get the file** — the user must provide the file. ChatGPT can handle file uploads directly; the file bytes will be base64-encoded automatically when passed to the tool.

3. **Upload** — call `upload_document(org_id, file_name, file_base64)`. Use the original filename as `file_name` so Booksmate can preserve it. The `file_base64` should be the full base64 encoding of the file bytes.

4. **Confirm upload** — the response includes a success flag. If successful, tell the user the document is queued for extraction. If the file is not valid base64, ask the user to re-upload.

5. **Find the result** — extraction runs asynchronously. Offer to search for the document after a moment:
   - `search_documents(org_id, sort_by="-date", limit=5)` to find recent uploads.
   - Match by filename or ask the user to identify the right document.
   - Use `get_document(document_id, org_id)` to check if extraction completed and show the results.

## Supported formats

Booksmate accepts PDF, PNG, JPG, JPEG, DOCX, and other common document and image formats. If the user provides an unsupported format, let them know and suggest conversion.

## Key rules

- Always include the original filename — it helps the user identify the document later.
- The file must be base64-encoded. If encoding fails, report the error clearly.
- Extraction is asynchronous — do not assume the document is fully processed immediately.
- After upload, proactively offer to search for the document so the user can verify extraction results.
- Auto-export is triggered on successful extraction — the document may already be in Xero or Google Drive if exporters are connected.

## Example exchanges

**User:** (uploads invoice.pdf) "Process this invoice."
1. `upload_document(org_id, file_name="invoice.pdf", file_base64="<base64 content>")`
2. "Uploaded invoice.pdf. Extraction is in progress. I'll check on it..."
3. `search_documents(org_id, sort_by="-date", limit=5)` → find the new document.
4. `get_document(org_id, document_id)` → present extracted vendor, total, category.

**User:** "Upload these 3 receipts." (uploads three images)
1. Call `upload_document` for each file, one at a time.
2. Report progress for each.
3. After all uploads: "All 3 receipts uploaded and queued for extraction. Would you like me to check on them?"
