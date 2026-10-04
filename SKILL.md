---
name: mcp
description: MCP management — list / start / disable any MCP server. Use when the user asks "what MCPs are installed / start X mcp / disable X mcp / enable browser mcp". Built on the official @deepseek-ai/dsh-mcp-client bridge + cordis.patch.yml config. (mcp-lazy is EOL — do not use.)
---

# MCP Management — List / Start / Disable Any MCP Server

Built on the official `@deepseek-ai/dsh-mcp-client` bridge + the active profile's `cordis.patch.yml`.
(`dsh-mcp-lazy` is EOL — do not use. If you previously used `dsh-mcp-panel` as a console, leftover history comments may remain in the patch after uninstalling it; comments are harmless.)

> **Where is the bridge (common misconception)**: the bridge ships with DSH — it is NOT in the profile's node_modules and NOT in the global node_modules.
> - Web server (running from source): the bridge is a workspace package in the checkout (`<checkout>/packages/mcp/mcp-client`).
> - Desktop client: the bridge is a built-in plugin (inside the asar).
> So **never install the bridge**; don't conclude "MCP can't start" just because the profile's node_modules lacks it — the bridge is always there.

## Operating Logic (three command types)

1. **Ask about MCPs** ("what MCPs are there" / "what MCPs are installed") → list two inventories: **installed** and **running**.
2. **Start an MCP** ("start X mcp") → append `disabled: false` to the patch.
3. **Disable an MCP** ("disable X mcp") → append `disabled: true` to the patch.

## Key Facts (verified on both desktop and web; mechanism is form-agnostic)

- **Hot reload takes effect immediately — no DSH restart needed for start/disable** (effective within seconds of appending to the patch).
- **Disabling strips the MCP's tools from the current session context immediately** — context savings apply at once, no new session needed.
  (Tool calls already recorded in history don't need deleting; keeping them preserves session continuity.)
- **Append-only, last entry wins**: append override lines at the end of the patch; never edit history lines.
- ⚠️ **Global `~/.dsh/cordis.patch.yml` and the profile patch conflict by application order**: if the same server id is written in both files, the later-applied one wins. **Write it in only one place.**
  (Real incident: a global `disabled: false` overrode the profile's `disabled: true`, so the server couldn't be turned off, and it was misdiagnosed as "needs a restart".)

## Which File to Edit

- The active profile is in env var `DSH_PROFILE` (e.g. `web` / `desktop`); edit `~/.dsh/profiles/<profile>/cordis.patch.yml`.
- The global `~/.dsh/cordis.patch.yml` is usually left alone (only touch it for cross-profile effect).

## Listing Installed / Running

**Installed** (servers defined in config) — find the `dsh-mcp-client` entries in the patch; extract `id` and `config.serverName`.
**Running** — either signal:
- the process is alive (a node process whose command line carries the server's signature, e.g. `chrome-devtools-mcp`)
- or the **last** `disabled` value for that server id in the patch: `false` = running, `true` = disabled.

## Start / Disable (append override lines)

Append to the end of the active profile's `cordis.patch.yml` (`<server-id>` = the `id` of that server in your config):

Start:
```yaml
- id: <server-id>
  name: '@deepseek-ai/dsh-mcp-client'
  disabled: false
```

Disable:
```yaml
- id: <server-id>
  name: '@deepseek-ai/dsh-mcp-client'
  disabled: true
```

(`name` must match the `name` in the insert block exactly, or the loader skips it with a name-mismatch.)

## Adding a New MCP Server (first-time config)

Append an insert block to the patch (chrome-devtools as an example):

```yaml
- insert:
    - id: mcp-<name>
      name: '@deepseek-ai/dsh-mcp-client'
      config:
        serverName: <name>
        transport: stdio
        command: npx            # bare npx is fine on Linux/macOS
        args:
          - '-y'
          - '<mcp-package>@latest'
          # ... remaining args for that server
```

⚠️ **On Windows, don't use bare `npx` as command**: Node spawning bare `npx` → `ENOENT`; `npx.cmd` → `EINVAL`
(Windows can't spawn `.cmd` directly). The most reliable form is running `npx-cli.js` with `node.exe` directly:
```yaml
        command: 'C:\Program Files\nodejs\node.exe'
        args:
          - 'C:\Program Files\nodejs\node_modules\npm\bin\npx-cli.js'
          - '-y'
          - '<mcp-package>@latest'
```

## Browser MCP (chrome-devtools) — the Browser Backend

- `--browserUrl` must point at a Chrome running with a CDP debugging port.
- **Use a dedicated Chrome instance** (its own profile); don't put a debugging port on the user's daily Chrome:
  when the profile is already in use, `--remote-debugging-port` is silently ignored and the port never comes up.
- Verify the backend: `curl http://127.0.0.1:<port>/json/version` returning a Browser version = connected.
- Changing `browserUrl` means editing the insert block (an append override line can't do it — that's a config change).

## Layering

Raw CDP is the low-level channel (`/json` lists pages, `/json/new` opens pages, WebSocket runs JS / takes screenshots); MCP is just the experience layer —
**disabling MCP does not affect raw CDP**; browser capability is fully intact.
