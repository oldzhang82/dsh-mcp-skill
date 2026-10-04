# dsh-mcp-skill

MCP 管理 skill for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) — list / start / disable any MCP server in natural language. No restart, no GUI.

用自然语言**列出 / 启动 / 禁用**任意 MCP 服务器，无需重启、无需 GUI 操作。

📖 **Languages / 语言**: [中文 README-cn.md](README-cn.md) · [English README-en.md](README-en.md)

## Quick Install / 快速安装

```bash
mkdir -p ~/.dsh/skills/mcp
cp SKILL.md ~/.dsh/skills/mcp/SKILL.md
```

No dependencies, no build, no plugin to install — the MCP bridge ships with DSH.
无依赖、无构建、无需装插件（MCP 桥是 DSH 自带的）。

## Highlights / 亮点

- **Hot reload** — start/disable take effect in seconds, no DSH restart · 热加载即时生效，启停不用重启
- **Disable strips context** — tools drop out of the session immediately · 禁用立即从会话上下文剥离，省 token
- **Append-only** — last entry wins, free audit trail · 只追加不改历史，天然留审计轨迹
- Works on both **desktop and web** clients · 桌面端 / web 端通用

Compatible with DeepSeek Harness 0.2.0-rc.1+ · 适用 0.2.0-rc.1+

## License / 许可证

[MIT](LICENSE)
