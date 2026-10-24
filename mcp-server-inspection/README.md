# Grade MCP server tools with live contract tests (n8n)

An n8n workflow that inspects an MCP server the way a real client would and grades every tool A–F – before an AI agent finds out the hard way that a tool is broken.

Works for MCP servers built with n8n's **MCP Server Trigger** and for any other **Streamable HTTP** MCP server.

## What it does

1. Connects like a real MCP client (MCP 2026-07-28, with fallback to the 2025 `initialize` handshake).
2. Builds fair test cases from each tool's own JSON Schema: one valid call, only the invalid calls the schema itself forbids, a few robustness probes and one unknown-method probe.
3. Calls the tools live, times them and checks every answer against JSON-RPC 2.0 and the MCP specification, including `structuredContent` against a declared `outputSchema`.
4. Shows a report page: a grade per tool and for the server, what was sent, what came back and a concrete fix for every failure. What it cannot decide is marked as *not graded* instead of guessed.

A built-in demo server (one careful tool, one deliberately broken tool) shows a full report without any setup.

## Safe by default

- Only read-only tools are called unless you choose otherwise; tools that may be destructive are never called.
- Only the demo and hosts listed in `allowedHosts` are contacted; redirects are not followed.
- Secrets in answers are masked in the report and flagged; no execution data is stored.

## Setup

1. Import `workflow.json` (*Workflows > Import from file*), publish it and open the form URL of **Start inspection**. Leave the URL empty to inspect the demo.
2. To inspect your own server, add its host to `allowedHosts` in **Inspection settings** and paste its Streamable HTTP endpoint.
3. Protected server: add a Header Auth credential to the five HTTP Request nodes named in the overview note.

## Requirements

None for the demo. For your own inspections: an MCP server reachable from your n8n instance. The workflow contains no credentials.

## Customize

Latency budget and tool mode in the form; `maxTools` and `allowedHosts` in **Inspection settings**.
