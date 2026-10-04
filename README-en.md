An MCP-management skill for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness): **list / start / disable** any MCP server in natural language — no restart, no GUI.

## What it does

DSH MCP servers are driven by the official `@deepseek-ai/dsh-mcp-client` bridge + `cordis.patch.yml` config. This skill distills that mechanism into an agent-executable workflow covering three command types:

| You say | It does |
|---|---|
| "what MCPs are installed" | lists **installed** and **running** inventories |
| "start X mcp" | appends a `disabled: false` override line to the patch |
| "disable X mcp" | appends a `disabled: true` override line to the patch |

## Key mechanics (verified on both desktop and web clients)

- **Hot reload, instant** — start/disable need no DSH restart; effective within seconds of the append
- **Disable strips context immediately** — the MCP's tools drop out of the current session's tool definitions; token savings apply at once
- **Append-only, last entry wins** — history lines are never edited; you get a free audit trail
- **Global patch vs profile patch conflict** — write each server id in exactly one file (real incident included in the skill)

## Install

Drop `SKILL.md` into your skills directory (DSH auto-discovers it):

```bash
mkdir -p ~/.dsh/skills/mcp
cp SKILL.md ~/.dsh/skills/mcp/SKILL.md
```

No dependencies, no build, no plugin to install (the MCP bridge ships with DSH).

## Compatible versions

DeepSeek Harness 0.2.0-rc.1+ (official mcp-client architecture). `dsh-mcp-lazy` is EOL and not required.

## License

MIT
