# Claude Browser Bridge

> An Obsidian plugin that lets AI agents (Claude, Playwright, Chrome DevTools MCP, anything that speaks the Chrome DevTools Protocol) drive Obsidian's built-in Web viewer.

You log in to a website once in Obsidian's Web viewer (Gmail, GitHub, Upwork — whatever) and an external AI tool can then drive that tab using your already-authenticated session. No headless browser, no separate authentication, no token storage. The login is yours; the bridge is just an attach point.

Pairs with the [`obsidian-bridge-mcp`](https://github.com/Zbrooklyn/obsidian-bridge-mcp) MCP server, which exposes the bridge to Claude Code (and any other MCP client) as 25 tools — navigate, click, type, screenshot, snapshot, scroll, etc.

---

## How it works

Obsidian is built on Electron. When Electron launches with `--remote-debugging-port=9222`, its embedded Chromium exposes the Chrome DevTools Protocol on `localhost:9222`. Any tool that speaks CDP can attach and drive the Web viewer.

This plugin's job is to make that one launch flag a one-toggle setting:

1. You enable the plugin
2. It scans your Obsidian shortcuts (Desktop, Start Menu, Taskbar pin) and adds `--remote-debugging-port=9222` to each
3. Your next normal Obsidian launch opens the debug port automatically — no special launcher
4. Toggle the plugin off → shortcuts revert cleanly

The plugin itself is small (~600 lines of vanilla JS, no build step). Most of what it does is shortcut management; the actual driving happens via CDP from outside.

## Install (BRAT — recommended)

[BRAT](https://github.com/TfTHacker/obsidian42-brat) is the standard way to install Obsidian plugins that haven't been added to the official community store yet.

1. **Install BRAT** — Settings → Community plugins → Browse → search "BRAT" → install + enable
2. **Open BRAT settings** — Settings → BRAT → "Add Beta plugin"
3. **Enter:** `Zbrooklyn/obsidian-claude-bridge`
4. **Choose latest release** → Add Plugin
5. **Enable** in Settings → Community plugins → Claude Browser Bridge → toggle on
6. **The plugin auto-patches your shortcuts.** A notice will fire telling you it patched N shortcuts. The bridge becomes active the next time you launch Obsidian — no special action needed, just close and reopen normally when you're ready.

## Install (manual)

If you don't want BRAT:

1. Download `main.js`, `manifest.json` from the [latest release](https://github.com/Zbrooklyn/obsidian-claude-bridge/releases)
2. Put both into `<your-vault>/.obsidian/plugins/claude-browser-bridge/`
3. Settings → Community plugins → reload → enable Claude Browser Bridge

## Settings

Open Settings → Claude Browser Bridge:

- **Bridge enabled** (master toggle) — flip ON to patch shortcuts; flip OFF to revert
- **Debug port** — default `9222`. Change only if it conflicts with another tool
- **Patched shortcuts** — list of which `.lnk` files the plugin modified, with original args saved so the toggle can restore
- **Restart Obsidian to apply bridge now** — optional. Quits and relaunches via a patched shortcut so the bridge activates immediately. Same effect as closing and reopening Obsidian normally — just faster

## Status bar indicator

| State | Meaning |
|---|---|
| 🟢 Bridge active | Debug port is live, AI agents can attach |
| 🟡 Bridge active on next launch | Shortcuts patched, but THIS Obsidian instance was launched without the flag — restart to activate |
| 🔴 Bridge needs setup | Toggle "Bridge enabled" on |
| ⚪ Bridge disabled | You toggled it off |

Click the status bar item to see the list of CDP targets currently visible to external tools.

## Commands

`Ctrl/Cmd+Shift+P` → "Claude Browser Bridge:":

- **Open URL in Web viewer** — prompts for URL, opens in a new Web viewer pane
- **List CDP targets** — shows all attachable targets
- **Check bridge status now** — forces a status poll
- **Restart Obsidian to apply bridge** — quits + relaunches with the flag
- **Enable / Disable bridge** — same as the settings toggle

## Platforms

- ✅ Windows (tested on Win11)
- ⚠️ macOS — should work, shortcut-patching code path is Windows-specific. The bridge mechanism (CDP attach) is platform-agnostic; only the auto-patch step is Windows-only for now. macOS users can manually launch Obsidian with `open -a Obsidian.app --args --remote-debugging-port=9222`. PRs welcome.
- ⚠️ Linux — same as macOS. Manual launch with `obsidian --remote-debugging-port=9222`.
- ❌ Mobile — Obsidian Mobile doesn't run on Electron, no debug port available.

## Privacy & data

- The plugin runs entirely locally. No telemetry, no remote calls beyond polling `localhost:9222` for status.
- The debug port binds only to `127.0.0.1` (localhost) — no LAN exposure unless you explicitly add `--remote-debugging-address=0.0.0.0` (don't do this).
- Patched shortcut paths are stored in `data.json` inside this plugin's folder. That's gitignored and never leaves your machine.

## Troubleshooting

**Status bar stays 🔴 after enabling**
The plugin couldn't find any Obsidian shortcuts to patch. Check Settings → Claude Browser Bridge → "Patched shortcuts" — should list at least one. If empty, your Obsidian shortcuts might be in non-standard locations. Run "Patch shortcuts now" from the command palette to force a re-scan.

**Status bar stays 🟡 forever**
You launched Obsidian via a shortcut the plugin didn't patch (e.g. a custom shortcut elsewhere on disk, or a .desktop file on Linux). Either run "Restart Obsidian to apply bridge" from the command palette, or close + reopen Obsidian via a patched shortcut.

**Plugin slow to load (>30 sec)**
Old issue with v0.2 of the plugin (pre-filtered .lnk scan was slow). Fixed in v0.2.1+. If you're seeing this, you're on an old version — update via BRAT.

**External tool can't see the webview**
Open a Web viewer pane in Obsidian first (Cmd palette → "Web viewer: Open"). The plugin doesn't auto-open one; the bridge only enumerates webviews that exist.

## License

MIT — see LICENSE.
