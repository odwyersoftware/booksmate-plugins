---
name: email-ingestion
description: Ingest invoices and receipts from connected Gmail or Microsoft email accounts. Check plugin status, trigger ingestion, and monitor progress.
---

# Email Ingestion

Help users pull invoices and receipts from their connected email accounts into Booksmate for AI extraction.

## Important constraints

Only **Gmail and Microsoft email plugins** work through this server. Web portal sources (eBay, Amazon, utility portals) require the Booksmate desktop app and cannot be triggered here.

## Workflow

1. **Find the email plugin** — call `list_plugins(org_id)`. Look for instances with `plugin_type="email"`. If none exist, tell the user to install an email plugin in the Booksmate app first.

2. **Check connection health** — call `connection_status(org_id, instance_id)` to verify the plugin is connected and authenticated. If broken, tell the user to reconnect in the Booksmate app.

3. **Review monitored mailboxes** — optionally call `list_mailboxes(org_id, instance_id)` to show which labels or folders are being scanned. If the user wants to change which mailboxes are monitored, use `set_mailboxes(org_id, instance_id, selected_mailboxes=["INBOX", "Receipts"])`.

4. **Start ingestion** — call `trigger_email_ingestion(org_id, instance_id)`. Optionally limit with `max_documents` if the user wants a capped first run. The response includes a `task_id`.

5. **Poll for completion** — call `check_ingestion_status(task_id)` to report progress. Tell the user whether the task is still running, completed, or failed. If completed, report how many documents were ingested.

6. **Verify results** — after ingestion completes, offer to search for the newly ingested documents with `search_documents` filtered by recent dates.

## Key rules

- Always check `connection_status` before triggering — a broken connection means ingestion will fail.
- Do not trigger ingestion without confirming which plugin instance to use if the user has multiple email plugins.
- Web portal plugins (`plugin_type="web_portal"`) cannot be triggered — remind the user they need the desktop app.
- Ingestion runs asynchronously — always poll with `check_ingestion_status` rather than assuming it is instant.

## Example exchanges

**User:** "Check my Gmail for new invoices."
1. `list_plugins(org_id)` → find the Gmail instance.
2. `connection_status(org_id, instance_id)` → confirm it is connected.
3. `trigger_email_ingestion(org_id, instance_id)` → get `task_id`.
4. `check_ingestion_status(task_id)` → report "processing 12 of 45 emails".
5. Repeat step 4 until done. Report final count.

**User:** "Only scan my 'Receipts' folder from now on."
1. `list_plugins(org_id)` → find the email instance.
2. `list_mailboxes(org_id, instance_id)` → show current selection.
3. `set_mailboxes(org_id, instance_id, selected_mailboxes=["Receipts"])` → update.
