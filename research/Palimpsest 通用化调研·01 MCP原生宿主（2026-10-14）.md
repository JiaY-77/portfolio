# MCP 原生宿主：记忆接入能力调研

调研对象：Claude Desktop、Claude Code、Cursor、Windsurf、Cline、Continue、GitHub Copilot (VS Code)
调研日期：2026-10-14
被接入方：Palimpsest（本地优先 AI 记忆底座，MCP stdio + streamable-http / REST / CLI）

约定：每条结论后附来源 URL。标「官方」= 官方文档明确写的；标「推断」= 基于事实的推论；标「未能确认」= 查不到可靠来源。所有配置字段名均照抄官方原文，未做改动或猜测。

---

## 1. 对比表

| 宿主 | 能否接 MCP | 官方记忆能力 | 上架入口 | 接入路径 | 工作量 |
|---|---|---|---|---|---|
| **Claude Desktop** | 能。`claude_desktop_config.json`，顶层 `mcpServers` 对象，字段 `command` / `args` / `env`。Windows 路径 `%APPDATA%\Claude\claude_desktop_config.json`，macOS `~/Library/Application Support/Claude/claude_desktop_config.json`。https://modelcontextprotocol.io/docs/develop/connect-local-servers | 无本地记忆文件层。Claude.ai 侧有服务端「记忆/项目」（账号级），与本地 MCP 服务器不是同一层，官方未描述二者冲突（未能确认是否存在隐式冲突） | 无独立服务器目录。社区侧走官方 MCP Registry：https://modelcontextprotocol.io/registry/about | 只配 MCP（stdio 或远程 URL）；Palimpsest 已提供 stdio + streamable-http，直接填 JSON 即可 | 极低（改一行 JSON + 重启） |
| **Claude Code** | 能。项目级 `.mcp.json`（顶层 `mcpServers`，共享给团队，支持 `${VAR}` 展开）；用户级走 `claude mcp add` / `~/.claude.json`；HTTP 服务器需 `"type": "http"`。https://code.claude.com/docs/en/settings-reference | **有**，文件式：`CLAUDE.md`（持久上下文，每会话加载，建议 <200 行）+ auto memory（`~/.claude/projects/<project>/memory/`，`MEMORY.md` 作索引，启动加载前 200 行 / 25KB）。https://code.claude.com/docs/en/memory | 无 MCP 目录；有插件 marketplace（插件可打包 MCP server）。https://code.claude.com/docs/en/plugins/overview | 只配 MCP 即可让 Claude 调用 Palimpsest 工具；若要「每会话自动注入记忆」需再加 hook/plugin（MCP 只给工具，不自动注入） | 低（配 MCP）；中（要自动注入则加 hook 或 plugin） |
| **Cursor** | 能。项目 `.cursor/mcp.json`，全局 `~/.cursor/mcp.json`；顶层 `mcpServers`，字段 `command`/`args`/`env`/`url`/`headers`；支持 `${env:NAME}`、`${workspaceFolder}` 插值。https://cursor.com/docs/mcp | **有**：Memories（从对话自动提取、按 Git 仓库作用域、存于 Cursor 自家基础设施，Privacy Mode 下不可用）+ Rules（`.cursor/rules` 的 `.mdc`）。注意：截至 2026-09 官方文档索引里没有独立 Memories 页面，行为描述散在 changelog/release notes（转载观察来源：https://www.past.dev/blog/cursor-memory ，官方当前页面**未能确认**） | **有，且明确**：Cursor Marketplace 提交入口 https://cursor.com/marketplace/publish ；插件需开源、人工审核。社区 MCP 另见 cursor.directory | 只配 MCP。Memories 与第三方 MCP 记忆服务器不冲突（Memories 是模型侧摘要，MCP 是工具调用，作用面不同） | 低（配 MCP）；用插件分发则中（做 `plugin.json` 清单 + 提审） |
| **Windsurf** | 能。`~/.codeium/windsurf/mcp_config.json`（Windows `%USERPROFILE%\.codeium\windsurf\mcp_config.json`），顶层 `mcpServers`，支持 stdio / Streamable HTTP / SSE；远程用 `serverUrl` 或 `url`。https://docs.devin.ai/windsurf/plugins/cascade/mcp | **有**：Cascade Memories，工作区作用域短笔记，存放于 `~/.codeium/windsurf/memories/`（含 `global_rules.md`）。https://github.com/omar-haris/smart-coding-mcp/blob/main/docs/ide-setup/windsurf.md （第三方来源，**推断**官方口径一致） | 有内置 MCP Marketplace（一键安装）。对外开放的提交入口**未能确认**——官方文档只写「Add Server」，未提供提交表单 URL | 只配 MCP | 低（配 MCP）。注意 Windsurf 全局工具上限约 100 个，工具多的 server 需按需加载 |
| **Cline** | 能。IDE 扩展读 `cline_mcp_settings.json`（面板 → MCP Servers → Configure → Configure MCP Servers 打开）；CLI 读 `~/.cline/data/settings/cline_mcp_settings.json`（可用 `CLINE_MCP_SETTINGS_PATH` 覆盖）。远程服务器字段 `type: "streamableHttp"` / `url` / `headers`。https://docs.cline.bot/mcp/mcp-overview | **无内置记忆层**。Memory Bank 是社区自定义指令方法论（`.clinerules/memory-bank.md` + `memory-bank/` 目录 markdown），靠 prompt 让模型读写。https://docs.cline.bot/best-practices/memory-bank | 有内置 MCP Marketplace。对外开放的提交入口**未能确认** | 只配 MCP。因为无官方记忆层，Palimpsest 不会与任何官方机制冲突，纯粹补空白 | 低（配 MCP） |
| **Continue** | 能。工作区 `.continue/mcpServers/<name>.yaml`（需 `name`/`version`/`schema` 元数据 + `mcpServers` 列表），或全局 `~/.continue/config.yaml` 的顶层 `mcpServers`；字段 `type`（`stdio`/`sse`/`streamable-http`）/`command`/`args`/`env`。**MCP 仅在 agent 模式生效**。https://docs.continue.dev/customize/deep-dives/mcp | **无官方记忆层**（社区有 Memory Bank 类 feature request，未实现：https://github.com/continuedev/continue/issues/4615 ）。持久化靠 rules / prompts | **有**：Continue Hub 是官方注册中心，可发布 assistants / blocks（含 MCP servers）：https://hub.continue.dev | 只配 MCP；可直接把 Claude/Cursor 的 JSON 配置丢进 `.continue/mcpServers/` 自动加载 | 低（配 MCP，YAML）；上架 Hub 中低 |
| **GitHub Copilot (VS Code)** | 能。工作区 VS Code 格式 `.vscode/mcp.json`（顶层 `servers`）；工作区可移植格式 `.mcp.json`（顶层 `mcpServers`，跨工具通用）；用户可移植格式 `~/.copilot/mcp-config.json`。字段 `type`（`stdio`/`http`/`sse`）/`command`/`args`/`url`/`headers`/`oauth`。https://code.visualstudio.com/docs/agents/reference/mcp-configuration ；配置方式见 https://docs.github.com/copilot/customizing-copilot/using-model-context-protocol/extending-copilot-chat-with-mcp | **无独立记忆工具**。持久指令走 custom instructions（`.github/copilot-instructions.md`、`.github/instructions/`）与 settings 指令字段，属静态指令而非可检索记忆 | **有**：GitHub MCP Registry（公开预览），VS Code 内 `@mcp` 搜索安装；官方入口 https://github.com/mcp | 只配 MCP。官方文档示例里就用了 `@modelcontextprotocol/server-memory`，说明记忆型 MCP 是被承认的用法 | 低（配 MCP）；走 Registry 上架中 |

---

## 2. 关键差异小结（按「官方记忆 vs 第三方记忆」分层）

「官方记忆能力」要拆成三类，否则容易误判冲突：

- **文件式 / 可干预**：Claude Code（`CLAUDE.md` + auto memory 目录）、Windsurf（memories 目录）、Cline（Memory Bank，社区方法论）。这类是**本地文件**，理论上可与 Palimpsest 并存甚至互通（推断：可让 Palimpsest 把摘要写进这些文件，但官方均未提供双向接口）。
- **服务端 / 不可干预**：Cursor Memories（存在 Cursor 自家基础设施上，Privacy Mode 下禁用）。这类**无法**与本地优先的 Palimpsest 打通，属平行系统（推断）。
- **无记忆层**：Continue、GitHub Copilot、Claude Desktop。这三个是 Palimpsest 的纯空白市场。

冲突判断：**均未发现官方记忆能力会阻止第三方 MCP 记忆服务器接入**。Cursor 的 Memories 与 MCP 是两套不同机制，官方文档未描述互斥（是否存在隐式冲突**未能确认**）。Cline / Continue / Copilot 无记忆层，不存在冲突。

---

## 3. 上架入口一览

| 入口 | URL | 说明 |
|---|---|---|
| 官方 MCP Registry | https://modelcontextprotocol.io/registry/about | 官方集中元数据库，preview 阶段。要求公开可安装（npm/Docker/公开远程 URL），不支持私有服务器。命名走反向 DNS（`io.github.user/server`），需 GitHub/DNS 验证 |
| Cursor Marketplace | https://cursor.com/marketplace/publish | 官方插件市场，需开源 + 人工审核（官方明确写「Every plugin is manually reviewed」） |
| Continue Hub | https://hub.continue.dev | 官方注册中心，可发布 MCP server blocks |
| GitHub MCP Registry | https://github.com/mcp | 官方策展目录，VS Code 内 `@mcp` 可搜可装（public preview） |
| Windsurf Marketplace | 未能确认 | 有内置一键安装市场，但未找到对外开放的提交入口 URL |
| Cline Marketplace | 未能确认 | 有内置市场，未找到对外开放的提交入口 URL |
| Claude Desktop / Claude Code | 无独立服务器目录 | Claude Code 有插件 marketplace 概念（https://code.claude.com/docs/en/plugins/overview），但不等于 MCP 服务器目录 |

---

## 4. 结论：哪 2-3 个最适合先做

**先做 1：GitHub Copilot (VS Code) 与 Claude Code —— 零官方记忆层或纯文件式记忆、接入成本最低、用户基数最大。**

- Copilot (VS Code)：无官方记忆层（纯空白市场），且官方文档**自己就演示了 `@modelcontextprotocol/server-memory`** 的配法（https://docs.github.com/copilot/customizing-copilot/using-model-context-protocol/extending-copilot-chat-with-mcp ），说明「记忆 MCP」是被官方承认的用法，认知负担为零；`.mcp.json` 是可移植格式，一份配置同时喂 Copilot CLI 与 VS Code。接入 = 教用户填一段 JSON，**无需写任何适配器**。
- Claude Code：官方记忆层是**文件夹式**的（`CLAUDE.md` + auto memory），与 Palimpsest 定位互补不冲突（前者是「每会话必加载的约定」，后者是「按需语义检索的库」）。接入只配 MCP 即低工作量；官方还留了 hook / plugin 两个扩展点，后续要做「自动注入」有正规路径（https://code.claude.com/docs/en/features-overview ）。

**先做 2（并列）：Cursor —— 唯一有明确、对外开放、可提审插件市场的宿主，获客效率最高。**

- 官方给了明确提交入口 https://cursor.com/marketplace/publish ，且插件机制允许**把 MCP server 打包进插件**一起分发（https://cursor.com/docs/plugins ），意味着 Palimpsest 可以做成「装一个插件，MCP 配置自动就位」，比教用户手改 JSON 转化率高得多。
- 门槛是插件需开源 + 人工审核，Palimpsest 本就是 MIT，天然满足（推断）。

**暂缓**：Claude Desktop（无官方服务器目录，只能靠 MCP Registry 与社区分发）、Windsurf / Cline（上架入口未能确认，且 Windsurf 有工具数量上限需适配）、Continue（无官方记忆层是利好，但用户盘子最小，优先级靠后，且 MCP 只在 agent 模式生效）。

**一句话**：优先「无官方记忆层 + 配 JSON 就能用」的 Copilot (VS Code) 和 Claude Code 打广度，优先「有可提审市场」的 Cursor 打深度转化；三者都不需要为宿主写专属适配器，纯 MCP 即可落地。
