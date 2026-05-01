# Obsidian MCP Web Agent

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/github/manifest-json/v/Zbrooklyn/obsidian-mcp-web-agent?label=version)](https://github.com/Zbrooklyn/obsidian-mcp-web-agent/releases)
[![Stars](https://img.shields.io/github/stars/Zbrooklyn/obsidian-mcp-web-agent?style=social)](https://github.com/Zbrooklyn/obsidian-mcp-web-agent/stargazers)

An Obsidian plugin (plus a small AI connector that runs alongside it) that lets Claude or any AI tool control a browser tab inside Obsidian, using websites you're already logged into.

Sign into Gmail once. Then ask Claude to read your inbox, draft replies, archive newsletters. It happens inside Obsidian, in the panel you already use, with your real session. No password handed to the AI, no separate browser opening up, no copy-pasting between windows.

> _Demo screenshot goes here. Capture the 🟢 status bar plus the Web viewer mid-task. Save as `docs/demo.png` and replace this line with `![demo](docs/demo.png)`._

---

## What you'd use it for

- **Manage Gmail without giving Claude your password.** Sign in once, ask it to triage, draft, archive, search.
- **Drive any logged-in dashboard.** Stripe, Vercel, GitHub, Upwork, Shopify Admin. Claude pulls data, fills forms, runs reports.
- **Read an article, write the note.** Claude opens the page in your Web viewer, reads it, writes a summary straight into your vault.
- **Test your own web apps.** Localhost dashboards, internal tools, prototypes. Claude navigates while you watch.

If you've used Playwright MCP, Chrome DevTools MCP, or agent-browser before, this fills a different slot: the browser that already has all your logins in it.

## How it's built (so you know what you're installing)

Two small pieces:

- **An Obsidian plugin** — flips on a "remote control" feature that's already built into Obsidian's browser
- **A small AI connector** (an MCP server) — uses that remote control to drive the browser on Claude's behalf

You install both. They work together. Setup is about 5 minutes total.

---

## Install

### What you'll need first

- Obsidian (desktop, v1.5.0+)
- Node.js v18 or higher (`node -v` to check; install from [nodejs.org](https://nodejs.org) if missing)
- Git (`git --version` to check)
- Claude Code or Claude Desktop

### 1. Install the Obsidian plugin (via BRAT)

In Obsidian:

1. **Settings → Community plugins → Browse**, search for **"BRAT"**, install + enable
2. **Settings → BRAT → "Add Beta plugin"**, paste `Zbrooklyn/obsidian-mcp-web-agent`, click **Add Plugin**
3. **Settings → Community plugins**, toggle **"MCP Web Agent"** ON

A notice will pop up: _"patched N Obsidian shortcut(s). Active on next launch."_ The plugin just added a launch flag to your Obsidian shortcuts. Quit Obsidian and reopen it normally. Status bar (bottom-right) should show 🟢 **Bridge active**.

### 2. Install the AI connector (via npm)

In a terminal:

```bash
npm install -g github:Zbrooklyn/obsidian-mcp-web-agent
```

> _The connector lives in the `/server/` subfolder of the same repo. The npm command pulls and installs from there automatically._

### 3. Tell Claude about it

If you use **Claude Code** (terminal):

```bash
claude mcp add obsidian-mcp-web-agent -- obsidian-mcp-web-agent
```

Then in any active Claude Code session, run `/reload-plugins`. Tools appear instantly.

If you use **Claude Desktop** (GUI), edit your config file:

| OS | Path |
|---|---|
| Mac | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

Add this under `"mcpServers"`:

```json
{
  "mcpServers": {
    "obsidian-mcp-web-agent": {
      "command": "obsidian-mcp-web-agent"
    }
  }
}
```

Save, fully quit and reopen Claude Desktop.

### 4. Try it

Ask Claude:

> Use `obsidian_status` to check the bridge.

You should get back `reachable: true` and your Chrome version. Then try something real:

> Open `https://news.ycombinator.com` in my Obsidian Web viewer and tell me the top 3 headlines.

You'll watch Claude drive the tab live.

Need a click-by-click walkthrough? See [INSTALL.md](INSTALL.md).

---

## FAQ

**Does this work with Claude Desktop?**
Yes. Same connector, different config file (see install step 3 above).

**Is my data safe?**
Everything runs on your computer. The remote-control port only listens on `127.0.0.1` (your machine, not the network). Cookies stay where they always were, in Obsidian's local data folder. Nothing leaves your machine.

**Why not just use Playwright MCP or Chrome DevTools MCP?**
Those spawn a fresh browser. Different cookies, different logins, doesn't reuse anything you've already authenticated with. The whole point of this is reusing your Obsidian session. If you don't need that, the others are great. (And you can run them all at once. They don't conflict.)

**Will this slow down Obsidian?**
The plugin is small and barely runs. The Web viewer itself can feel heavy when loading sites like Gmail, but that's the site's weight, not the plugin's.

**The plugin isn't in the official Obsidian community store. Is it official?**
Not yet. This is a beta release distributed through BRAT. If usage and stability hold up, the official store comes next.

---

<details>
<summary><b>How it actually works (technical)</b></summary>

Obsidian is an Electron app. Its Web viewer is a Chromium webview. When Electron is launched with `--remote-debugging-port=9222`, that Chromium exposes the Chrome DevTools Protocol (CDP) on `localhost:9222`. Anything that speaks CDP — Playwright, Chrome DevTools MCP, raw `chrome-remote-interface`, this project's MCP server — can attach and drive it.

This plugin's job is making the launch flag a one-toggle setting:

1. Enable the plugin
2. It scans your Obsidian shortcuts (Desktop, Start Menu, Taskbar pin) and adds `--remote-debugging-port=9222` to each, saving the original args so the toggle can revert
3. Your next normal launch opens the debug port automatically
4. Toggle the plugin off and the shortcuts revert cleanly

The plugin is about 600 lines of vanilla JavaScript, no build step. The MCP server is about 900 lines, also vanilla, talks raw CDP over WebSocket. Both pieces are MIT-licensed and reviewable.

### MCP tools (25)

The MCP server exposes these to Claude:

- **Diagnostic:** `obsidian_status`, `obsidian_list_tabs`, `obsidian_list_targets`
- **Tab management:** `obsidian_new_tab`, `obsidian_close_tab`, `obsidian_navigate`, `obsidian_back`, `obsidian_forward`, `obsidian_reload`
- **Reading state:** `obsidian_read_state`, `obsidian_read_text`, `obsidian_get_html`, `obsidian_snapshot` (structured interactive elements list), `obsidian_evaluate`
- **Interaction:** `obsidian_click`, `obsidian_type` (works with React-controlled inputs), `obsidian_press_key`, `obsidian_hover`, `obsidian_select_option`, `obsidian_wait_for`, `obsidian_scroll`
- **Visual / debug:** `obsidian_screenshot`, `obsidian_console_messages`, `obsidian_wait_for_network_idle`, `obsidian_cookies` (names + flags only, values redacted)

</details>

<details>
<summary><b>Settings, commands, and the status bar</b></summary>

### Plugin settings (Obsidian → Settings → MCP Web Agent)

- **Bridge enabled** — master toggle. ON patches your shortcuts. OFF reverts them.
- **Debug port** — defaults to 9222. Change only if it conflicts with another tool.
- **Patched shortcuts** — list of `.lnk` files the plugin modified, with original args saved.
- **Restart Obsidian to apply bridge** — quits and relaunches via a patched shortcut.

### Status bar indicator

| State | Meaning |
|---|---|
| 🟢 Bridge active | Debug port is live, AI agents can attach |
| 🟡 Bridge active on next launch | Shortcuts are patched, but THIS Obsidian instance was launched without the flag. Restart. |
| 🔴 Bridge needs setup | Toggle "Bridge enabled" on. |
| ⚪ Bridge disabled | You toggled it off. |

### Commands (Cmd/Ctrl-Shift-P → "MCP Web Agent:")

- Open URL in Web viewer
- List CDP targets
- Check bridge status now
- Restart Obsidian to apply bridge
- Enable / Disable bridge

</details>

<details>
<summary><b>Comparison vs other browser MCPs</b></summary>

| Tool | What it drives | When to use |
|---|---|---|
| **Obsidian MCP Web Agent** (this) | Obsidian's Web viewer (your real session) | Logged-in personal sessions; results land in your vault |
| [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) | Fresh Chromium spawned by the MCP | Anonymous browsing, parallel contexts |
| [@playwright/mcp](https://github.com/microsoft/playwright-mcp) | Playwright-managed browsers | Headless mode, cross-browser testing |
| [browsermcp](https://browsermcp.io) | Your actual desktop Chrome via extension | Driving the browser you already use |

These complement each other. Routing pattern: this for personal logged-in stuff, the others for fresh anonymous work.

</details>

<details>
<summary><b>Platforms and known limits</b></summary>

### Platforms

- **Windows** — fully supported, tested on Win11
- **macOS** — bridge mechanism works, auto-patch step is Windows-specific. Mac users launch Obsidian with `open -a Obsidian.app --args --remote-debugging-port=9222`. PRs welcome.
- **Linux** — same as macOS. Manual launch with `obsidian --remote-debugging-port=9222`.
- **Mobile** — not supported. Obsidian Mobile isn't Electron, no debug port.

### Privacy

- Plugin runs entirely locally. No telemetry.
- Debug port binds to `127.0.0.1` only.
- Patched shortcut paths are stored in `data.json` inside the plugin folder. Gitignored, never leaves your machine.
- The MCP server only connects to `localhost:9222`.

### Known issues

- **Cloudflare error 1002 on some sites** — Cloudflare refuses to load some sites in Obsidian's webview because of how its DNS resolution looks. Use a regular browser MCP for those specific sites.

</details>

<details>
<summary><b>Troubleshooting</b></summary>

**Status bar stays 🔴 after enabling.** The plugin couldn't find shortcuts. Settings → MCP Web Agent → "Patched shortcuts" should list at least one. If empty, run "Patch shortcuts now" from the command palette.

**Status bar stays 🟡 forever.** You launched Obsidian via a shortcut the plugin didn't patch. Either run "Restart Obsidian to apply bridge" from the command palette, or close + reopen Obsidian via a patched shortcut.

**External tool can't see the webview.** Open a Web viewer pane in Obsidian first (Cmd palette → "Web viewer: Open"). The plugin doesn't auto-open one.

**Tools don't appear in Claude after install.** For Claude Code, run `claude mcp list` to confirm `obsidian-mcp-web-agent` is registered, then `/reload-plugins`. For Claude Desktop, check `claude_desktop_config.json` for syntax errors (no trailing commas) and fully quit + reopen.

</details>

<details>
<summary><b>Roadmap</b></summary>

**Planned:**
- Cross-platform shortcut patching (macOS, Linux)
- Submission to the official Obsidian community store
- npm registry publish (currently install via `github:` URL)
- Demo videos

**Maybe (open to discussion):**
- Per-tab visual badge in Obsidian showing which Web viewer Claude is driving
- Webhook subscription so external tools can react to navigations

**Not in scope:**
- Replacing your actual browser
- Mobile support
- Headless mode

</details>

---

## Contributing

Issues and PRs welcome at https://github.com/Zbrooklyn/obsidian-mcp-web-agent/issues.

For PRs: open an issue first to discuss. Match the existing code style (vanilla JS, no build step, idiomatic Obsidian Plugin API). Test on your own vault. Update the README and INSTALL.md if your change affects user-facing behavior.

For bug reports, include: Obsidian version, plugin version, OS, what you ran, what you expected, what happened, and the status bar state at the time.

## Acknowledgments

Built on:
- [Obsidian](https://obsidian.md) and its plugin API
- The [Model Context Protocol](https://modelcontextprotocol.io/) and the [@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- [BRAT](https://github.com/TfTHacker/obsidian42-brat) for beta plugin distribution
- The Chrome DevTools Protocol

Inspired by other "AI inside Obsidian" plugins, each doing a different piece:
- [Claudian](https://github.com/YishenTu/claudian) (Claude Code as sidebar chat)
- [obsidian-mind](https://github.com/breferrari/obsidian-mind) (vault as persistent memory for AI agents)
- [obsidian-agent-client](https://github.com/RAIT-09/obsidian-agent-client) (Agent Client Protocol bridge)

This project's contribution: the browser bridge specifically.

## License

MIT — see [LICENSE](LICENSE).
