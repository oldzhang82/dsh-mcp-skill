# dsh-mcp-skill

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的 MCP 管理 skill：用自然语言**列出 / 启动 / 禁用**任意 MCP 服务器，无需重启、无需 GUI 操作。

## 它解决什么

DSH 的 MCP 服务器由官方 `@deepseek-ai/dsh-mcp-client` 桥 + `cordis.patch.yml` 配置驱动。这个 skill 把这套机制沉淀成 agent 可直接执行的流程，覆盖三类指令：

| 你说 | 它做 |
|---|---|
| "有哪些 mcp / 装了什么 mcp" | 列出**已装**和**已启动**两个清单 |
| "启动 X mcp" | 往 patch 追加 `disabled: false` 覆盖行 |
| "禁用 X mcp" | 往 patch 追加 `disabled: true` 覆盖行 |

## 核心机制（实测，桌面端 / web 端通用）

- **热加载即时生效**——启停都不用重启 DSH 进程，append 后几秒内生效
- **禁用立即剥离上下文**——该 MCP 的工具从当前会话工具定义中移除，省 token 即时生效
- **append-only、最后一条生效**——不改历史行，天然留审计轨迹
- **全局 patch 与 profile patch 会打架**——同一 server id 只在一个文件里写（skill 里有踩坑实例）

## 安装

把 `SKILL.md` 放进你的 skills 目录即可（DSH 自动发现）：

```bash
mkdir -p ~/.dsh/skills/mcp
cp SKILL.md ~/.dsh/skills/mcp/SKILL.md
```

无依赖、无构建、不需要装任何插件（MCP 桥是 DSH 自带的）。

## 适用版本

DeepSeek Harness 0.2.0-rc.1+（官方 mcp-client 架构）。`dsh-mcp-lazy` 已停更，本 skill 不依赖它。
