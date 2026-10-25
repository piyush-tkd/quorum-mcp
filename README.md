# Quorum MCP server

Give Claude, Cursor, VS Code, Windsurf, Codex or your own agent access to your databases through [Quorum](https://www.quorumdb.ai), with the same rules as Quorum's editor: read-only sessions, masking, and a person approving every write.

- Website: https://www.quorumdb.ai/mcp
- Guide: [How to connect Claude, OpenAI or Cursor to your database safely over MCP](https://www.quorumdb.ai/guides/connect-ai-to-database-mcp)

## Contents

- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Set up an AI client](#set-up-an-ai-client)
- [Choose the connections AI clients may use](#choose-the-connections-ai-clients-may-use)
- [Tools](#tools)
- [How writes are handled](#how-writes-are-handled)
- [Quorum Gate](#quorum-gate)
- [The quorum command](#the-quorum-command)
- [About this repository](#about-this-repository)

## How it works

MCP (Model Context Protocol) is how AI tools call outside tools. Quorum ships an MCP server that the AI client starts on its side. The server talks to the running Quorum app over a local bridge on your computer.

- **Quorum must be open.** The AI client reaches your databases only while Quorum runs.
- **No passwords leave Quorum.** The client sends a request; Quorum runs it with its own saved login.
- **Each connection is off for AI clients until you turn it on.**
- **Same rules as the editor:** read-only sessions for reads, masking, and approval for writes.

Reads run in a read-only session and return up to 10,000 rows. Masked columns come back masked, and a query that names a masked column is refused. On DuckDB and SQLite, AI clients cannot read or write files on your computer.

## Requirements

- The Quorum desktop app, version 4.0.0 or later, on macOS, Windows or Linux. Quorum is a paid product with a free 14-day trial coming soon; request access at https://www.quorumdb.ai/#access to hear first.
- An MCP client. Quorum sets up Claude Desktop, Claude Code, Cursor, VS Code, Windsurf and Codex for you, and gives you the configuration to copy for any other client.

The server runs on the Node runtime inside Quorum, so nothing else needs installing.

## Set up an AI client

1. In Quorum, open **Settings › AI clients**.
2. Under **1. AI clients**, find the client. Its status is Set up, Not set up, Not found, or Set up for another copy.
3. Click **Set up**. Quorum adds its entry to the client's own configuration file and keeps everything else in it.
4. Quit the client and open it again.
5. Under **2. Connections the AI may use**, turn on the connections it should reach.

For another client, open **Another AI client: copy the config**, click **Copy** and paste it into that client's MCP settings.

| Client | Configuration file Quorum writes (macOS) |
|---|---|
| Claude Desktop | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Code | `~/.claude.json` |
| Cursor | `~/.cursor/mcp.json` |
| VS Code | `~/Library/Application Support/Code/User/mcp.json` |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |
| Codex | `~/.codex/config.toml`, or the folder in `CODEX_HOME` |

Linux uses `~/.config` and Windows uses `%APPDATA%` for the Claude Desktop and VS Code rows.

## Choose the connections AI clients may use

AI clients see no connection until you share it. In **Settings › AI clients**, click **Choose connections…**, search or browse by environment, and turn on **Let AI clients use** for each one.

- A shared connection is readable only while **Allow AI features on this connection** is on.
- Where **AI may see row values** is off, results reach the client as a profile of each column, not the values.
- Imported and shared connections always arrive with AI clients off.

## Tools

| Tool | What it does |
|---|---|
| `quorum_list_connections` | Lists the shared connections and how their writes are handled. |
| `quorum_get_schema` | Reads a connection's schema: tables, views, columns, types and keys. |
| `quorum_describe_table` | Describes one table, with the team's descriptions. |
| `quorum_search_schema` | Finds tables and columns by name. |
| `quorum_sample_rows` | Reads up to 100 rows of a table. Masked columns come back masked. |
| `quorum_explain_query` | Shows the plan of a single read. EXPLAIN only, so nothing runs. |
| `quorum_execute_query` | Runs a read in a read-only session, or a write if the write setting allows it. |
| `quorum_list_saved_queries` | Lists the team's saved queries. |
| `quorum_run_saved_query` | Runs a saved query as a read. |
| `quorum_get_query_history` | Reads query history for shared connections. |
| `quorum_get_audit_log` | Reads the write audit: who, what, risk and rows affected. |
| `quorum_get_data_dictionary` | Reads the team's descriptions of tables and columns. |
| `quorum_get_service_health` | Reads service health from Last9 or Datadog, when set up in Quorum. |
| `quorum_propose_write` | Proposes a write, with a reason, for a person to approve in Quorum. |
| `quorum_get_proposal` | Checks a proposal's outcome. |
| `quorum_list_proposals` | Lists recent proposals and their status. |

## How writes are handled

**Settings › AI clients › 3. Writes from AI clients** has two choices:

| Choice | What happens |
|---|---|
| Ask me in Quorum (default, recommended) | A write waits in Quorum until you approve it. On PROD it also needs PROD typed. |
| Run allowed writes directly | Allowed data changes run at once on LOCAL, DEV, QA, UAT and STAGE. Writes on PROD are refused. |

Even when writes run directly, Quorum still refuses any write on PROD and any critical write; DDL and TRUNCATE on any connection except LOCAL; writes on a connection that is read-only, always reviewed, needs a ticket or has a cool-down; and anything the safety checks block, such as an UPDATE without WHERE. Every write and every refusal is recorded in the write audit, as made by an AI agent.

## Quorum Gate

With **Ask me in Quorum**, the AI client proposes a write with a one-line reason, and Quorum waits for you.

1. Quorum shows the request: the kind, environment, connection, risk and the client's reason, then the SQL, formatted.
2. **What it will change** shows how many rows, the columns it sets and whether any value actually changes, and a few of the matched rows with masked columns masked.
3. If asked, type the confirmation word (for example PROD) and a ticket.
4. Click **Approve and run once**, or type a reason and **Reject**. The client sees your reason.

An approved write runs exactly as proposed, once, through the same checks as the editor. A request expires after 30 minutes. No response means no write.

In clients that support MCP Apps, such as Claude, Claude Desktop, VS Code, Cursor and Goose, the request also appears as a card in the chat. The card shows the status; approving happens in Quorum.

## The quorum command

The same bridge powers a command-line tool that runs queries through the running Quorum app and its rules. Install it from **Settings › AI clients › Command-line tool**.

```
quorum connections
quorum query <connection> "<sql>" [--format table|csv|json] [--limit N]
quorum describe <connection> <table> [--schema S]
quorum saved
quorum run-saved "<name>" [--connection C] [--format …]
quorum history [--connection C] [--limit N]
quorum propose <connection> "<sql>" --reason "<why>" [--wait SECONDS]
quorum request <id> [--wait SECONDS]
```

Only connections shared with AI clients are reachable. Writes go through `propose` and wait for approval in Quorum.

## About this repository

This repository documents Quorum's MCP server and holds its MCP Registry entry (`server.json`). The server itself ships with the Quorum app. Questions: https://www.quorumdb.ai/#contact.

Product and company names are trademarks of their respective owners and are used only to describe what Quorum works with.
