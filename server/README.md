# obsidian-mcp-web-agent (server)

The MCP server half of the [Obsidian MCP Web Agent](../README.md) project. Connects to the Obsidian plugin via Chrome DevTools Protocol on `localhost:9222` and exposes 25 tools to Claude (or any MCP client).

## Install

You need the [Obsidian plugin](../README.md) installed and enabled first. Then:

```bash
npm install -g github:Zbrooklyn/obsidian-mcp-web-agent
```

> _Note: this server lives inside the parent repo's `/server/` subfolder. The `npm install -g github:...` command pulls the whole repo and installs from there. The plugin part isn't installed via npm — that goes through BRAT inside Obsidian._

## Wire into Claude

For Claude Code:

```bash
claude mcp add obsidian-mcp-web-agent -- obsidian-mcp-web-agent
```

For Claude Desktop, edit `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "obsidian-mcp-web-agent": {
      "command": "obsidian-mcp-web-agent"
    }
  }
}
```

Full setup instructions: see the [main README](../README.md) and [INSTALL.md](../INSTALL.md).

## Tools (25)

See the main README's "How it works" collapsible for the full tool list, or run `obsidian-mcp-web-agent` and inspect the tools/list MCP response.

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `OBSIDIAN_BRIDGE_HOST` | `127.0.0.1` | CDP host |
| `OBSIDIAN_BRIDGE_PORT` | `9222` | CDP port (must match the plugin) |
| `OBSIDIAN_BRIDGE_LOAD_CAP_MS` | `8000` | Cap on waiting for `loadEventFired` |
| `OBSIDIAN_BRIDGE_NAV_TIMEOUT_MS` | `15000` | Cap on `Page.navigate` itself |

## License

MIT — see LICENSE.
