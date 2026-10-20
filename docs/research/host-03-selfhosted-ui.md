# 自托管对话 UI 的外部记忆接入与插件市场

调研对象：Open WebUI、LibreChat、AnythingLLM、LobeChat、Cherry Studio
调研方法：官方文档 + 官方仓库一手核实；结论后附 URL。标注规则：「官方」= 官方文档/仓库明确写；「推断」= 我基于事实的推论；「未能确认」= 查不到官方明确说法。

---

## 1. 对比表

| 产品 | 插件/记忆机制 | MCP 支持 | 自带记忆 | 上架入口 | 接入路径 | 工作量 |
|---|---|---|---|---|---|---|
| **Open WebUI** | Workspace Tools（进程内 Python，最强）/ Functions / Pipelines；另有 MCP 外部工具服务器 | ✅ 原生 MCP，v0.6.31+，**仅 Streamable HTTP**（stdio/SSE 需经 mcpo 代理转 OpenAPI） | ✅ 内置 Memory（Settings > Personalization > Memory），原生工具 `add_memory`/`search_memories`/`replace_memory_content`/`delete_memory`；存本地 DB | https://openwebui.com/ （社区站，注册账号即可发帖分享；**无审核**，官方明说「listing 未经审计」） | 加一个 Palimpsest MCP server 到 Admin > Integrations（类型选 MCP Streamable HTTP）即可被模型调用；或写 Workspace Tool 直连 REST | 低（配置 + 一次连通性验证即可） |
| **LibreChat** | MCP servers（一等公民）+ Agents + Actions（OpenAPI）；Skills（Markdown） | ✅ 原生 MCP，配置在 `librechat.yaml` 的 `mcpServers`，也可 UI 添加；支持 stdio / SSE / **推荐 Streamable HTTP**；OAuth 2.1、用户级凭据 | ✅ User Memory，**key/value 存储**（非语义记忆），`librechat.yaml` 的 `memory` 对象配置，默认关闭；有 `set_memory`/`delete_memory` 工具 | 无官方插件市场；靠 **Smithery**（smithery.ai）发现安装 MCP server（第三方目录，非其自有上架体系） | 在 `librechat.yaml` 注册 Palimpsest MCP server → Agent Builder 挂载工具；配置简单 | 低—中（写 yaml + 重启；若要多用户隔离需评估其工具身份传递限制） |
| **AnythingLLM** | Agent Skills（Node.js 自定义技能）+ MCP servers；社区 Hub 分发 skills/prompts/commands | ✅ 支持 MCP **stdio / SSE / Streamable** 三种传输，配置文件 `plugins/anythingllm_mcp_servers.json` + UI 管理 | ✅ 有 agent 长期记忆（`agent_memory.txt`＋RAG Search 可写入 agent memory）。注：官方文档对 agent memory 的机制说明不完整，社区有 issue 质疑其黑盒性 | https://hub.anythingllm.com （Community Hub，**beta**，支持 Agent Skills / System Prompts / Slash Commands；公开或私密分享） | 在 `anythingllm_mcp_servers.json` 加一个 Palimpsest MCP server（stdio 或 streamable 均可），或写 custom agent skill 走 REST | 低（配置文件 + 重启） |
| **LobeChat / LobeHub** | 插件系统（函数调用）+ MCP 插件 + 内置 Agent Memory 插件 | ✅ 原生 MCP，插件商店可「Add MCP Plugin」；本地/远程均可（stdio 命令、SSE、Streamable HTTP） | ✅ Agent Memory 内置插件（结构化、可编辑、有 confidence/category/tag、记忆 Agent 管理）；另有 Labs 的 Agent Self-Learning | https://lobehub.com/mcp → 用官方 CLI `@lobehub/market-cli`（`plugin publish`）发布；需 Node 22+，**需浏览器登录两次**；另有旧插件仓库走 PR | 插件商店 → Add MCP Plugin → 填 Palimpsest MCP URL → 安装；上架走 market-cli | 低（接入）；上架中等（CLI + 双登录 + manifest 维护） |
| **Cherry Studio** | MCP 服务器（内置市场 + 服务商集成）+ 技能 + Agent | ✅ 原生 MCP，桌面客户端：stdio / SSE / **Streamable HTTP**；`设置 → MCP 服务器`；有内置 MCP 市场（`发现 → 内置服务器`），并与 ModelScope 魔搭官方打通同步 | ✅ Agent 记忆（跟随 Agent，存长期偏好/项目事实/技术决策/经验；非知识库），中文官方文档明确说明 | 无面向第三方开发者的开放插件上架机制（内置市场/服务商集成是官方维护，未能确认开放提交入口）；社区分享多在 GitHub Discussion | `设置 → MCP 服务器 → 添加`，类型选 Streamable HTTP，填 Palimpsest MCP URL | 低（配置即可）；上架—未能确认 |

---

## 2. 逐产品详解

### 2.1 Open WebUI

**1) 挂外部记忆的机制**
- 三条路：**Workspace Tools**（进程内 Python 脚本，能力最强、最无限制）、**Functions/Pipelines**、**外部工具服务器（MCP / MCPO / OpenAPI）**。来源：https://docs.openwebui.com/features/extensibility/plugin/tools
- 官方明确定位：外部工具服务器是「分离进程/机器，通过 HTTP 调用」；Workspace Tools 跑在进程内、能访问整个 `open_webui` 代码库和请求上下文，是最强的扩展方式。

**2) MCP 支持**（官方）
- ✅ 原生支持 MCP，**从 v0.6.31 开始**；**仅支持 Streamable HTTP**。
- stdio / SSE 的 MCP server 需要经 **mcpo**（https://github.com/open-webui/mcpo）代理转成 OpenAPI 端点。
- **MCP server 只能由管理员添加**（Settings > Admin > Integrations），普通用户不能加 MCP（只能加 OpenAPI）；设计理由：MCP 有状态、可执行命令、在信任边界内，风险高。
- 官方明确说**多数部署仍推荐 OpenAPI** 作为首选集成路径。
- 来源：https://docs.openwebui.com/features/extensibility/mcp

**3) 自带记忆**（官方）
- 内置 Memory：存储用户事实/偏好，两种使用方式——① 手动管理（Settings > Personalization > Memory）② 原生工具调用（Agentic 模式）下模型自主管理。
- 官方列出的记忆工具名：`add_memory`、`update_memory`、`replace_memory_content`、`delete_memory`、`search_memories`、`list_memories`、`list_memory_paths`、`read_memory_path`。
- 记忆带 **type**（`user` / `context`）和可选 **path**（如 `work/projects`）。
- 管理开关：`ENABLE_MEMORIES`、按角色/组权限 `USER_PERMISSIONS_FEATURES_MEMORIES`、`ENABLE_MEMORY_SYSTEM_CONTEXT`、`ENABLE_MEMORY_BACKGROUND_REVIEW`、`MEMORIES_REVIEW_INTERVAL_TURNS`、`MEMORIES_CONTEXT_CHAR_LIMIT`、`MEMORIES_USER_CHAR_LIMIT`。
- 数据**存本地 Open WebUI 数据库**、按用户账号隔离、可清空。
- 来源：https://docs.openwebui.com/features/chat-conversations/memory

**4) 上架机制**
- 社区站 https://openwebui.com/ 承载社区 Tools/Functions，一键导入。官方**明确声明：listing 是社区提交、未经审计，featured 不等于审核通过**。即「注册账号 → 发帖分享」，**无审核环节**。
- 来源：https://docs.openwebui.com/features/extensibility/community

**5) 接 Palimpsest 路径与工作量**
- 路径：在 Admin > Integrations 加一个「MCP (Streamable HTTP)」类型连接，URL 指向 Palimpsest 的 streamable-http MCP 端点。
- 工作量：**低**。纯配置 + 一次工具列表/调用验证。
- 推断（非官方）：若想让记忆出现在**每轮对话的上下文**里（而非仅作为可调用工具），Open WebUI 的 Memory 是内置实现、不开插槽，需要额外写 Workspace Tool/Filter 来注入，成本会上升；仅「让模型能调用 Palimpsest 工具」则很低。

---

### 2.2 LibreChat

**1) 挂外部记忆的机制**
- **MCP servers** 是第一等集成；另有 Agents 的 `memory` 能力与 Actions（OpenAPI）。
- 来源：https://www.librechat.ai/docs/features/mcp

**2) MCP 支持**（官方）
- ✅ 原生 MCP。配置在 `librechat.yaml` 的 `mcpServers`，也可直接在 UI 的 MCP Settings 面板添加（无需改文件/重启）。
- 传输：支持 stdio / SSE；**生产环境推荐 Streamable HTTP**（无状态、适合多用户扩展）。
- 支持 OAuth（含 Dynamic Client Registration）；`mcpSettings` 控制安全（`allowedDomains` / `allowedAddresses` 等）。
- 支持用户级凭据（user-provided credentials）。
- 来源：https://www.librechat.ai/docs/features/mcp 、https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/mcp_servers 、https://www.librechat.ai/docs/features/agents

**3) 自带记忆**（官方）
- **User Memory 是 key/value store**，官方明确写「**不是**语义记忆，不索引/嵌入/检索历史对话」，是「每轮注入的持久化便签」。
- 必须在 `librechat.yaml` 配置 `memory` 对象才生效，**默认不启用**。
- 字段示例（官方）：`memory.disabled`、`personalize`、`tokenLimit`、`maxInputTokens`、`messageWindowSize`、`validKeys`、`agent.enabled`、`agent.provider`、`agent.model`、`agent.instructions`、`charLimit`。
- Agent 挂 Memory 后有 `set_memory` / `delete_memory` 工具；支持「按 Agent 隔离记忆分区」。
- 来源：https://www.librechat.ai/docs/features/memory 、https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/memory
- 与第三方记忆是否冲突：官方**未提供**「可插拔 Memory Provider」接口（对比：Open WebUI 也没有换脑插槽，只有 MCP 工具）。**推断**：第三方记忆只能以 MCP 工具形式并存，内建 key/value memory 与外部记忆**互不感知**；若要「用 Palimpsest 完全替代内置记忆」，只能把内置 memory 关掉（`disabled: true`）+ 走 MCP 工具，且失去自动注入。

**4) 上架机制**
- **没有官方自建插件市场**。官方推荐用 **Smithery**（https://smithery.ai）发现并安装 MCP server。Smithery 是第三方 MCP 目录，非 LibreChat 自有上架审核体系。
- 来源：https://www.librechat.ai/docs/features/mcp

**5) 接 Palimpsest 路径与工作量**
- 路径：`librechat.yaml` 加 `mcpServers` 条目指向 Palimpsest（推荐 streamable-http）→ 重启（或 UI 添加免重启）→ Agent Builder 挂载工具。
- 工作量：**低—中**。配置本身简单。
- 需注意（官方已知限制，来自官方文档的 Future Improvements 与社区 Discussion）：**工具调用无法自动识别用户身份**——多用户场景下，MCP 工具要做到 per-user 记忆隔离，得额外把用户标识传进工具调用（社区讨论 #5378 提到这是 deal-breaker 之一）。这是接 Palimpsest 多用户隔离时必须解决的工程点。

---

### 2.3 AnythingLLM

**1) 挂外部记忆的机制**
- **MCP servers**（AI Agents 用）+ **Custom Agent Skills**（Node.js）。
- 来源：https://docs.anythingllm.com/mcp-compatibility/overview 、https://docs.anythingllm.com/agent/custom/introduction

**2) MCP 支持**（官方）
- ✅ 支持全部 MCP **tools**（官方明确：不支持 Resources / Prompts / Sampling）。
- 传输：**stdio / SSE / Streamable** 三种（`type` 字段，streamable 需 `url`；stdio 需 `command`）。
- 配置：`plugins/anythingllm_mcp_servers.json`（结构同 MCP Server Specification）+ UI 管理页（启动/停止/看日志/看工具列表/删除）。
- 可选 `anythingllm.autoStart: false` 阻止自动启动（AnythingLLM 特有）。
- 来源：https://docs.anythingllm.com/mcp-compatibility/overview 、https://docs.anythingllm.com/mcp-compatibility/docker 、https://docs.anythingllm.com/mcp-compatibility/desktop

**3) 自带记忆**
- 有 agent 长期记忆（`agent_memory.txt`）+ RAG Search 可让 agent 把内容写进自己的 memory 以便后续召回。
- 来源：https://docs.anythingllm.com/agent/usage/rag-search
- ⚠️ 官方文档对 `agent_memory.txt` 的写入策略/索引/审计说明不完整，社区 issue #4821 公开质疑其黑盒性（「什么时候写、怎么索引、用户如何审计/关闭」官方未答）。
- 与第三方记忆冲突：**未能确认**官方是否有冲突处理；**推断**：内置 agent memory 与 MCP 外部记忆是两套，互不感知。

**4) 上架机制**（官方）
- **Community Hub**：https://hub.anythingllm.com ，平台+市场，支持分享 **Agent Skills / System Prompts / Slash Commands**，可公开或私密。**目前 beta**，Workspaces / Data Connectors / Auth Providers「coming soon」。
- 来源：https://docs.anythingllm.com/community-hub/about

**5) 接 Palimpsest 路径与工作量**
- 路径 A（推荐）：`anythingllm_mcp_servers.json` 加 Palimpsest MCP server（stdio 或 streamable）+ UI 启动。
- 路径 B：写 custom agent skill（Node.js）直连 Palimpsest REST。
- 工作量：**低**（配置）+ 可选把 skill 发布到 Community Hub 做占位（beta，门槛低、无审核描述）。

---

### 2.4 LobeChat / LobeHub

**1) 挂外部记忆的机制**
- 插件系统（函数调用）+ **MCP 插件** + 内置 **Agent Memory 插件**。
- 来源：https://lobehub.com/docs/usage/getting-started/memory 、https://lobehub.com/mcp

**2) MCP 支持**（官方/生态）
- ✅ 原生 MCP。插件商店内「Add MCP Plugin」可加自定义 MCP 插件，本地/远程均可（社区文档示例含 `mcp:sse`、命令式 stdio；阿里云教程示例选 Streamable HTTP 模式）。
- 来源：https://lobehub.com/mcp （官方市场页）、https://github.com/DBFritz/lobechat-mcp-plugin 、https://www.alibabacloud.com/blog/602178
- 注意：LobeHub 的 MCP Marketplace 页面自称收录 **107,189** 个 MCP servers（官方页面数字，随时变动）。

**3) 自带记忆**（官方）
- **Agent Memory 是内置插件**，需逐个 Agent 开启（Agent Profile → + Add Plugin → Memory）。
- 记忆结构化：每条含 Title、Preference Weight、Created Time、Memory Instruction、Possible Actions、Tags；有 category（Professional/Personal/Projects/Skills/Relationships/Custom）与 confidence（高/中/低）。
- 有专门的 **Memory Agent** 负责抽取/组织/维护/注入。
- 另有 **Agent self-learning (Labs)** 为独立实验能力，与 User Memory 分开（官方明确「开启 Memory 不等于开启 self-learning，也不训练模型」）。
- 来源：https://lobehub.com/docs/usage/getting-started/memory
- 与第三方冲突：**未能确认**官方冲突说明；**推断**：Agent Memory 是内置插件，与外部 MCP 记忆并存但互不感知。

**4) 上架机制**（官方）
- MCP 上架：LobeHub MCP Marketplace → 官方 CLI **`@lobehub/market-cli`**（`login` + `github connect` → `plugin init` → `plugin publish`）。需 **Node 22+**，需浏览器登录（社区指南称「两次登录，每台机器一次」）。
- 官方发布指引：https://lobehub.com/publish-mcp/skill.md ；市场页 https://lobehub.com/mcp
- 另有旧插件体系：向 `lobe-chat-plugins` 仓库提交 PR 上架官方插件市场（https://chat-plugin-sdk.lobehub.com/quick-start/intro）。
- 来源：https://manycp.com/directories/lobehub 、https://chat-plugin-sdk.lobehub.com/quick-start/intro

**5) 接 Palimpsest 路径与工作量**
- 接入：插件商店 → Add MCP Plugin → 填 Palimpsest MCP URL → 装 → 在 Agent 里启用。工作量 **低**。
- 上架：走 market-cli 发布 Palimpsest 到 Marketplace，工作量**中**（CLI + 双登录 + manifest 维护）。

---

### 2.5 Cherry Studio

**1) 挂外部记忆的机制**
- **MCP 服务器**（内置市场 + 服务商集成）+ 技能 + Agent。
- 来源：https://docs.cherryai.com.cn/advanced-basic/mcp

**2) MCP 支持**（官方中文文档）
- ✅ 原生 MCP。客户端添加：`设置 → MCP 服务器 → 添加`；传输支持 stdio / SSE / **Streamable HTTP**（社区教程示例：类型选「可流式传输的 HTTP (streamableHttp)」，填 URL + Authorization Header）。
- **内置 MCP 市场**：`设置 → MCP 服务器 → 发现 → 内置服务器`，可一键安装社区和官方插件；`服务商` 集成部分服务商。
- 与 **ModelScope 魔搭** 官方合作（v1.2.9+）：可「同步服务器」增量拉取魔搭社区海量 MCP 服务器。
- 来源：https://docs.cherryai.com.cn/advanced-basic/mcp 、https://docs.cherry-ai.com/advanced-basic/mcp/tian-jia-modelscope-mcp-fu-wu-qi

**3) 自带记忆**（官方中文文档）
- **Agent 记忆**：跟随 Agent（不跟随任务/工作目录），保存长期有效的偏好、项目事实、技术决策、经验；一次性进度记成带时间的日志。
- 官方明确「记忆不能代替知识库」：记忆=少量稳定跨任务事实；知识库=成体系文档 + 可控检索范围。
- 来源：https://docs.cherryai.com.cn/advanced-basic/agent-workspace/permissions-memory-background
- 注：另有社区 issue #14214 请求「可插拔 Memory Provider（如 Hindsight）」，官方标注为 inactive/p2，说明**目前无官方可插拔记忆层**（除 MCP 途径外），与背景描述一致。

**4) 上架机制**
- 有内置 MCP 市场（`设置 → MCP 服务器 → 发现 → 内置服务器`）和与魔搭的同步，但**面向第三方开发者的开放上架入口/审核要求：未能确认**（未找到官方「提交你的 MCP 到 Cherry 市场」的公开文档）。
- 社区分享主要走 GitHub Discussion（例：https://github.com/CherryHQ/cherry-studio/discussions/12331 ）。
- 来源：https://docs.cherryai.com.cn/advanced-basic/mcp

**5) 接 Palimpsest 路径与工作量**
- 路径：`设置 → MCP 服务器 → 添加`，类型选 Streamable HTTP，URL 指向 Palimpsest MCP 端点（+ Header 认证如有）。
- 工作量：**低**（纯配置）。
- 上架：**未能确认**官方开放提交机制。

---

## 3. 结论：哪几个的插件市场最值得占位

**按「上架门槛低 + 用户量大 + MCP 原生 + 有明确分发入口」排序，最值得占位的是这三个：**

1. **LobeHub MCP Marketplace（首选）**
   唯一有**正式、可脚本化、官方 CLI 驱动**的 MCP 上架通道（`@lobehub/market-cli`），且市场规模大（官方页称收录 10 万+ MCP servers）、LobeChat 原生 MCP + 内置记忆，Palimpsest 作为「带知识图谱 + bi-temporal 演化 + 本地优先」的记忆 server 定位清晰、差异化强。代价是要过 CLI 双登录，属一次性投入。

2. **Open WebUI 社区站（次选，占位成本最低）**
   社区站**注册即可发帖分享、无审核**，Open WebUI 用户基数大、原生 MCP（Streamable HTTP）。注意其社区站分发的主要是 Python Tools/Functions 而非 MCP server——**推断**：最省事的占位是发一个「Palimpsest 连接指南/配置帖」，同时 Palimpsest 侧保证 Streamable HTTP 端点开箱可用。

3. **AnythingLLM Community Hub（第三，赌它长大）**
   Hub 目前 **beta**、明确支持 Agent Skills 且「Workspaces/Data Connectors soon」，门槛低、官方活跃；AnythingLLM 三种 MCP 传输全支持，接 Palimpsest 最省事。适合早期占位吃「先发」红利。

**其余两个评估：**
- **LibreChat**：MCP 原生且质量高（推荐 Streamable HTTP、有 OAuth），但**没有自有插件市场**，分发借道第三方 Smithery。占位价值主要在「写一份 LibreChat + Palimpsest 集成指南」，而非「上架」。
- **Cherry Studio**：MCP 体验好（内置市场 + 魔搭同步）、中文用户多，但**未找到面向第三方的开放上架入口**——现阶段只能做「用户自己加 MCP 服务器」，无市场占位动作可做。

**一句话总结**：真想「占位」——LobeHub 是唯一首选（有正规通道）；Open WebUI 和 AnythingLLM 做「低成本曝光」，LibreChat/Cherry Studio 只做接入文档、不做上架。

---

## 附：接入 Palimpsest 的通用最优解（推断）

Palimpsest 已支持 **stdio + streamable-http** 两种 MCP 传输，而这 5 个产品**全部**支持 Streamable HTTP（Open WebUI 甚至只支持它）：
- **stdio**：AnyAnythingLLM、LibreChat、LobeChat、Cherry Studio 支持；Open WebUI 需 mcpo 代理。
- **streamable-http**：5 家全支持 ← **跨产品统一入口，优先把 Palimpsest 的 streamable-http 端点做成一等公民**。

即：只要 Palimpsest 的 streamable-http MCP 端点稳定，**同一个配置形态可以覆盖全部 5 个宿主**，无需为每个宿主定制。这是接入成本最低、覆盖面最广的路径。

（本文件所有断言均附 URL；标注「推断」处为基于官方事实的推论，标注「未能确认」处为查无官方明确说法。）
