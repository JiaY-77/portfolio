# 调研：Dify/FastGPT、编码 agent，以及国内 agent 平台插件市场

> 调研日期：2026-10-14
> 调研对象：Dify、FastGPT、Aider、Goose(Block)、OpenHands，另加国内 agent 平台插件市场
> 纪律：官方文档明确写的标「官方」；我方推断标「推断」；查不到写「未能确认」。
> 每条结论后附 URL。

---

## 一、对比表

| 产品 | 插件/工具机制 | MCP 支持 | 上架/分发渠道 | 接入路径 | 工作量 |
|---|---|---|---|---|---|
| **Dify** | 插件系统（plugin，`.difypkg` 包）：工具/模型/Agent 策略/数据源/扩展/合集六类。官方：[Develop Plugin](https://docs.dify.ai/en/develop-plugin/publishing/marketplace-listing/submit-plugin-to-marketplace) | **双向，但仅 HTTP**。作为客户端：`Tools > MCP > Add MCP Server`，「Only MCP servers with HTTP transport are supported」。作为服务端：应用可「Publish as an MCP Server」。官方：[Use Tools](https://docs.dify.ai/en/cloud/use-dify/workspace/tools)、[Publish MCP](https://docs.dify.ai/en/cloud/use-dify/publish/publish-mcp)、[v1.6.0 双向支持](https://dify.ai/blog/v1-6-0-built-in-two-way-mcp-support) | ①官方 Marketplace（PR 到 `langgenius/dify-plugins`，12 项自动检查+人工审核）②自建 GitHub 仓库（免审，用 repo URL 安装）③本地 `.difypkg` 文件直发 | ①纯 MCP：把 Palimpsest 的 streamable-http 端点填进 Dify 的 MCP 客户端即可，**零代码**；②要进 Marketplace：写 Dify 工具插件（Python，包 `dify-plugin>=0.9.0`），内部转调 Palimpsest MCP/REST | ①纯 MCP 接入：**极小**（约 0.5 天，含配置与联调）；②做插件上架：**中**（3~5 天，含打包、隐私政策、README 英文、本地 validator、PR 应答） |
| **FastGPT** | 插件系统（`.pkg` 包）：系统工具/工具集/模型预设/App 模板。官方：[插件系统说明](https://doc.fastgpt.io/zh-CN/plugin/intro) | **双向，偏 HTTP/SSE**。作为客户端：「MCP 工具集」应用类型，填 MCP 地址（SSE URL）解析工具。作为服务端：「MCP server」发布。官方：[MCP 工具集](https://doc.fastgpt.io/zh-CN/guide/build/tools/mcp_tools)、[MCP 发布](https://doc.fastgpt.io/zh-CN/guide/build/publish/mcp_server) | ①官方 Marketplace（社区插件先提 `labring/fastgpt-community-plugins` 仓库，基础审核后进市）②团队上传 `.pkg` ③系统管理员上传。Marketplace 是 SaaS 分发，**不提供私有化版本** | ①纯 MCP：「新建 → MCP 工具集」，填 Palimpsest 服务地址（需 HTTP/SSE，**stdio 不支持**），**零代码**；②要上架：用 `@fastgpt-plugin/cli` + `@fastgpt-plugin/sdk-factory` 写插件（TS，声明 manifest/inputSchema/outputSchema/secretSchema），打包 `.pkg` 后向 community 仓库提 PR | ①纯 MCP 接入：**小**（1 天，需先把 Palimpsest 的 streamable-http 暴露到 FastGPT 容器可达的网络）；②做插件上架：**中**（3~5 天，TS + 双语文案 + 端到端测试） |
| **Aider** | 无插件系统。能力扩展靠三件套：`/read`（只读文件注入）、`/run`（执行 shell 命令）、`/web`（抓网页）。官方：[Specifying coding conventions](https://aider.chat/docs/usage/conventions.html) | **不支持（原生）**。截至 2026-07（aider 0.86.x）配置参考里没有 MCP 选项，MCP 相关 PR 未合并，issue #4506 仍开着。官方 issue：[Aider-AI/aider#4506](https://github.com/Aider-AI/aider/issues/4506)；第三方佐证：[Warp 指南](https://www.wearewarp.com/agents/mcp/aider)、[DeployHQ 指南](https://www.deployhq.com/guides/aider) | 无插件市场。分发=PyPI 包 + GitHub 仓库 | ①**项目级记忆已内建**：`.aider.conf.yml` 里 `read:` 一个 markdown 文件，会话每次自动加载（就是当年 Hermes MEMORY.md 那套的角色）；②要真·外部记忆：得靠 `mcpm-aider` 这类**第三方**桥（未官方支持、实验性），或写 `/run` 调 Palimpsest CLI | ①简单：写个 `CONVENTIONS.md`（Palimpsest 导出要点）+ `read:` 配置，**极小**（1 小时，但是浅集成、只读、非检索）；②真 MCP 工具调用：**大且不可靠**（依赖第三方桥，官方不背书） |
| **Goose (Block)** | 扩展系统 = MCP server（内置扩展本身就是 MCP server）。官方：[Using Extensions](https://goose-docs.ai/docs/getting-started/using-extensions)、[Building Custom Extensions](https://goose-docs.ai/docs/tutorials/custom-extensions) | **原生，全支持**。任何 MCP server 都能 `goose configure` → Add Extension 装上，stdio(Standard IO)/SSE/HTTP 都行；也支持 MCP Roots（扩展能看到会话工作目录） | ①官方扩展目录（内置 + 精选）②任何第三方 MCP server 都能自己加（**无需上架**） | ①**首选**：把 Palimpsest 的 MCP server 直接加成 Goose 扩展，`goose configure` 里选 Standard IO 填启动命令即可，**零代码**；②Goose 自带 Memory 扩展（`.goose/memory/` 本地文件，工具 `remember_memory` 等）可作参照或并存 | **极小**（约 0.5 天，纯配置）。这是全套里最省事的一个 |
| **OpenHands** | MCP + 两级持久记忆（持久记忆为 opt-in 文件制）。官方：[MCP 总览](https://docs.openhands.dev/overview/model-context-protocol)、[Persistent Memory](https://docs.openhands.dev/sdk/guides/persistent-memory)、[CLI MCP Servers](https://docs.openhands.dev/openhands/usage/cli/mcp-servers) | **原生，全支持**。CLI 用 `openhands mcp add <name> --transport stdio/http`，配置在 `~/.openhands/mcp.json`；SDK 用 `mcp_config` 字典；支持 stdio / SSE / Streamable HTTP 三种传输 | 无插件市场。分发=PyPI 包 + GitHub 仓库 | ①**首选**：`openhands mcp add palimpsest --transport stdio -- python -m palimpsest_mcp` 之类的 MCP 注册，**零代码**；②替代/补充：持久记忆走 `AgentContext(load_memory=True)`，读 `~/.openhands/memory/MEMORY.md` + `./.openhands/memory/MEMORY.md` 两级 markdown（**约 6000 字符上限**，超出从顶部整行丢弃）。可让 Palimpsest 生成/同步这份 MEMORY.md 摘要 | ①纯 MCP：**极小**（约 0.5 天）；②记忆文件同步器（Palimpsest → MEMORY.md 摘要）：**小**（1~2 天，注意 6000 字符预算与「不可信内容」包裹） |

---

## 二、Dify / FastGPT 插件市场上架机制与门槛（细项）

### Dify Marketplace
- **提交入口**：向 `langgenius/dify-plugins` 仓库提 PR（fork → 分支 → 每次 PR 只放一个 `.difypkg`）。
  URL：https://github.com/langgenius/dify-plugins ｜ 文档：https://docs.dify.ai/en/develop-plugin/publishing/marketplace-listing/submit-plugin-to-marketplace
- **账号**：需 GitHub 账号（PR 用）。发布后想看待下载/评分/反馈，走 **Creator Center**，用个人账号连接发布插件的 GitHub 账号。URL：https://creators.dify.ai/
- **审核**：Marketplace 走「12 项自动检查 + 人工审核」；可先用自建 GitHub 仓库免审分发，稳定后再进市。官方：[Publish Plugins 概览](https://docs.dify.ai/en/develop-plugin/publishing/marketplace-listing/release-overview)
- **许可**：**未强制**要求开源许可（文档未把 license 列为门槛）。但**硬性要求**：manifest 要声明 author/name/version/type/runner/icon/**source repository**/contact/**privacy policy**；`README.md` 主文档必须英文；`PRIVACY.md` 说明数据收集/外发；Python 插件须 `dify-plugin>=0.9.0`；依赖做 OSV 漏洞扫描；PR 模板必须选一个风险等级（Low/Medium/High）。官方同上。
- **隐私/网络申报**：可在 `manifest.yaml` 里 `network.domains` 声明外联域名。官方同上。

### FastGPT Marketplace
- **提交入口**：社区插件先提交到 `labring/fastgpt-community-plugins` 仓库，经**基础审核**后再进 Marketplace。
  URL：https://github.com/labring/fastgpt-community-plugins ｜ 开发文档：https://github.com/labring/fastgpt-plugin/blob/main/docs/dev/how-to-devlop-plugin.md
- **账号**：需 GitHub 账号（提 PR）。分发边界写明 **Marketplace 是 SaaS 分发服务，不提供私有化部署版本**；私有化用户走「团队上传插件」或系统管理员上传 `.pkg`。官方：[插件系统说明](https://doc.fastgpt.io/zh-CN/plugin/intro)
- **门槛**：官方插件（`fastgpt-official-plugins`）需经过更严格的代码/功能/安全/上架测试；社区插件保留来源与风险提示。manifest 的 `pluginId`、输入输出字段名发布后须保持稳定；名称/描述用 `{ en, 'zh-CN' }` 双语；密钥字段须标 `isSecret: true`。
- **许可**：文档**未列出**开源许可作为硬门槛（「未能确认」是否有隐含要求）。
- **注意**：插件最终以 `.pkg` 文件交付，需在测试环境手动安装并做端到端测试（本地 debug 不模拟生产子进程池/真实 IPC/网络/超时）。

---

## 三、中文生态：除 Dify/FastGPT 外，还有哪些国内 agent 平台有插件市场

| 平台 | 是否插件市场 | 上架门槛 | 官方 URL | 状态提示 |
|---|---|---|---|---|
| **扣子 / Coze** | ✅ 有（插件商店 + 2026-01 上线的 Agent Skills/技能商店；开发平台 2025-11 起内置 MCP 客户端，可「基于 MCP 服务创建插件」） | 个人空间建插件 → 调试通过 → 点「上架插件」→ **平台审核通过后进商店**。注册即用（火山引擎/扣子账号）。官方更新日志：[扣子更新动态](https://docs.coze.cn/recent-updates)；插件配置：[coze-studio Wiki](https://github.com/coze-dev/coze-studio/wiki/4.-%E6%8F%92%E4%BB%B6%E9%85%8D%E7%BD%AE)。具体资质/许可要求**未能确认**（官方文档未找到明确条目） | 上架流程见第三方教程佐证：[火山引擎社区](https://developer.volcengine.com/articles/7387288256942981139) |
| **智谱清言 / BigModel** | ✅ 有（「插件广场」；支持自建插件上架公开） | 自建插件填名称/类别/描述/图标/鉴权 → 配 OpenAPI 3.0 schema → 调试通过 → 保存 → **可选择上架至插件广场公开给所有用户**。官方：[智能体开发平台](https://docs.bigmodel.cn/cn/guide/platform/intelligent-agent) | 官方文档明确写了「可以选择将您自建的插件上架至插件广场，公开给所有用户使用」。具体审核时长/资质**未能确认** |
| **Cursor（光标）** | ✅ 有（Cursor Marketplace 官方插件 + cursor.directory 社区 MCP 目录） | 社区插件提交入口：https://cursor.directory/plugins/new（旧仓库 `cursor/mcp-servers` 已废弃并指向此处）。官方：[Cursor MCP 文档](https://cursor.com/docs/mcp) | 门槛细节**未能确认**（目录页未给审核标准说明） |
| **腾讯元器** | ⚠️ **已停服** | 不适用 | https://yuanqi.tencent.com/cl/service-discontinuation | **官方公告：2026-11-09 正式停止全部服务**；2026-10-29 起停止创建新智能体与 API key 分发；数据保留过渡期至 2027-01-09。公告更新于 2026-10-08。新闻佐证：[新浪科技](https://finance.sina.com.cn/tech/roll/2026-10-12/doc-iniuxmtq0466773.shtml) |
| **百度文心智能体平台 (AgentBuilder)** | ⚠️ **收缩中** | 不适用 | https://agents.baidu.com/docs/commonQuestion/question | 报道称平台已关闭智能体创建、做同款、工作流创建入口（8-31 起），已创建的暂不受影响。来源：[人人都是产品经理](https://www.woshipm.com/ai/6476321.html)（**该条为媒体转述，百度官方公告 URL 未能确认**） |

---

## 四、结论：国内插件市场值不值得投入，先做哪个

**先说结论：国内「插件市场」这条线，现在不值得作为 Palimpsest 的主投入方向；应该先接现成的 MCP 通道（Goose / OpenHands / Dify / FastGPT 的 MCP 客户端），而不是去平台上架插件。**

理由（按分量排序）：

1. **国内插件市场正在退潮，且上架机制对「本地优先」型产品天然不友好。**
   最硬的证据：腾讯元器官方公告 2026-11-09 停服（https://yuanqi.tencent.com/cl/service-discontinuation），百度文心智能体平台据报已关停创建入口（https://www.woshipm.com/ai/6476321.html）。这类「零代码智能体商店」是第一代形态，正在集体收缩，押注其上架生态风险高。
   另外 FastGPT 明确写了「Marketplace 是 SaaS 分发服务，**不提供私有化部署版本**」（https://doc.fastgpt.io/zh-CN/plugin/intro）——而 Palimpsest 的核心卖点是「数据全本地、不上云」，两者的价值主张存在结构性张力。

2. **Dify / FastGPT 都支持 MCP 客户端，而且零代码就能接。**
   Dify：`Tools > MCP > Add MCP Server`，官方明示「Only MCP servers with HTTP transport are supported」（https://docs.dify.ai/en/cloud/use-dify/workspace/tools）。
   FastGPT：「MCP 工具集」应用类型，填 MCP 地址解析工具（https://doc.fastgpt.io/zh-CN/guide/build/tools/mcp_tools）。
   → 只要 Palimpsest 暴露 **streamable-http** 端点（其 MCP 已支持该传输），就能被这两家**直接挂上，不用写一行插件代码、不用过审**。性价比碾压「做插件上架」。

3. **编码 agent 那三个里，Goose 和 OpenHands 是最值得先接的，且都是纯配置。**
   - **Goose**：扩展系统本身就是 MCP，任何 MCP server 都能 `goose configure` 装上（https://goose-docs.ai/docs/getting-started/using-extensions）。零代码。
   - **OpenHands**：`openhands mcp add ... --transport stdio`，配置在 `~/.openhands/mcp.json`（https://docs.openhands.dev/openhands/usage/cli/mcp-servers）。零代码。另有原生两级持久记忆可直接受益（https://docs.openhands.dev/sdk/guides/persistent-memory）。
   - **Aider**：**不支持原生 MCP**（issue #4506 仍开，配置参考无 MCP 项，https://github.com/Aider-AI/aider/issues/4506），只能靠第三方实验性桥（`mcpm-aider`）。投入产出比最差，**建议最后考虑或跳过**。

4. **国内平台里唯一的「真·MCP 上架」机会是扣子/Coze**：2025-11 起开发平台内置 MCP 客户端，支持「基于 MCP 服务创建插件」（https://docs.coze.cn/recent-updates），且 C 端有插件商店（个人空间 → 上架插件 → 审核 → 进商店）。它目前仍在活跃迭代（不像元器/文心在收缩）。**若将来要碰国内上架，Coze 是唯一值得评估的候选**，且路径恰好也是 MCP，与 Palimpsest 现有能力对齐。

**推荐执行顺序（投入从小到大、收益从快到慢）：**

1. **Goose + OpenHands 的 MCP 接入**——纯配置，各 0.5 天，立刻可验证，先做。这也和「先做能用起来的真实宿主」一致。
2. **Dify / FastGPT 的 MCP 客户端接入**——需要把 Palimpsest 的 streamable-http 端点暴露到它们可达的网络（FastGPT 私有化要处理容器网络），约 1 天/家。做完能覆盖两大国内平台，且零上架成本。
3. **OpenHands 的 MEMORY.md 摘要同步器**（可选加分项）——1~2 天，注意 6000 字符预算与「不可信内容」包裹语义。
4. **Coze 插件（基于 MCP）**——若确有国内分发诉求再评估；先看 Coze 官方对「基于 MCP 服务创建插件」的具体要求。
5. **Dify / FastGPT 的 Marketplace 上架**——**不建议现在做**。等 MCP 通道验证过真实用户需求、且平台生态方向明朗后再回头考虑；届时 Dify 的路子（纯 PR + 隐私政策 + 英文 README）比 FastGPT（TS 插件 + SaaS-only 分发）更轻。
6. **Aider**——**建议跳过**。无原生 MCP，第三方桥不可靠，且 Aider 的目标用户（终端派单兵改代码）与「跨会话记忆底座」的契合度本身较低。

---

## 五、关键事实与 URL 速查（便于引用）

**Palimpsest 侧（本次调研的输入事实）**
- 三种接口共用一核心：MCP（stdio + streamable-http）/ FastAPI REST / CLI。
- 目前只做了 Hermes 一个宿主插件（Memory Provider + Context Engine）。

**Dify**
- 插件开发与上架：https://docs.dify.ai/en/develop-plugin/publishing/marketplace-listing/submit-plugin-to-marketplace
- 三种分发方式对比：https://docs.dify.ai/en/develop-plugin/publishing/marketplace-listing/release-overview
- 插件仓库：https://github.com/langgenius/dify-plugins
- MCP 工具（客户端，仅 HTTP）：https://docs.dify.ai/en/cloud/use-dify/workspace/tools
- 应用发布为 MCP Server：https://docs.dify.ai/en/cloud/use-dify/publish/publish-mcp
- 双向 MCP 官宣（v1.6.0）：https://dify.ai/blog/v1-6-0-built-in-two-way-mcp-support
- Creator Center：https://creators.dify.ai/

**FastGPT**
- 插件系统说明 / Marketplace 边界：https://doc.fastgpt.io/zh-CN/plugin/intro
- MCP 工具集（客户端）：https://doc.fastgpt.io/zh-CN/guide/build/tools/mcp_tools
- MCP 发布（服务端）：https://doc.fastgpt.io/zh-CN/guide/build/publish/mcp_server
- 系统工具开发指南：https://doc.fastgpt.io/zh-CN/plugin/system-tool-development
- 开发者文档：https://github.com/labring/fastgpt-plugin/blob/main/docs/dev/how-to-devlop-plugin.md
- 社区插件仓库（上架入口）：https://github.com/labring/fastgpt-community-plugins

**Aider**
- 约定文件机制（`/read` + `.aider.conf.yml` 的 `read:`）：https://aider.chat/docs/usage/conventions.html
- MCP 支持状态（Native MCP issue，仍开）：https://github.com/Aider-AI/aider/issues/4506
- 佐证「无原生 MCP」：https://www.wearewarp.com/agents/mcp/aider ｜ https://www.deployhq.com/guides/aider

**Goose (Block)**
- 扩展（MCP）总览：https://goose-docs.ai/docs/getting-started/using-extensions
- 构建自定义扩展：https://goose-docs.ai/docs/tutorials/custom-extensions
- 内置 Memory 扩展：https://goose-docs.ai/docs/mcp/memory-mcp
- MCP 协议说明：https://block-goose.mintlify.app/development/mcp-protocol

**OpenHands**
- MCP 总览（支持 stdio/SSE/SHTTP）：https://docs.openhands.dev/overview/model-context-protocol
- CLI MCP 配置（`~/.openhands/mcp.json`）：https://docs.openhands.dev/openhands/usage/cli/mcp-servers
- SDK MCP 指南（`mcp_config`）：https://docs.openhands.dev/sdk/guides/mcp
- 持久记忆（两级 MEMORY.md，opt-in）：https://docs.openhands.dev/sdk/guides/persistent-memory
- Context Condenser：https://docs.openhands.dev/sdk/guides/context-condenser

**国内平台**
- 扣子/Coze 更新动态（含 MCP 插件、技能商店）：https://docs.coze.cn/recent-updates
- 扣子插件配置（coze-studio 开源版）：https://github.com/coze-dev/coze-studio/wiki/4.-%E6%8F%92%E4%BB%B6%E9%85%8D%E7%BD%AE
- 智谱智能体开发平台（插件广场上架）：https://docs.bigmodel.cn/cn/guide/platform/intelligent-agent
- Cursor MCP 文档：https://cursor.com/docs/mcp ｜ 社区提交入口：https://cursor.directory/plugins/new
- 腾讯元器停服公告：https://yuanqi.tencent.com/cl/service-discontinuation
- 百度文心收缩报道（非官方）：https://www.woshipm.com/ai/6476321.html

**未能确认的事项（诚实标注）**
- Dify Marketplace 是否有开源许可的硬性要求：文档未列，**未能确认**。
- FastGPT Marketplace 是否有许可/资质硬门槛：文档未列，**未能确认**。
- 扣子插件商店的具体审核标准与资质要求：**未能确认**（仅确认有「上架插件 → 平台审核 → 进商店」流程）。
- 智谱插件广场的审核时长与资质要求：**未能确认**（仅确认可上架）。
- Cursor 社区插件（cursor.directory）的审核标准：**未能确认**。
- 百度文心 AgentBuilder 关闭创建入口的官方公告原文 URL：**未能确认**（仅有媒体转述）。
