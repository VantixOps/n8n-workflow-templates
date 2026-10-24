# n8n workflow templates by VantixOps

Ready-to-import n8n workflows we built, tested and use as building blocks. Free under the MIT license.

| Template | What it does | Nodes |
|---|---|---|
| [MCP server inspection](mcp-server-inspection/) | Connects to any Streamable HTTP MCP server, tests every tool against its own schema and the MCP spec, and grades it A–F with a concrete fix per failure. Built-in demo server included. | 30 |
| [Deduplicated error alerts](error-alerts/) | Error workflow that sends one clear email per problem, holds back repeats during a cooldown and still alerts if the alert itself fails. | 15 |
| [Uptime monitor](uptime-monitor/) | Checks your URLs on a schedule, confirms an outage before alerting, reminds while it lasts and sends an all-clear when it is back. | 11 |

## How to use

1. Download the `workflow.json` of a template.
2. In n8n: *Workflows > Import from file*.
3. Follow the template's README – each one lists the credentials and settings it needs.

None of the files contain credentials, API keys or personal data. Example addresses use `example.com`.

## Also from us

- [n8n-nodes-etermin](https://github.com/VantixOps/n8n-nodes-etermin) – community node for the eTermin booking API.

## About

Built by [VantixOps](https://vantix-ops.de) – office automation for small businesses from Bremen, Germany. Questions or ideas: open an issue or write to kontakt@vantix-ops.de.

Not affiliated with or endorsed by n8n GmbH.
