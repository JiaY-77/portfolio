# 主流记忆工具对比调研 — Mem0 / Letta / Cognee / Zep

> 一手调研，2026-10-09。每条结论标来源 URL，所有数字标注查询时间。

---

## 一、调研目的

为产品立项做方向判断：市面上的现成方案能否直接满足需求？网上流传的草稿有**未经核实的说法**，故必须一手查证。**核心要回答的问题**：
> 市面上有没有「本地优先 + 混合检索 + 记忆治理 + 跨 Agent 复用」四者兼备的记忆方案？

---

## 二、四个主流记忆方案对比

### 1. Mem0

| 维度 | 结论 | 来源 |
|---|---|---|
| 定位 | 智能记忆层，夹在应用和 LLM 之间，简单 API | mem0.ai/library |
| 架构 | **混合三存储**：向量库 + 知识图谱 + KV 存储，并行写入、并行检索后融合 | YouTube: Hybrid Memory Systems；arxiv 2504.19413 |
| 记忆模型 | LLM 提取事实 → 评估 → 更新（"只添加不覆盖"式，去重+实体链接） | arxiv 2504.19413 |
| 图记忆 | Mem0g：有向标记图，节点=实体、边=关系；支持 Neo4j/FalkorDB | arxiv 2504.19413 |
| **能否本地部署** | ✅ **能**。`pip install mem0ai` 本地库模式（默认本地 Qdrant + SQLite）；或 Docker 自托管服务（FastAPI + pgvector + Neo4j 三容器） | docs.mem0.ai/open-source/overview；mem0.ai/library/integrations/self-host-mem0-docker |
| 自托管门槛 | 服务模式=三容器；**默认调 OpenAI**，要纯本地需换 Ollama 本地模型 | 同上 |
| 规模 | 官网称社区 90,000+ / 100k+ 开发者 | mem0.ai/guide；mem0.ai/library |
| 许可 | Apache-2.0（自托管栈） | docs.mem0.ai |

### 2. Letta（前身 MemGPT）

| 维度 | 结论 | 来源 |
|---|---|---|
| 定位 | **不是纯记忆层，而是带记忆的 Agent 运行时**（agent loop + 工具 + 记忆一体） | hindsight.vectorize.io/blog |
| 血统 | MemGPT 论文（2023，UC Berkeley/Stanford）——"LLM as OS"范式 | letta.com/blog/memory-blocks |
| 记忆模型 | **两层**：① Core Memory（Memory Blocks，常驻上下文，agent 用工具自编辑：`memory_replace`/`memory_insert`/`memory_rethink`）② Archival Memory（长尾，out-of-context，可检索） | letta.com/blog/memory-blocks；leoniemonigatti.com/blog/memgpt |
| 关键机制 | 自编辑记忆：agent 自己改自己的上下文窗口 | graphlit.com/vs/letta |
| **能否本地部署** | ✅ 能。Docker 自托管（`letta/letta:latest`），数据持久化到本地 Postgres | docs.letta.com/self-hosting；docs.letta.com/v1-sdk/docker |
| 短板 | 需要 LLM 调用管理记忆（token 开销）；架构重（整个 agent runtime） | 综合 |
| 规模 | ~24.2K stars（第三方统计，2026-08） | hindsight.vectorize.io/blog |
| 许可 | Apache-2.0 | 同上 |

### 3. Cognee

| 维度 | 结论 | 来源 |
|---|---|---|
| 定位 | 开源 AI 记忆平台，主打**自托管知识图谱引擎** | github.com/topoteretes/cognee |
| 架构 | 关系库（文档/块/来源）+ 向量 + 图；管线 Add → Cognify → Memify → Search | cognee.ai 博客 |
| API | `remember` / `recall` / `improve` / `forget` | docs.cognee.ai |
| **无 LLM 也能跑** | ✅ 文本摄入/检索/会话存储无需 LLM；LLM 相关增强阶段自动跳过 | github.com/topoteretes/cognee |
| **能否本地部署** | ✅ 能，自托管（DuckDB/LanceDB 轻量起步，可本地模型） | github.com/topoteretes/cognee |
| 短板 | 以服务端知识图谱为核心；本地部署有门槛 | 综合 |
| 定位语 | 官网自述"self-hosted knowledge graph engine" | 同上 |

### 4. Zep / Graphiti

| 维度 | 结论 | 来源 |
|---|---|---|
| 定位 | 时序知识图谱记忆（temporal knowledge graph）；Zep=平台，Graphiti=开源引擎 | getzep.com/ai-agents/temporal-knowledge-graph |
| 记忆模型 | **双时间**（bi-temporal）：每个事实记 valid time（世界真值时间）+ ingestion time；**事实失效而非删除** | 同上 |
| 架构 | 图更新式存储对话 episode；实体带有效期窗口；混合检索（语义 embedding + BM25 + 图遍历） | atlan.com/know/zep-vs-mem0 |
| **能否自托管** | ⚠️ **只能自托管 Graphiti 引擎**。**Zep Community Edition 已于 2025 年 4 月弃用**（2026-02 进一步退役功能）；完整 Zep 应用栈不能自托管 | getzep.com/blog（官方公告）；atlan.com；kronvex.io |
| 自托管门槛 | 需 Graphiti + 独立图库（Neo4j/FalkorDB/Kuzu），**至少三套系统** | atlan.com |
| 规模 | Graphiti ~24K stars（另说 20,000+/29.8K，视时间）；Apache-2.0 | atlan.com；getzep.com；hindsight.vectorize.io |
| 官方 Docker | `zepai/graphiti:latest` 已 6 个月未更新（v0.10 vs v0.22），issue #1030 | github.com/getzep/graphiti/issues/1030 |

---

## 三、关键结论：空白究竟在哪

**四家都有"某种形式"的本地化路径**，但**没有一家同时满足本项目的四个条件**：

| 方案 | 本地优先 | 混合检索 | 记忆治理（版本链/冲突消解） | 跨 Agent 复用 | 备注 |
|---|---|---|---|---|---|
| Mem0 | ✅ 可自托管 | ✅ 向量+图+KV | ⚠️ 去重/更新，非版本链 | ⚠️ 主要是库/服务 | 默认调云端 LLM |
| Letta | ✅ 可自托管 | ⚠️ 块+归档 | ⚠️ 自编辑，无冲突消解 | ❌ 绑定自己 runtime | 重（整个 agent 框架） |
| Cognee | ✅ 可自托管 | ✅ 向量+图 | ⚠️ 有 improve 阶段 | ⚠️ | 服务端图为核心 |
| Zep/Graphiti | ⚠️ 仅引擎 | ✅ 语义+BM25+图 | ✅ 时序失效（最接近） | ⚠️ | **CE 已弃用** |

**精确定位空白**（可核验的说法）：
> 主流方案多为**服务端/云端架构**（默认依赖外部 LLM 或远程服务）；即便支持自托管，也多为**多组件重量级栈**（Mem0 三容器、Letta 整个 runtime、Zep 需 Graphiti+外部图库）。
> **没有一个方案**同时做到：**单机嵌入式 + 混合检索 + 冲突消解/版本链 + 宿主无关（可接任意 Agent）**。

**Palimpsest 的差异点**（以下均可由代码验证，非声称）：
- 本地优先：单进程嵌入式存储（TriviumDB），进程内读写，零外部依赖
- 混合检索：语义向量 + FTS5 全文 + 图谱扩散，RRF 融合（`core/` 可验）
- 记忆治理：冲突检测分级 + `REVISED_BY` 版本链（`core/conflict.py`）+ 相似合并（`core/consolidator.py`）
- 宿主无关：从 Hermes 插件下沉为独立核心，暴露 lifecycle 协议（`core/strategy.py`）

---

## 四、⚠️ 待核实 / 存疑项

| 项 | 状态 |
|---|---|
| 网上草稿提到的 **"Hicortex 插件"** | ❌ Palimpsest仓库 grep 全库**查无此物**；网上也未搜到。**高度怀疑是那个网页 AI 编的**——不得写入任何材料 |
| 草稿说的 **"Hermes 五层记忆架构"** | ⚠️ 未找到官方"五层架构"说法，需核实（Hermes 有 Memory/Skill/Nudge 等机制，但"五层"表述无据） |
| 草稿说的 **"0.95^(days/30) 遗忘曲线"** | ⚠️ 代码里 `0.95` 是 `NEAR_DUP_THRESHOLD`（近似重复阈值），**不是遗忘曲线**；Palimpsest的衰减机制另有实现，需单独确认 |
| Hermes 主仓库 star 数 | ⚠️ 未核实到可信数字（一处说 220k+ 不可信）；self-evolution 子项目 5.5k |
| nanobot "4000 行" | ⚠️ 官方 roadmap 说 **3500 行核心**（非 4000）；star 46k→48.5k（2026-09） |

---

## 五、来源清单

- Mem0 架构：arxiv.org/html/2504.19413v1
- Mem0 自托管：docs.mem0.ai/open-source/overview；mem0.ai/library/integrations/self-host-mem0-docker
- Letta 记忆块：letta.com/blog/memory-blocks；docs.letta.com/self-hosting
- Cognee：github.com/topoteretes/cognee；docs.cognee.ai
- Zep/Graphiti：getzep.com/ai-agents/temporal-knowledge-graph；getzep.com/blog（CE 弃用公告）；atlan.com/know/zep-vs-mem0
- 第三方横评：hindsight.vectorize.io/blog/2026/08/11/open-source-agent-memory-systems
- Palimpsest本地赛道（已有）：知识库《Palimpsest 阶段四方案（记忆赛道深挖）》——pi-hermes-memory / memorix
