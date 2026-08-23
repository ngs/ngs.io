---
title: "google-mcp-server: Read and Write Google Workspace from LLMs"
slug: "google-mcp-server"
description: Introducing google-mcp-server, a Go-based MCP server that lets Claude and other MCP clients read and write Google Calendar, Drive, Gmail, Sheets, Docs, and Slides — and how it compares to the claude.ai Google connectors and Google's official Sheets MCP API.
date: "2026-08-12T09:00:00+09:00"
public: true
tags: ["google-mcp-server","mcp","go","google-workspace","claude","oss"]
archives: ["2026-08"]
---

**[google-mcp-server](https://github.com/ngs/google-mcp-server)** is an MCP server I built that lets Claude and other MCP clients read and write Google Calendar, Drive, Gmail, Sheets, Docs, and Slides.

It's a single Go binary, installable via Homebrew:

```bash
brew tap ngs/tap
brew install google-mcp-server
```

<!--more-->

## Motivation

I originally wrote this for myself last year, but recently a client project put it to a real test.

I wanted a Google Spreadsheet shared with the client to work as a TODO sheet that both humans and LLMs read and write.

The claude.ai Google Drive connector could read the spreadsheet, but **couldn't write cell values**.

Google's official [Sheets MCP API](https://developers.google.com/workspace/sheets/api/guides/configure-mcp-server) looked promising, but it's in Developer Preview, covers Sheets only, and wasn't something I could plug in right away in my environment.

So I deployed my own google-mcp-server instead — and promptly hit a few bugs in production use, including diagnostics leaking into stdout and corrupting the JSON-RPC stream. I fixed them on the spot and shipped v0.4.0 and v0.5.0.

Now that it has survived a real project, it feels worth a proper introduction.

## How it compares

There are roughly three ways to let an LLM touch Google Workspace:

| | claude.ai Google connectors | Google's Sheets MCP API | google-mcp-server |
|---|---|---|---|
| Services | Drive / Gmail / Calendar, etc. | Sheets only | Calendar / Drive / Gmail / Sheets / Docs / Slides |
| Cell-level writes | No (as of 2026-08, in my environment) | Yes | Yes |
| Hosting | Built into claude.ai | Google-hosted, HTTP + OAuth | Local stdio, single binary |
| Status | Generally available | Developer Preview | OSS (MIT) |
| Setup | Connect from the UI | Enable APIs in GCP + per-client redirect URI | Create an OAuth client in GCP + drop in the binary |
| Multiple accounts | One connection per account | — | One server, automatic account selection |

**The claude.ai connectors** are by far the easiest to set up. If all you need is Drive search, file reading, and browsing mail and calendars, they're enough. They don't offer spreadsheet cell updates or Slides manipulation, though.

**Google's Sheets MCP API** is a managed MCP server you reach over HTTP + OAuth 2.0 after enabling `sheetsmcp.googleapis.com`. It provides tools like `get_values`, `update_values`, `update_formulas`, and `insert_dimension`. Google hosts it, so there's nothing to run — but it's Developer Preview, Sheets-only, and each MCP client needs its own redirect URI registered.

**google-mcp-server** runs locally over stdio and covers all six services in one binary. It uses an OAuth client from your own GCP project, so tokens and data stay between your machine and Google's APIs.

## Usage

Once registered with an MCP client, requests like these work in plain language:

- "Append today's work to this TODO sheet and update the status column" (`sheets_values_get` / `sheets_values_update`)
- "Turn this Markdown meeting note into a Google Doc in the shared Drive folder" (`drive_markdown_upload`)
- "Find three free slots next week" (`calendar_freebusy_query`)
- "Build a slide deck from this Markdown and export it as PDF" (`slides_markdown_create` / `slides_export_pdf`)

You can authenticate multiple Google accounts, and the server **picks the right account automatically** from email addresses or domains mentioned in your request.

Say "add this to my work calendar" and it calls the API with that account's token. There are also cross-account tools like `calendar_events_list_all_accounts`.

## Setup

You need your own Google Cloud project and an OAuth 2.0 client (Desktop app type).

(1) Create a project in Google Cloud Console and enable the APIs you plan to use (Google Sheets API, Google Drive API, and so on).

(2) Under "APIs & Services > Credentials", create an OAuth client ID with the "Desktop app" type.

(3) Pass the client ID and secret via environment variables or `config.json`:

```bash
export GOOGLE_CLIENT_ID="YOUR_CLIENT_ID.apps.googleusercontent.com"
export GOOGLE_CLIENT_SECRET="YOUR_CLIENT_SECRET"
```

(4) Register it with your MCP client.

For Claude Code:

```bash
claude mcp add google -- /opt/homebrew/bin/google-mcp-server
```

For Claude Desktop, add this to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "google": {
      "command": "/opt/homebrew/bin/google-mcp-server"
    }
  }
}
```

On first run it opens a browser for the OAuth consent flow. Once you grant access, tokens are stored and refreshed automatically.

To add more accounts, just ask your client to "add a Google account" — the `accounts_add` tool returns an authorization URL.

## Feedback welcome

I'm still fixing things as real-world usage surfaces them, so bug reports, feature requests, and pull requests are all welcome.

Feel free to open an issue on the [GitHub repository](https://github.com/ngs/google-mcp-server).
