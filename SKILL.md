---
name: mcp
description: MCP 管理 — 列出 / 启动 / 禁用任意 MCP 服务器。Use when the user asks "what MCPs are installed / start X mcp / disable X mcp / enable browser mcp". Built on the official @deepseek-ai/dsh-mcp-client bridge + cordis.patch.yml config. (mcp-lazy is EOL — do not use.)
---

# MCP 管理 — 列出 / 启动 / 禁用任意 MCP 服务器

底层：官方 `@deepseek-ai/dsh-mcp-client` 桥 + 活动 profile 的 `cordis.patch.yml` 配置。
（`dsh-mcp-lazy` 已停更，勿用；如曾用 `dsh-mcp-panel` 做控制台，卸载它后 patch 里可能残留历史注释，注释无害。）

> **桥在哪（常见误区）**：桥是 DSH 自带的，**不在** profile 的 node_modules、也不在全局 node_modules。
> - web 服务器（源码运行）：桥是 checkout 里的 workspace 包（`<checkout>/packages/mcp/mcp-client`）。
> - 桌面客户端：桥是内置插件（asar 里）。
> 所以**不用装桥**；别因为"profile node_modules 里没有桥"就误判 MCP 起不来——桥一直有。

## 运行逻辑（三类指令）

1. **询问 MCP**（"有哪些 mcp" / "装了什么 mcp"）→ 列出**已装**和**已启动**两个清单。
2. **启动某 MCP**（"启动 X mcp"）→ 往 patch 追加 `disabled: false`。
3. **禁用某 MCP**（"禁用 X mcp"）→ 往 patch 追加 `disabled: true`。

## 关键事实（实测，桌面端 / web 端通用）

- **热加载即时生效，启动/禁用都无需重启 DSH 进程**（append 到 patch 后几秒内生效）。
- **禁用会立即把该 MCP 的工具从当前会话上下文剥离**——省上下文即时生效，不用开新会话。
  （已发生在历史里的工具调用记录不必删，留着不破坏会话连续性。）
- **append-only、最后一条生效**：往 patch 末尾追加覆盖行，不改历史行。
- ⚠️ **全局 `~/.dsh/cordis.patch.yml` 和 profile patch 会按覆盖顺序打架**：同一 server id
  在两个文件各写一条时，后应用的覆盖先写的。**只在一个地方写**。
  （踩坑实例：全局的 `disabled: false` 覆盖了 profile 的 `disabled: true`，导致关不掉，
  还误判成"要重启"。）

## 改哪个文件

- 活动 profile 看环境变量 `DSH_PROFILE`（如 `web` / `desktop`），改 `~/.dsh/profiles/<profile>/cordis.patch.yml`。
- 全局 `~/.dsh/cordis.patch.yml` 一般不动（除非要跨 profile 生效）。

## 列出已装 / 已启动

**已装**（配置里定义了 server 的）——在 patch 里找 `dsh-mcp-client` 条目，提取 `id` 和 `config.serverName`。
**已启动**——两个信号取其一：
- 进程在跑（node 跑的，命令行带 server 特征，如 `chrome-devtools-mcp`）
- 或看每个 server id 在 patch 里**最后一条** `disabled` 值：`false`=启动，`true`=禁用。

## 启动 / 禁用（追加覆盖行）

往活动 profile 的 `cordis.patch.yml` **末尾**追加（`<server-id>` 用配置里那个 server 的 `id`）：

启动：
```yaml
- id: <server-id>
  name: '@deepseek-ai/dsh-mcp-client'
  disabled: false
```

禁用：
```yaml
- id: <server-id>
  name: '@deepseek-ai/dsh-mcp-client'
  disabled: true
```

（`name` 必须和 insert 块里的 `name` 一致，否则 loader 会 name-mismatch 跳过。）

## 新增一个 MCP 服务器（首次配置）

往 patch 追加 insert 块（以 chrome-devtools 为例）：

```yaml
- insert:
    - id: mcp-<name>
      name: '@deepseek-ai/dsh-mcp-client'
      config:
        serverName: <name>
        transport: stdio
        command: npx            # Linux/macOS 用裸 npx 即可
        args:
          - '-y'
          - '<mcp-package>@latest'
          # ... 该 server 的其余参数
```

⚠️ **Windows 上 command 别用裸 `npx`**：Node spawn 裸 `npx` → `ENOENT`；用 `npx.cmd` → `EINVAL`
（Windows 不能直接 spawn `.cmd`）。最稳是 `node.exe` 直接跑 `npx-cli.js`：
```yaml
        command: 'C:\Program Files\nodejs\node.exe'
        args:
          - 'C:\Program Files\nodejs\node_modules\npm\bin\npx-cli.js'
          - '-y'
          - '<mcp-package>@latest'
```

## 浏览器 MCP（chrome-devtools）的浏览器后端

- `--browserUrl` 指向一个开着 CDP 调试端口的 Chrome。
- **建议独立 Chrome 实例**（独立 profile），别把用户日常 Chrome 挂调试端口：
  profile 被占用时 `--remote-debugging-port` 会被忽略，端口起不来。
- 验证后端通：`curl http://127.0.0.1:<port>/json/version` 返回 Browser 版本 = 通。
- 改 `browserUrl` 要动 insert 块（不是 append 覆盖行能解决的，属配置变更）。

## 分层认知

raw CDP 是底层通道（`/json` 列页、`/json/new` 开页、WebSocket 执行 JS/截图），MCP 只是体验层——
**禁用 MCP 不影响 raw CDP**，浏览器能力一点不少。
