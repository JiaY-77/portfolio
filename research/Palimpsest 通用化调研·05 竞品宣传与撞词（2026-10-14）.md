# 记忆赛道竞品：打包宣传方式与叙事占位

调研日期：2026-10-14
调研对象：mem0、Zep / Graphiti、Letta（原 MemGPT）、Cognee、Memobase、Supermemory、LangMem
方法：只读官方 README / 官网 / 官方文档，"定位原文"逐字引用并附 URL；推断处均标注「[推断]」；未查到的一律写「未能确认」。

---

## 0. 一句话速览（官方定位原文）

| 项目 | 定位原文（逐字） | 出处 |
|---|---|---|
| mem0 | "The Memory Layer for AI Agents - Drop-in memory infrastructure for AI agents and apps. Context that persists. Built for production." | https://github.com/mem0ai/mem0 |
| Zep | "Agent memory, at enterprise scale. Memory of users, the business, and work done. Managed, governed, and served at scale." | https://www.getzep.com |
| Graphiti（Zep 开源引擎） | "Graphiti is a framework for building and querying temporal context graphs for AI agents."；副标题 "A Framework for Building Temporal Knowledge Graphs" | https://github.com/getzep/graphiti |
| Letta | "Build stateful agents with memory that can learn and improve over time."（README 首句）；官网 "Letta is an AI research lab in San Francisco building machines that learn." | https://github.com/letta-ai/letta ；https://www.letta.com |
| Cognee | "Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long-term memory across sessions with a self-hosted knowledge graph engine."（GitHub About）；README 首句 "Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long-term memory with small models for free" | https://github.com/topoteretes/cognee |
| Memobase | "User Profile-Based Long-Term Memory for AI Chatbot Applications."（仓库 About）；正文 "Memobase is a **user profile-based memory system** designed to bring long-term user memory to your LLM applications." | https://github.com/memodb-io/memobase |
| Supermemory | "State-of-the-art memory and context engine for AI."（README 正文首句）；仓库 About "Memory and context engine + app that is extremely fast, scalable, and can be run fully locally. The Memory API for the AI era." | https://github.com/supermemoryai/supermemory |
| LangMem | "LangMem helps agents learn and adapt from their interactions over time."（README 首句） | https://github.com/langchain-ai/langmem |

补充：Memobase 新版文档站（memobase.ai/docs）已改为 toC 工具口径——"Memobase provides a persistent, shared memory layer for AI agents. It ensures context is never lost and intelligence is unified across all your tools."（https://memobase.ai/docs）。同一家产品出现「toC 工具站 / toD 开源仓库」两套定位，[推断] 是双线打法。

---

## 1. 对比表

| 项目 | 定位原文 | 集成矩阵规模 | 分发渠道 | 已占叙事点 |
|---|---|---|---|---|
| **mem0** | "The Memory Layer for AI Agents - Drop-in memory infrastructure for AI agents and apps."（https://github.com/mem0ai/mem0） | 官方 Integrations 页列出 **20+** 条目：AgentOps / Camel AI / LangChain / LlamaIndex / AutoGen / CrewAI / LangGraph / Vercel AI SDK / LangChain Tools / Dify / Livekit / ElevenLabs / Pipecat / Agno / Respan / Raycast / Mastra / OpenAI Agents SDK / Google ADK / Flowise / AWS Bedrock / ChatDev（https://docs.mem0.ai/integrations）。Agent Plugins 覆盖 Claude Code / Cursor / Codex / Kimi（仓库含 `.claude-plugin` `.cursor-plugin` `.codex-plugin` `.kimi-plugin` `marketplace.json`）。明确举「Universal Integration」大旗："Use Mem0 MCP for a standardized protocol that works with ANY AI client."（https://docs.mem0.ai/integrations） | PyPI `mem0ai`、npm `mem0ai`、MCP、**插件市场**（marketplace.json + 各 IDE 插件目录）、Docker 自托管服务端、云托管（app.mem0.ai 免费 API key，agent 可自助 mint key）、Chrome 扩展 | 「memory layer 记忆层」、「drop-in / 生产就绪 production」、「token 效率与 benchmark」（LoCoMo 92.5 / LongMemEval 94.4，https://github.com/mem0ai/mem0）、「多级记忆 User/Session/Agent」、「temporal reasoning 时间感知检索」（2026-04 新算法条目明确写 "Temporal Reasoning -- time-aware retrieval"，但**不是** bi-temporal 有效期模型）、「不需要改 pipeline」。**未占**：本地优先、显式版本链、图谱为核心（图谱在 Pro 档，https://vectorize.io/articles/mem0-vs-zep） |
| **Zep**（+ 开源引擎 Graphiti） | Zep 官网："Agent memory, at enterprise scale."（https://www.getzep.com）。Graphiti："A framework for building and querying temporal context graphs for AI agents."（https://github.com/getzep/graphiti） | 官方 `getzep/zep` 仓库的集成包按框架组织：**Python** Google ADK / Microsoft Agent Framework / Microsoft AutoGen / AG2 / CrewAI / LangGraph / LiveKit / Pydantic AI / Strands Agents；**TypeScript** Google ADK / Mastra / Vercel AI SDK；**Go** Google ADK（https://github.com/getzep/zep，integrations/ 目录说明）。另有 Claude Desktop/Cowork 与 ChatGPT Work 的 agent plugin、Graphiti MCP Server（仓库含 `mcp_server/`）。用「Works with any agent framework, or none.」 | PyPI（graphiti-core、zep 相关 SDK）、Docker（Graphiti 自带 Dockerfile/docker-compose，第三方有 FalkorDB/Neo4j 托管）、Graphiti MCP Server（可经 MCP registry 分发，Smithery 类）、云托管 Zep Cloud（三种部署：Cloud / BYOK / BYOC，含 SOC 2 Type II、HIPAA BAA） | 「temporal knowledge graph 时序知识图谱」**（最强占位）**、「validity window / 有效期窗口（valid_from / valid_to / invalid_at）」（https://vectorize.io/articles/mem0-vs-zep）、「provenance 溯源到源 episode」、「Context Graph / Context Lake」、「治理 governance：access control / retention / audit」、「Sub-200ms 检索延迟」、「enterprise scale 企业级」、「episodic / semantic entity / community 三层子图」。Graphiti 仓库明确写 telemetry 默认开启（可关）——**不是**全本地叙事。**未占**：本地优先、显式 declared 取代（它是自动失效旧事实） |
| **Letta**（原 MemGPT） | "Build stateful agents with memory that can learn and improve over time."（https://github.com/letta-ai/letta）；官网 "machines that learn"，标语 "Context is selfhood."（https://www.letta.com） | **不自称记忆层**，是 agent 平台/运行时：Agent SDK（TypeScript）、桌面 app（macOS/Windows/Linux）、浏览器 chat.letta.com（含移动端）、渠道 Slack / Telegram / Discord（https://github.com/letta-ai/letta）。MCP 双向（Letta agent 可作 MCP client 调外部 server；社区有 letta-mcp-server 反向暴露，https://github.com/oculairmedia/letta-mcp-server）。集成叙事是「agent 运行时 + 多渠道」，**不是**「一个记忆层接所有 agent」 | npm `@letta-ai/letta-code`、Letta Cloud（记忆/身份/会话跨设备）、App Server 自托管、Docker（社区 Railway 模板）、官方 MCP 生态 | 「stateful agents 有状态智能体」、「self-editing memory 自我编辑记忆」（MemGPT 血统，core memory blocks + archival memory）、「**git-based versioning / Context Repositories**」——2026-02 博客明确写 "Context Repositories, a rebuild of how memory works ... based on programmatic context management and git-based versioning"，每次记忆改动自动 versioned 带 commit message（https://www.letta.com/blog/context-repositories）、「continual learning 持续学习」、「sleep-time compute / 离线学习」、「virtual context / LLM as OS」。**注意：这家的"版本"是 git 式文件版本，不是事实级 bi-temporal 取代链**——叙事上离 Palimpsest 的「覆而不失、改写可溯」最近，但机制不同。[推断] 是本次盘点中与 Palimpsest 叙事最易混淆的一家 |
| **Cognee** | GitHub About："Cognee is the open-source AI memory platform for agents. Give your AI agents persistent long-term memory across sessions with a self-hosted knowledge graph engine."（https://github.com/topoteretes/cognee） | 1.0 公告明确列「一个安装接一堆客户端」：Claude Desktop / Claude Code / Cursor / Codex / OpenClaw / Windsurf / Gemini CLI / Cline / plain REST / any MCP-compatible agent（https://www.cognee.ai/cognee-1-0-announcement）。文档 Client Integrations 另列 Continue、Python Agent，并说 "works with any client that speaks MCP — not only the ones listed"，点名 OpenCode / Windsurf / Zed（https://docs.cognee.ai/cognee-mcp/integrations）。口号 "One brain. Every agent." / "One brain. Every MCP agent reads and writes the same memory." | `pip install cognee`（PyPI）、**Docker**（`cognee/cognee-mcp:main`，官方 MCP 页给 docker run 命令）、npm/TypeScript SDK、Rust SDK（edge/on-device）、CLI、托管 Cloud（platform.cognee.ai 免费 key）、一键部署配置（Modal/Railway/Fly.io/Render/Daytona） | 「**open-source AI memory platform**」、「self-hosted knowledge graph engine」、「remember / recall / improve / forget」四动词 API、**「本地跑、免费、不用 OpenAI key」（README 已改为 local-first 开场："Start locally for free without an OpenAI API key"）**、「**Local AI Memory：Keeping Agent Memory Off the Cloud**」整篇长文主张本地四级（local storage / local retrieval / local memory processing / fully offline）（https://www.cognee.ai/local-ai-memory）、「数据可携带 COGX 开放格式」、「on-prem / 数据主权」、「memory that gets better with use (improve)」、「BEAM benchmark 79%」。**注意**：Cognee 现在是本次盘点里对「本地优先」口号最用力的一家，会与 Palimpsest 直接撞词 |
| **Memobase** | "User Profile-Based Long-Term Memory for AI Chatbot Applications."（https://github.com/memodb-io/memobase）；docs 站："Memobase provides a persistent, shared memory layer for AI agents. It ensures context is never lost and intelligence is unified across all your tools."（https://memobase.ai/docs） | 集成相对少：MCP-native server（https://mcp.memobase.ai）、OpenAI Actions（ChatGPT Custom GPT）、Claude Code HTTP Hooks（自动 session 追踪）、SDK（Python / Node / Go / Deno JSR）、MCP badge 在 README（https://github.com/memodb-io/memobase）。**不提**「接所有 agent」的宏大叙事，偏「toC 应用的用户记忆」 | PyPI `memobase`、npm `@memobase/memobase`、jsr、Go pkg、**Docker**（`ghcr.io/memodb-io/memobase`，docker-compose 一键）、云托管（app.memobase.io，**免费测试 token + 可视化 Dashboard/Playground/Inspector**）、GitHub Container Registry | 「**Memory for User, not Agent**」（核心反叙事：给终端用户建画像，不是给 agent 存记忆）、「User Profile / **Time-aware Memory** 用户事件时间线」、「Controllable Memory 可设计 profile 结构」、「SOTA on LOCOMO（85% temporal reasoning，比 Mem0/Zep 高）」（https://x.com/memobase_io）、「性能/成本/延迟三指标」、「Knowledge Graph（docs 站新增）」、「Hybrid Search（语义 + BM25）」、「Memory Consolidation / Context Reconstruction」、「**被夸大的隐私承诺做反面教材**：docs 站主动写明 "it is not zero-knowledge ... Memobase can process your data"」。**未占**：显式取代链、本地优先（它是云 vault） |
| **Supermemory** | "State-of-the-art memory and context engine for AI."（https://github.com/supermemoryai/supermemory）；仓库 About "The Memory API for the AI era."；MCP 页 "One endpoint, every assistant remembers." | Plugins 内置：Claude Code / Muse Code / Cursor / Codex / OpenCode / OpenClaw / **Hermes**（README 原文："Supermemory comes built with plugins for Claude Code, Muse Code, Cursor, Codex, OpenCode, OpenClaw, and Hermes."）。MCP 客户端：Claude Desktop/Code、ChatGPT、Cursor 及任意 MCP client（https://supermemory.ai/mcp）。Connectors：Google Drive / Gmail / Notion / OneDrive / GitHub。**明确用「one memory layer / 一个记忆层接所有」叙事**："One memory layer, carried between the assistants you already use." | npm `supermemory`、PyPI `supermemory`、**MCP registry（Smithery、Gamut、Composio 都有条目）**、远程托管 MCP endpoint（https://mcp.supermemory.ai/mcp，OAuth）、**自托管单二进制（`curl https://supermemory.ai/install | bash` / `npx supermemory local`，无 Docker 无 DB）**、云托管（console.supermemory.ai）、`skills/` 目录 + 有 Claude Skills 市场上架记录 | 「**#1 on every major AI memory benchmark** LongMemEval / LoCoMo / ConvoMem」（https://github.com/supermemoryai/supermemory）、「95% Recall@15 + 99.4% context reduction」、「**can be run fully locally**（About 原文）+ 单二进制 + 零配置 + 全离线（Ollama）」、「automatic forgetting 自动遗忘 / 矛盾自动解决」、「Memory is not RAG」（"I just moved to SF" supersedes "I live in NYC"，https://supermemory.ai/docs/concepts/memory-vs-rag）、「User Profiles ~50ms」、「Connectors 接一堆 SaaS」、「**MemoryBench 开源基准 + 让竞品来跑分**」。**关键**：它既占了「本地优先」的一半，又占了「自动取代/自动遗忘」——但机制是**自动**矛盾消解，不是调用方**显式声明**取代 |
| **LangMem** | "LangMem helps agents learn and adapt from their interactions over time."（https://github.com/langchain-ai/langmem） | **集成矩阵最窄**：本质是 LangChain 官方 SDK，核心 = 「任何 storage system 都能用」+ "Native integration with LangGraph's Long-term Memory Store, available by default in all LangGraph Platform deployments"（README）。不自称记忆层、不做「接所有 agent」，是 LangChain 生态内的一个包 | PyPI `langmem`（`pip install -U langmem`）、LangGraph Platform 内置、托管服务（LangChain 博客提到 managed service 免费额度，https://www.langchain.com/blog/langmem-sdk-launch）。无独立 MCP / Docker / 插件市场条目 | 「semantic / episodic / procedural 三种记忆（认知心理学框架）」、「hot path vs background 两种路径」、「**prompt optimization / 让 agent 通过 prompt refinement 改自己行为**」（独特点）、「namespace 做用户隔离」、「Learning/RSI」。**未占**：本地优先、知识图谱、时间有效性、版本链——几乎不碰这些叙事 |

---

## 2. 谁占了哪块（逐点归属）

| 叙事点 | 谁明确占了 | 占位强度与备注 |
|---|---|---|
| **本地优先 / 数据不出本机** | Cognee（最用力，整篇长文 + README 改成 local-first 开场）、Supermemory（"can be run fully locally" + 单二进制 + 全离线）、Mem0/Graphiti（开源可自托管，但默认云/自动遥测，叙事重点不在本地） | **半空半满**。Cognee 与 Supermemory 都用了 "local"，但两者都仍以云托管为主推路线、本地是「也支持」。[推断] 「本地优先（local-first）作为**默认形态**」这块没有被哪家真正当成品牌核心占死 |
| **版本链 / 显式取代** | Letta（git-based versioning，Context Repositories）、Supermemory（automatic forgetting + contradiction resolution）、Zep/Graphiti（validity window 自动失效） | **「版本/时间有效性」被占了，但「显式声明取代」这个精确表述没人占**。Letta 是文件级 git 版本；Zep 是自动 temporal invalidation；Supermemory 是自动矛盾消解。**没有一家**做「调用方传 replaces 声明取代、默认不因相似度自动取代」——这正是 Palimpsest 的精确空档 |
| **事实时效 / bi-temporal** | Zep / Graphiti（最正统：valid_from / valid_to / invalid_at、"when it became true, when it was superseded"）、Mem0（"Temporal Reasoning"，但只是 time-aware retrieval，非有效期模型）、Memobase（Time-aware Memory / event timeline） | **Zep 占得最深**。它是这块的学术 + 产品双权威（arXiv 2501.13956）。Palimpsest 若直接喊「时序知识图谱」会撞 Zep，需要用不同的词（见下） |
| **知识图谱** | Zep/Graphiti（context graph 为核心）、Cognee（self-hosted knowledge graph engine）、Memobase（docs 站 Knowledge Graph）、Mem0（图谱在 Pro 档，非核心） | **高度拥挤**。「知识图谱」作为卖点已饱和，且默认理解为**自动抽取**实体关系（LLM 抽取）。Palimpsest 的图谱是**加权 + 支持显式声明的关系/边**，是个差异点但必须先讲清「不是自动抽取的那种」 |
| **集成矩阵 / 一个记忆层接所有 agent** | mem0（20+、Universal Integration、"ANY AI client"）、Cognee（"One brain. Every agent."）、Supermemory（"One endpoint, every assistant"、plugins 含 Hermes）、Zep（框架优先，Python/TS/Go 三语言）、Memobase（MCP + Actions + Hooks） | **最拥挤的战场，且已被占成红海**。Palimpsest 目前只有 Hermes 一个宿主，**硬拼集成数量必输**，不能在这块打 |
| **benchmark 分数** | mem0（LoCoMo 92.5）、Supermemory（三个第一）、Zep（LoCoMo 94.7）、Cognee（BEAM 79%）、Memobase（LOCOMO 85% temporal） | 已被占满，[推断] 新项目硬报分会被质疑口径，且 Palimpsest 未必有可比数据 |
| **企业级治理 / 合规** | Zep（access control / retention / audit / BYAC / SOC 2 / HIPAA / provenance）、mem0（生产就绪） | 已被 Zep 占死，Palimpsest 不必碰 |
| **敏感信息扫描 / 写入前拒写** | **未查到任何一家以此为宣传点** | **空档**。Memobase 甚至反其道自曝「not zero-knowledge」，说明这块没人做成卖点 |
| **中文支持 / 中文子串检索** | **未查到任何一家在宣传里提中文** | **空档**（且是真实差异化技术点：FTS5 trigram 支持中文子串）。注意 Cognee README 有多语言版本（含中文），但那是文档翻译，不是检索能力 |
| **覆而不失 / 改写可溯** | 拆分看：Zep 有 "old facts stay as history" + provenance；Letta 有 git history | 概念被零散碰过，但**没有一家用「覆而不失」这组词整体打包** |

---

## 3. 结论：可占的差异化空档 + 建议宣传口径

### 3.1 可占的空档（按优先级）

1. **「显式取代」+「覆而不失」的精确组合** —— 全场无人占。
   Zep 自动失效、Supermemory 自动消解矛盾、Letta 是 git 文件版本；**没有一家**做「调用方显式传 replaces 才取代，默认不因相似度自动改写」。这是一个可防守的机制差异，而且叙事上很干净：**记忆的改写权在你手上，不在相似度算法手上**。

2. **「本地优先」作为默认形态（不是可选项）** —— 半空档。
   Cognee/Supermemory 都说 local，但都先推云。Palimpsest 可以占「不是也支持本地，是**只跑在本地、没有云这一层**」——顺带接住「敏感信息写入即拒」这条没人用的点，把「本地」从部署方式升格为**安全承诺**。

3. **敏感信息写入前扫描 / 拒写** —— 完全空档。
   竞品都在讲「记住多少」，没人讲「什么坚决不记」。反向叙事，记忆赛道天然容易被质疑「什么都存 = 隐私灾难」，这条正好接住。

4. **中文一等公民** —— 完全空档。
   全场没有一家宣传中文检索能力。Palimpsest 的 FTS5 trigram 中文子串是真实能力，可作为面向中文用户/中文知识库的切入。

5. **多接口共用一核（MCP × REST × CLI 同源）** —— 弱空档。
   mem0 提 MCP、Zep 提 SDK，但「三种接口同一个核心、能力完全对等」这个说法没人讲清楚。（这条略弱，优先级最低。）

### 3.2 建议的宣传口径（一句话）

主口径：

> **Palimpsest —— 你的记忆，只在你机器上，且改写权归你：本地优先的 AI 记忆底座，覆而不失、改写可溯，写入前先过敏感信息扫描。**

更短版本（README 副标题级）：

> **Local-first memory base for AI — nothing leaves your machine, and nothing gets rewritten unless you say so.**

一句话里的三段各自对应一个空档：①「只在你机器上」= 本地优先 + 不提云；②「改写权归你 / 覆而不失」= 显式取代；③「写入前敏感扫描」= 别人都没做的反向叙事。

### 3.3 建议避开的撞词（已被占用）

- **memory layer / 记忆层** —— mem0 占（"The Memory Layer for AI Agents"）。用它做核心词会被当成 mem0 的模仿者。
- **temporal knowledge graph / 时序知识图谱** —— Zep/Graphiti 占死，且有 arXiv 背书。要讲时间维度，改用**「事实有效期 / 覆而不失 / 版本链」**这类机制词，不用「时序知识图谱」。
- **One brain, every agent / 一个记忆层接所有 agent** —— Cognee、Supermemory、mem0 三家都用。Palimpsest 眼下只有 Hermes 一个宿主，**不要在这块正面对标**，改讲「接口能力对等（MCP/REST/CLI 同一核心）」，把比较维度从"广度"换到"一致性"。
- **local-first 这个词本身可用，但必须加限定** —— "local" 已被 Cognee/Supermemory 使用（作为可选项）。建议用 **"only runs on your machine" / 「只跑在本机」/「无云层」** 来拉开距离，而不是笼统的 local-first。
- **automatic forgetting / 自动遗忘** —— Supermemory 占。Palimpsest 恰恰**反对**自动取代，可以反向用它：「不自动遗忘，改写要你签字」。
- **stateful agents / 有状态智能体** —— Letta 占，且它是 agent 平台不是记忆层，不要用。
- **benchmark 排行** —— 已被四家占满，且无可靠可比数据，避免主动挑起比分数。

### 3.4 需要提请注意的风险

- [推断] **Letta 的「Context Repositories = git-based versioning」与 Palimpsest 的「版本链」在宣传上最容易被混为一谈**。建议 Palimpsest 讲版本链时明确区分：「不是文件级 git 快照，是**事实级**的取代链（valid_at / invalid_at + REVISED_BY）」。
- Cognee 近期（2026-09-26）刚发了 Local AI Memory 长文，正在主动抢占本地叙事空间。若要占「本地优先」，宜尽快落笔，且用「无云层 / 数据不出机」这类它没用的强表述。
