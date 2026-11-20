# Booksmate

Fetch invoices and receipts from email and supplier portals, then work them
in chat.

Booksmate pulls documents from connected Gmail and Microsoft mailboxes and
from supplier web portals, extracts the fields with AI, and exports to Xero
or Google Drive. This plugin connects Claude Code to your Booksmate account
through the hosted MCP server at
[mcp.booksmate.com](https://mcp.booksmate.com). Ask for last month's Amazon
invoices and you get the real rows out of your account, not a guess.

Email ingestion runs inside the plugin. Web portal fetching runs in the
Booksmate desktop app, because portals need a real browser session.

## Install

Clone the public plugin repository and load this directory in Claude Code:

```shell
git clone https://github.com/odwyersoftware/booksmate-plugins.git
claude --plugin-dir booksmate-plugins/claude
```

`--plugin-dir` loads the plugin for that session. The plugin ships a
`.mcp.json` at this directory root pointing at the hosted server over
streamable HTTP. The server entry sets `"type": "http"`, which Claude Code
requires on a URL. There is nothing else to configure.

Only `plugin.json` belongs in `.claude-plugin/`. Skills stay in `skills/`
next to `.mcp.json`.

## Sign in

There is no API key. Authentication is OAuth 2.1 with dynamic client
registration, so the first tool call opens a browser for consent. Approve it
once and the client holds the token from then on. If you have no Booksmate
account yet, sign up at [booksmate.com](https://booksmate.com/?utm_source=claude&utm_medium=plugin_directory&utm_campaign=mcp_plugin) first.

Revoke access any time from your Booksmate account settings.

## Network endpoints

Every request goes to `mcp.booksmate.com` and nowhere else.

| Purpose | Endpoint |
| --- | --- |
| MCP server | `https://mcp.booksmate.com` |
| OAuth issuer | `https://mcp.booksmate.com/` |
| Authorization | `https://mcp.booksmate.com/oauth/authorize` |
| Token | `https://mcp.booksmate.com/oauth/token` |
| Client registration | `https://mcp.booksmate.com/oauth/register` |

Discovery metadata lives at
`https://mcp.booksmate.com/.well-known/oauth-authorization-server`.

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
[booksmate.com/docs/features/mcp-server/](https://booksmate.com/docs/features/mcp-server/?utm_source=claude&utm_medium=plugin_directory&utm_campaign=mcp_plugin).

## About this directory

This `claude/` tree is the Claude Code surface of the `booksmate-plugins`
artifact repository. Its contents, including `skills/` and `assets/`, are
generated from the Booksmate source repository and pushed here by CI, so
direct edits get overwritten on the next sync. File issues and suggestions
at [booksmate.com](https://booksmate.com/?utm_source=claude&utm_medium=plugin_directory&utm_campaign=mcp_plugin) or email richard@booksmate.com.

## License

MIT.
