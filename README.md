# Booksmate

Fetch invoices and receipts from email and supplier portals, then work them
in chat.

Booksmate pulls documents from connected Gmail and Microsoft mailboxes, and
from supplier web portals via the Booksmate desktop app. It extracts fields
with AI and exports to Xero or Google Drive. This plugin connects an agent to
your account through the hosted MCP server at
[mcp.booksmate.com](https://mcp.booksmate.com). Email ingestion runs here.
Web portal fetching stays on the desktop app. Ask for last month's Amazon
invoices and you get the real rows out of your account, not a guess.

## Install

### Cursor

Open the Cursor Marketplace and search for "Booksmate", then click install.
You can also open Customize and search there.

The plugin ships an `mcp.json` pointing at the hosted server. Cursor picks it
up automatically. There is nothing to configure.

### Grok Bot

Add the server URL `https://mcp.booksmate.com` as an MCP connector. The
skills in this repo are plain `skills/*/SKILL.md` files, so any agent that
reads skill directories can use them alongside the connector.

### Claude Code

Clone https://github.com/odwyersoftware/booksmate-plugins and start Claude
Code with `claude --plugin-dir booksmate-plugins`.

That flag loads the repository root for the session. It ships a
`.mcp.json` pointing at the hosted server over streamable HTTP. The server
entry sets `"type": "http"`, which Claude Code requires on a URL. There is
nothing else to configure.

### Any other MCP client

Point the client at `https://mcp.booksmate.com`. The transport is streamable
HTTP.

## Sign in

There is no API key. Authentication is OAuth 2.1 with dynamic client
registration, so the first tool call opens a browser for consent. Approve it
once and the client holds the token from then on. If you have no Booksmate
account yet, sign up at [booksmate.com](https://booksmate.com/?utm_source=github&utm_medium=referral&utm_campaign=mcp_plugin) first.

Revoke access any time from your Booksmate account settings.

## What you can ask for

- Show my invoices from last month.
- What did I spend on software this quarter, broken down by vendor?
- Find any duplicate payments over 500 in the last 90 days.
- Upload this receipt PDF and pull out the vendor and total.
- Import my latest receipts from Gmail.
- Export everything unexported to Xero.
- Zip up my Q1 invoices and give me a download link for my accountant.

## Skills

Seven skills teach the agent how to drive the tools well.

| Skill | What it covers |
| --- | --- |
| `document-search` | Find and inspect invoices and receipts by vendor, date, amount, or status |
| `spend-analysis` | Totals and trends by category, vendor, month, or document type |
| `upload-extract` | Upload PDFs, images, and office files for AI extraction |
| `duplicate-detection` | Catch double payments by comparing vendor, amount, and currency |
| `email-ingestion` | Pull documents from connected Gmail and Microsoft mailboxes |
| `export-workflow` | Send documents to Xero or Google Drive |
| `document-packaging` | Bundle documents into a ZIP with a signed download link |

## Docs

Full documentation lives at
[booksmate.com/docs/features/mcp-server/](https://booksmate.com/docs/features/mcp-server/?utm_source=github&utm_medium=referral&utm_campaign=mcp_plugin).

## Download packages

Download the latest platform packages from the
[GitHub Releases page](https://github.com/odwyersoftware/booksmate-plugins/releases/latest).
The release includes separate ZIP files for Cursor, Claude, Grok, and OpenAI.

## About this repository

This repository is a build artifact. Its contents are generated from the
Booksmate source repository and pushed here by CI. Direct edits to
`.cursor-plugin/`, `mcp.json`, `skills/`, `assets/`, `grok/`, `claude/`,
and this README get overwritten on the next sync. File issues and
suggestions at
[booksmate.com](https://booksmate.com/?utm_source=github&utm_medium=referral&utm_campaign=mcp_plugin) or email richard@booksmate.com.

## License

MIT.
