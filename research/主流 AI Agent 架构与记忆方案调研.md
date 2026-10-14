# 主流 AI Agent 的架构与记忆方案调研

> 一手查证，2026-10-09。**每条结论标官方来源 URL**，数字标注查询时间。
> **方法**：GitHub 官方 API（`gh api`）取仓库事实；官方文档/仓库内 docs 取架构与记忆说明。**凡查不到即标注，不编。**
> 姊妹篇：《主流记忆工具对比（Mem0 / Letta / Cognee / Zep）》。

---

## 0. 一句话结论

五个 Agent 都**有记忆**，但**没有一个是"记忆优先"的系统**——
- **Codex / Claude Code**：记忆是"**给单个编码助手用的本地 Markdown 便签**"，绑死各自的 CLI，服务"这项目怎么改代码"，不服务"跨 Agent 的长期知识"。
- **OpenClaw / nanobot**：记忆是"**文件 + 一层小索引**"（无向量库），把"无隐藏状态、可读可改"当卖点。
- **Hermes**：记忆是"**小容量常驻便签（MEMORY.md/USER.md）+ 按需加载的技能 + 会话全文检索**"，记忆与"自我进化"绑定。
- **五者共同点（也是 Palimpsest 的机会）**：记忆几乎都是**纯文件、无版本链、无冲突消解、绑死宿主**。没有一家同时做到「本地嵌入式 + 混合检索 + 冲突消解/版本链 + 宿主无关」。

---

## 1. 总览对比表

> 仓库事实 = GitHub API，查询时间 **2026-10-09 21:22（UTC+8）**。star 数会变，引用时注明日期。

| Agent | 开发者 | 开源状态 | 语言 | Star（2026-10-09） | 运行位置 | 记忆形态（核心） |
|---|---|---|---|---|---|---|
| **Codex** | OpenAI | ✅ 开源 Apache-2.0（`openai/codex`） | Rust | 128,363 | 本地 CLI 为主 + 云端沙箱（云版闭源） | `AGENTS.md`（手写规则）+ `~/.codex/memories/`（后台自动摘要，纯 Markdown） |
| **Claude Code** | Anthropic | ⚠️ 仓库开源但**无 license 文件**（专有）；SDK/沙箱运行时有开源部分 | TypeScript | 149,825 | 本地 CLI 为主 + 云版（Claude Code on the web） | 4 层 `CLAUDE.md` 作用域 + Auto Memory（`~/.claude/projects/<proj>/memory/`，`MEMORY.md` 索引 200 行/25KB 上限） |
| **OpenClaw** | OpenClaw Foundation（独立 501(c)(3)） | ✅ 开源 MIT（`openclaw/openclaw`） | TypeScript | 391,531 | **本地优先**（自带硬件跑；一个 Gateway，团队部署只是配置差异） | 文件分层（Instructions/Curated core/Episodic/Prospective/Review），纯文件 + 一个 SQLite 索引；"Dreaming"后台整理 |
| **nanobot** | HKUDS（港大数据智能实验室） | ✅ 开源 MIT（`HKUDS/nanobot`） | Python | 48,897 | **本地优先**（自称 ultra-lightweight / self-hosted） | `SOUL.md` + `USER.md` + `memory/MEMORY.md` + `memory/history.jsonl`；无向量库；"Dream"定时整理；Git 版本化 |
| **Hermes Agent** | Nous Research | ✅ 开源 MIT（`NousResearch/hermes-agent`） | Python | 252,193 | 本地/自托管为主（也能跑 VPS/云） | 内置 `MEMORY.md`(2200 字符) + `USER.md`(1375 字符)，常驻注入；Skills=过程记忆；`session_search`=会话全文检索；可挂外部记忆 provider |

**共同参照**：`AGENTS.md` 是一个独立开放标准（`agentsmd/agents.md`，MIT，24,843 star，2026-10-09），Codex 与多个 Agent 都读它。

---

## 2. 逐个拆解

### 2.1 Codex（OpenAI）

**架构 / 运行位置**
- 三种形态：**CLI（本地跑在你自己电脑上）**、IDE 插件、以及 **Codex Web 云端**（chatgpt.com/codex 上跑在 OpenAI 托管沙箱里，可并行）。
  - 来源：github.com/openai/codex README（"Codex CLI is a coding agent from OpenAI that runs locally on your computer… If you are looking for the cloud-based agent… go to chatgpt.com/codex"）
- **开源**：CLI 仓库 `openai/codex` 开源，Apache-2.0，Rust 实现（GitHub API，2026-10-09）。
- 云沙箱版有"沙箱记忆"：workspace 下的 `memories/`（`memory_summary.md`/`MEMORY.md`/`raw_memories.md`/`rollout_summaries/`/`skills/`），"让后续沙箱运行从前序运行学到东西"。
  - 来源：developers.openai.com/api/docs/guides/agents/sandboxes

**记忆方案（分两层）**
1. **静态指令层 = `AGENTS.md`**（手写）——分层合并（hierarchical）：
   - `~/.codex/AGENTS.md`（全局，个人）
   - 项目根 `AGENTS.md`（团队共享）
   - 子目录 `AGENTS.md`（局部细化）
   - `AGENTS.override.md`（Codex 扩展，**替换**父级指令）
   - 加载从 git 根往下走到当前工作目录，逐层叠加；默认有大小上限（约 32 KiB）。
   - 来源：developers.openai.com/codex/guides/agents-md；仓库 docs/agents_md.md 指向该页
2. **生成层 = `~/.codex/memories/`**（后台自动）——"`AGENTS.md` 是手写的静态层；Memories 是 Codex 自己生成的层"，它**在后台总结过往会话**并写入 `~/.codex/memories/`，下次会话读取：
   - `memory_summary.md`：下次会话先读的合并视图
   - `MEMORY.md`：长文合并记忆，按需 grep
   - `raw_memories.md`：合并前的抽取产物
   - `rollout_summaries/*.md`：每次会话的摘要，喂给合并流程
   - `skills/<x>/SKILL.md`：技能专属记忆
   - **无 SQLite、无 embedding 索引、无黑盒 blob，纯 Markdown**。
   - 来源：mem0.ai《How Memory Works in Codex CLI》；mem0 在 X 的同名文章；OpenAI 开发者文档 developers.openai.com/api/docs/guides/agents/sandboxes（`memory_summary.md`/`MEMORY.md`/`rollout_summaries/`）；learn.chatgpt.com/docs/customization/memories
   - **渐进披露**：运行开始注入 `memory_summary.md`，需要时 agent 搜 `MEMORY.md`，更细再开 rollout summaries（同上沙箱文档）。

---

### 2.2 Claude Code（Anthropic）

**架构 / 运行位置**
- **本地 CLI 为主**（跑在你终端），另有 Claude Code on the web（云端）。
- **开源状态（需谨慎）**：仓库 `anthropics/claude-code` 公开，但 **无 license 文件**（GitHub API `license=null`，`/license` 接口 404）——即**源码可见但非标准开源许可**；其**沙箱运行时**（`@anthropic-ai/sandbox-runtime`）以开源 preview 形式发布。
  - 来源：GitHub API（2026-10-09）；anthropic.com/engineering/claude-code-sandboxing（"available as an open source research preview"）
- **沙箱**：`/sandbox` 命令，给 bash 工具加沙箱（限制可访问的目录与网络主机），不重启容器。
  - 来源：anthropic.com/engineering/claude-code-sandboxing
- **Computer Use**：✅ **有**。在 CLI/Desktop 里打开应用、操作你的屏幕，"跑在你真实桌面上"，按应用逐个弹权限提示。
  - 来源：code.claude.com/docs/en/sandboxing（"Computer use: when Claude opens apps and controls your screen, it runs on your actual desktop"）；platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
- **插件体系**：✅ 有。一个 plugin = 一个目录，打包 **skills + subagents(agents/) + hooks(hooks/hooks.json) + MCP servers + commands**，带 `.claude-plugin/plugin.json`，经 **marketplace** 用 `/plugin` 一处安装、统一更新。
  - 来源：code.claude.com/docs/en/plugins/overview；shiplight.ai/blog/claude-code-plugins

**记忆方案**
- **两套系统，都在会话开始加载、会话中持续更新**（不是定时器）：
  1. **`CLAUDE.md`（手写规则，四作用域，拼接而非覆盖）**：
     - 托管策略（managed policy）→ 用户 `~/.claude/CLAUDE.md` → 项目 `./CLAUDE.md` 或 `./.claude/CLAUDE.md` → 本地 `./CLAUDE.local.md`
     - 官方明确："**所有被发现的文件是拼接进上下文，而不是互相覆盖**"。
     - 来源：code.claude.com/docs/en/memory；claudecertificationguide.com（引官方"concatenated into context rather than overriding"）
  2. **Auto Memory（自动）**：Claude 工作中自己保存四类笔记（`user`偏好 / `feedback`纠正 / `project`决策 / `reference`指针），写入 `~/.claude/projects/<project>/memory/`；`MEMORY.md` 作为**索引**（启动时加载**前 200 行或 25 KB**，取先到者），topic 文件按需加载。**机器本地**，同一 git 仓库的 worktree/子目录共享一份；不跨机器/云同步。
     - 来源：code.claude.com/docs/en/memory；vectorize.io/articles/claude-code-memory；milvus.io/blog/claude-code-memory-memsearch
- **后台整合**：官方 Managed Agents API 的 **Dreams**（Research Preview）原语——读记忆库 + 过往会话记录，清理重复/矛盾/过期项；Claude Code 用它做各项目记忆的整合。
  - 来源：vectorize.io/articles/claude-code-memory（引 Anthropic Dreams 官方文档）
- 命令：`/memory`（列出并编辑记忆文件）、`/context`（查看本会话实际加载了什么）。
- ⚠️ 社区提到有未公开的 "KAIROS" 常驻守护进程（来自泄露源码）——**未经官方证实，不写入材料**。

---

### 2.3 OpenClaw（开源，OpenClaw Foundation）

**定位 / 运行位置**
- **本地优先的私人 AI 助理**：跑在**你自己的设备**（自带硬件），一个 **Gateway** 同时管个人助理或团队部署（只是配置差异）。
- **"状态、记忆、凭证都住在你的硬件上"**；模型与 agent harness（Claude、Codex、本地模型）是**可换的插件**。
- 无付费层、无托管服务、无 token；由独立的 501(c)(3) 基金会治理。
  - 来源：github.com/openclaw/openclaw README（2026-10-09）
- 架构：单一长驻 **Gateway** 拥有所有消息面（WhatsApp/Telegram/Slack/Discord/Signal/iMessage/WebChat…），控制面客户端走 WebSocket（默认 `127.0.0.1:18789`）。
  - 来源：仓库 docs/concepts/architecture.md

**记忆方案（"文件 + 一个 SQLite 索引"，分层 + 信任级 + 后台整理）**
- **核心设计原则**（官方原文摘）：
  1. **无隐藏状态**——模型只记得写进 workspace 文件的东西，每个记忆面都可用文本编辑器查看/编辑。
  2. **写才是难点**——长程评测显示"写了什么"比"怎么索引"更重要（引 LongMemEval, arXiv:2410.10813），故把整理挪出回复路径、放进后台。
  3. **写路径就是安全边界**——写入时用来源（provenance）把关，而非事后检测坏记忆。
  4. **确定性门 + 门内模型判断**。
  5. **失败不阻断回复**——每步都有超时/回退。
- **分层**（tier）：Instructions(`AGENTS.md`) / Curated core(`MEMORY.md`+`USER.md`) / Episodic(`memory/YYYY-MM-DD.md`+会话记录) / Prospective(standing intents+cron) / Review(`DREAMS.md`)。
- **来源（provenance）**：每条记忆带 origin class（`owner`/`agent`/`untrusted`/`system`）、session kind、时间戳与 supersession key。
- **写路径**：日常写 daily notes → 压缩前 memory flush 抢救未写内容 → 会话结束 transcript 归档 → 全部落在 episodic 层 → 等 **Dreaming** 后台三步（Light/REM/Deep）过两道门（确定性门 + 模型合并）晋升到 `MEMORY.md`/`USER.md`。
- **召回两条道**：Lane 1 零模型调用（bootstrap 注入 + 排序检索，30 天半衰期 + importance 乘子 + trigger 注入）；Lane 2 升级成子代理。
- **导入他人记忆**：Control UI 可导入 **Codex / Claude Code / Hermes** 的 Markdown 记忆（Hermes 只取 `MEMORY.md`+`USER.md`）。
  - 来源：仓库 docs/concepts/memory-architecture.md、docs/concepts/memory.md

---

### 2.4 nanobot（HKUDS）

**定位 / 运行位置**
- **超轻量、开源、自托管**的私人 AI agent 框架（Python）；WebUI/终端/聊天应用都能跑。
- 官方描述："Ultra-lightweight, open-source, self-hosted personal AI agent framework in Python with WebUI, tools, memory, MCP, multi-agent workflows…"
  - 来源：github.com/HKUDS/nanobot README + GitHub API（2026-10-09）
- **运行位置**：本地优先（也能部署到 Render/Zeabur 等服务器）。
- **⭐ 关于"轻量化程度（行数）"——需纠正的说法**：
  - 官方 philosophy/roadmap 与多个社区帖说"**~4000 行核心**"（如 GitHub Discussion #1609 提到 "Lightweight (~4000 lines core)"；Zeabur 模板页亦称 "~4K lines of core agent code"）。
  - **但实测当前代码规模远超此数**：按 GitHub 仓库文件树，`nanobot/agent/`（agent 核心，55 个 .py 文件）≈ **960 KB / 约 3 万行**；单 `nanobot/agent/loop.py` 就 114 KB；整个 `nanobot/` 包 .py 文件 455 个（含测试）。
  - **结论**：`~4000 行`是**早期的设计口号/路线图目标，不是当前事实**；引用时须注明"官方自称约 4K 行核心，但当前仓库核心已显著增长（≈3 万行）"。**核查时间 2026-10-09。**

**记忆方案（"层层分工的文件式记忆 + Dream 整合 + Git 版本化"）**
- 官方理念："记忆应该感觉是活的，但不应感觉混乱"；**不把记忆当一个巨型文件**，按"不同种类的记忆用不同的工具"分层：
  - `session.messages`：短期活对话
  - `memory/history.jsonl`：压缩过往轮次的**追加式**归档（每行一个 JSON：`{"cursor", "timestamp", "content"}`）
  - `SOUL.md` / `USER.md` / `memory/MEMORY.md`：**持久知识文件**
  - `GitStore`：记录这些持久文件如何随时间变化
- **两阶段流**：
  1. **Consolidator**：会话变大时总结被压缩的对话，追加进 `history.jsonl`；模型基于摘要继续。
  2. **Dream**：更慢、更有思考的一层，默认按 cron 跑（也可手动触发），读新 `history.jsonl` + 当前 `SOUL.md`/`USER.md`/`MEMORY.md`，**外科手术式**最小改动地编辑长期文件（"不是重写一切，而是做出让记忆保持连贯的最小诚实改动"）。
- 四个文件分工：`SOUL.md`=语气/风格；`USER.md`=用户是谁/偏好；`MEMORY.md`=工作本身持久为真的东西；`history.jsonl`=路上发生了什么。
- **无向量库、无 embedding、无 RAG 管线**（社区讨论 #566 亦印证"Two files… No vector DB"）；检索靠 grep/jq。
  - 来源：github.com/HKUDS/nanobot docs/memory.md；docs/architecture.md；GitHub Discussion #566
- 版本化：workspace 有 `.git/`，长期记忆文件走 Git。

---

### 2.5 Hermes Agent（Nous Research）

**定位 / 学习闭环**
- **自我改进的 AI agent**：官方向："The only agent with a built-in learning loop — it creates skills from experience, improves them during use, nudges itself to persist knowledge, and builds a deepening model of who you are across sessions."
  - 来源：hermes-agent.nousresearch.com/docs；github.com/NousResearch/hermes-agent README
- **开源**：MIT，Python，252,193 star（GitHub API，2026-10-09）。可跑 $5 VPS / GPU 集群 / serverless；可从 Telegram 等平台对话。
- **运行位置**：自托管/本地为主（也支持 Docker / Modal / SSH / Daytona / Singularity 等多种部署）。

**学习闭环机制（skill 自创建）**
- Agent 通过 `skill_manage` 工具可创建/更新/删除自己的技能——**这是它的过程记忆（procedural memory）**。
- 官方给的触发时机：想出值得复用的多步工作流时、踩过坑找到可行路径时、**被用户纠正时**。
- 技能遵循 **agentskills.io 开放标准**（同一批 `SKILL.md` 可被其他兼容运行时加载）。
- 有 **write-approval gate**（`skills.write_approval`）可要求写入前先人工审。
  - 来源：hermes-agent.nousresearch.com/docs/user-guide/features/skills（llms-full.txt 11529 起、12130 起）

**记忆架构**
- **内置记忆 = 两个小容量常驻文件**（会话开始作为**冻结快照**注入系统提示）：
  - `MEMORY.md` = agent 个人笔记（环境事实/约定/学到的东西），**2,200 字符**（~800 tokens）
  - `USER.md` = 用户画像（偏好/沟通风格/期望），**1,375 字符**（~500 tokens）
  - **不自动压缩**：写超限时工具返回错误，由 agent 当场腾挪（合并/删除）后重试。
  - 存在 `~/.hermes/memories/`，**按 profile 隔离**。
  - 来源：hermes-agent.nousresearch.com/docs/user-guide/features/memory（llms-full.txt 13057 起）
- **技能 = 过程记忆**（按需加载，零 token 直到被加载）；**记忆 = 事实**（每会话注入，小但恒定成本）。
  - 官方路数："想写进参考文档的是技能，想写便签的是记忆。"
- **会话全文检索**：`session_search`（FTS5 + LLM 摘要）补记忆的空白——记忆围绕"会话结束"这一时刻设计。
  - 来源：同上 memory.md、README
- **可挂外部记忆 provider**（**加法**，内置记忆始终并行）：内置 3 个（Holographic / RetainDB / ByteRover），plugin catalog 另有 Honcho / Hindsight / Supermemory / Mem0 / OpenViking；一次只能激活一个外部 provider。
  - 来源：hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers（llms-full.txt 13561、13579 起）
- **关于草稿里的"Hermes 五层记忆架构"——查无此说**：官方文档对 Hermes 记忆的表述是"**内置 MEMORY.md/USER.md + 技能 + 会话检索 + 可选外部 provider**"，**没有官方"五层架构"提法**。⚠️ 存疑（见第 3 节）。

---

## 3. ⚠️ 未证实 / 存疑 / 已纠正的说法

| # | 网传草稿的说法 | 核查结果 | 证据 |
|---|---|---|---|
| 1 | **"Hicortex 插件"** | ❌ **查无此物**。全库/全网无 "Hicortex" 这一产品。相关联想是若干叫 **"Cortex"** 的东西（如 Prem Cortex 记忆层、YantrikDB 的 OpenClaw 插件 Cortex、sopaco/cortex-mem），但**都不是 "Hicortex"**，功能也不同。**高度怀疑是网页 AI 编的**，不得写入任何材料。 | 多轮网络检索 + Palimpsest库 grep 均无 "Hicortex" |
| 2 | **"Hermes 五层记忆架构"** | ⚠️ **无官方依据**。官方文档未见"五层"表述；Hermes 实际是"内置两文件 + 技能 + 会话检索 + 可选外部 provider"。**不采用此说法。** | hermes-agent.nousresearch.com/docs/user-guide/features/memory |
| 3 | **Hermes 主仓库 star 数（"220k+"）** | ✅ **基本属实**：`NousResearch/hermes-agent` = **252,193**（2026-10-09 21:22，GitHub API）。早期草稿给的"220k 不可信"判断需更新为"当前约 25.2 万，属实"。 | GitHub API |
| 4 | **nanobot "4000 行"** | ⚠️ **是早期口号非当前事实**。官方自称"~4000 行核心"，但当前 `nanobot/agent/` 核心 ≈3 万行（55 文件 / ~960KB）。**引用须加注。** | GitHub 仓库文件树，2026-10-09 |
| 5 | **"0.95^(days/30) 遗忘曲线"（针对 Palimpsest）** | ⚠️ 见 B线记录：代码里 `0.95` 是近重复阈值，**不是遗忘曲线**。（与本 A 线对象无关，此处仅留索引） | B线文档 |
| 6 | "Codex 是否开源" | ✅ **CLI 已开源**（Apache-2.0，Rust）；但**云版（Codex Web）闭源**、托管在 OpenAI 沙箱。表述须区分"CLI 开源 / 云版闭源"。 | github.com/openai/codex README + GitHub API |
| 7 | "Claude Code 是否开源" | ⚠️ **仓库公开但无 license 文件**（非标准开源）；其沙箱运行时以开源 preview 发布。**不能简单说"Claude Code 开源"**。 | GitHub API `license=null` + `/license` 404；anthropic.com/engineering/claude-code-sandboxing |
| 8 | "Claude Code 有没有 Computer Use" | ✅ **有**：CLI/Desktop 里可操控你的真实桌面，按应用弹权限。 | code.claude.com/docs/en/sandboxing |
| 9 | 社区传的 Claude Code "KAIROS 常驻守护进程" | ⚠️ **源自泄露源码，未经官方证实**，不写入材料。 | 社区讨论（milvus.io/blog、Reddit） |

---

## 4. 对产品叙事直接可用的话（每条可核验）

1. **"主流 Agent 都有记忆，但都是'便签式/文件式'的本地 Markdown"**——Codex(`~/.codex/memories/`)、Claude Code(`~/.claude/projects/*/memory/`)、OpenClaw(`MEMORY.md`+daily notes)、nanobot(`MEMORY.md`+`history.jsonl`)、Hermes(`MEMORY.md`+`USER.md`) —— 五家全部以 **纯文件**为主，**无一内置向量库/RAG**（nanobot 尤甚）。
2. **"记忆绑死宿主"**——Codex 的记忆只服务 Codex CLI、Claude Code 的只服务自己的项目、`CLAUDE.md`/`AGENTS.md` 是给"这项目怎么改代码"用的指令，**不是可复用的跨 Agent 知识层**。
3. **"缺冲突消解与版本链"**——各家要么"覆盖/更新"（Codex `memory_summary.md` 合并、Claude Code Dreams 清理矛盾、OpenClaw supersession key 让新观察取代旧的），要么"追加"（nanobot `history.jsonl`）；**没有一家提供"新事实覆盖旧事实但旧迹可溯的版本链（REVISED_BY）"**——这正是 Palimpsest 的差异点。
4. **"分层共识但实现各异"**——五家都认同"分层"（指令/常驻/情景/预测），但 Hermes/Claude Code 用**固定字符预算**（2200/1375、200 行/25KB）硬控上下文，OpenClaw/nanobot 用**后台整理（Dreaming/Dream）**晋升——**都是启发式，没有"检索质量可量化评测"**。

> **Palimpsest 的差异点**（这些是本项目里**代码可验证**的，非声称）：本地优先单进程嵌入式存储（TriviumDB）+ 语义向量/FTS5/图谱**混合检索**（RRF 融合）+ **冲突检测分级 + REVISED_BY 版本链 + 相似合并**（记忆治理）+ 宿主无关（独立核心 + lifecycle 协议）。→ 与 A 线五家形成直接对照。

---

## 5. 来源清单（均为一手：官方仓库 / 官方文档 / GitHub API）

**Codex**
- github.com/openai/codex（README + GitHub API：Apache-2.0/Rust/128,363★，2026-10-09）
- developers.openai.com/codex/guides/agents-md（AGENTS.md 分层）
- developers.openai.com/api/docs/guides/agents/sandboxes（沙箱记忆文件）
- learn.chatgpt.com/docs/customization/memories
- mem0.ai/library/coding-agents/how-memory-works-in-codex-cli（`~/.codex/memories/` 文件清单）

**Claude Code**
- code.claude.com/docs/en/memory（四作用域 + Auto Memory + 200行/25KB）
- code.claude.com/docs/en/plugins/overview（插件体系）
- code.claude.com/docs/en/sandboxing（沙箱 + Computer Use）
- platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
- anthropic.com/engineering/claude-code-sandboxing（沙箱开源 preview）
- GitHub API：anthropics/claude-code（149,825★，license=null，2026-10-09）
- vectorize.io/articles/claude-code-memory（Dreams 整合）

**OpenClaw**
- github.com/openclaw/openclaw（README + GitHub API：MIT/TS/391,531★，2026-10-09）
- 仓库 docs/concepts/memory-architecture.md（分层/provenance/Dreaming/双召回道）
- 仓库 docs/concepts/memory.md（文件清单 + 导入 Codex/Claude Code/Hermes）
- 仓库 docs/concepts/architecture.md（Gateway/WebSocket 架构）

**nanobot**
- github.com/HKUDS/nanobot（README + GitHub API：MIT/Python/48,897★，2026-10-09）
- 仓库 docs/memory.md（分层文件 + Consolidator/Dream + history.jsonl）
- 仓库 docs/architecture.md（AgentLoop/AgentRunner 拆分）
- GitHub Discussion #1609 / #566（"~4000 lines core" 说法；"No vector DB"）
- GitHub 仓库文件树（`nanobot/agent/` ≈3 万行，2026-10-09）

**Hermes Agent**
- hermes-agent.nousresearch.com/docs（官网）
- hermes-agent.nousresearch.com/docs/user-guide/features/memory（内置记忆 2200/1375）
- hermes-agent.nousresearch.com/docs/user-guide/features/skills（skill_manage 学习闭环 + agentskills.io）
- hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers（外部 provider）
- hermes-agent.nousresearch.com/llms.txt（官方链接索引）
- github.com/NousResearch/hermes-agent（GitHub API：MIT/Python/252,193★，2026-10-09）

**共通标准**
- agentsmd/agents.md（AGENTS.md 开放标准，MIT，24,843★，2026-10-09）
- agentskills.io/specification（技能开放标准）

---

*调研：小九（A线）｜2026-10-09｜只读调研，未改动任何代码*
