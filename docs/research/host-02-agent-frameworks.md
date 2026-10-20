# Agent 框架的记忆抽象与换后端能力调研

> 调研对象：LangGraph（含 LangMem 与官方 Store）、AutoGen、CrewAI、LlamaIndex、Semantic Kernel、Google ADK、OpenAI Agents SDK
> 目的：评估 Palimpsest（本地优先 AI 记忆底座）接入各框架的路径与工作量
> 纪律：每条结论附 URL；区分「官方文档明确写的」与「推断」；不确定写「未能确认」
> 调研日期：2026-10-14

---

## 1. 对比表

| 框架 | 记忆抽象 | 能否换后端 | 注入钩子 | 官方支持的第三方记忆 | 接入路径 | 工作量 |
|---|---|---|---|---|---|---|
| **LangGraph**（+LangMem、Store） | 长期记忆：`BaseStore`（异步方法 `aput` / `aget` / `adelete` / `asearch` / `alist_namespaces`）；短期记忆：Checkpointer（`InMemorySaver` / `PostgresSaver`）。LangMem 建在其上：`create_manage_memory_tool` / `create_search_memory_tool` | **能，官方明确**：`docs.langchain.com`「To use a storage backend other than the built-in implementations, subclass `BaseStore` and implement its required methods」；部署侧可在 `langgraph.json` 用 `store` 键指定自定义 `BaseStore` | **有**：`create_react_agent` 的 `pre_model_hook`（Zep 官方用它做「每轮模型调用前」注入）；工具型注入（`create_search_memory_tool`）；LangMem 的 `SummarizationNode`（压缩/摘要节点） | Zep 官方集成（`create_zep_pre_model_hook` + graph-search tool）；mem0 有集成文档 | ①自定义 `BaseStore` 子类（Python）②`pre_model_hook` 直接调 Palimpsest MCP/REST ③memory tools 包装 MCP ④LangGraph 支持 MCP adapter | 中。`BaseStore` 是清晰接口，但语义检索要自己接 embedding；`pre_model_hook` 路线最省事 |
| **AutoGen** | `Memory` 协议（`autogen_core.memory.Memory`），关键方法 `add` / `query` / `update_context` / `clear` / `close` | **能，官方明确**：「You can build on the `Memory` protocol to implement more complex memory stores... overload the `add`, `query` and `update_context` methods」 | **有**：`update_context`（每步把检索结果写进 agent 的 `model_context`），官方原话「intelligently added to the context of the agent just before a specific step」 | **官方 `autogen_ext` 内置**：`Mem0Memory`（`autogen_ext.memory.mem0`）、`ChromaDBVectorMemory`、`RedisMemory`；Zep 官方也提供 AutoGen 记忆类 | 写一个继承 `Memory` 协议的类，`add` 走 Palimpsest 写入、`query` 走 Palimpsest 检索、`update_context` 注入；或纯 MCP 工具 | **低**。协议只有 5 个方法，是「最薄」的接口之一。**注意**：AutoGen 已进维护模式（见下） |
| **CrewAI** | `Memory` 类（统一记忆），配合 `StorageBackend` 协议（`crewai.memory.storage.backend`） | **能，官方明确**：「Custom backend: Implement the `StorageBackend` protocol and pass an instance to `Memory(storage=your_backend)`」 | **有**：官方说明「Before each task, the agent recalls relevant context from memory and injects it into the task prompt」；Flow 内 `self.recall()` / `self.remember()`；外部记忆经 `external_memory` 参数 | 官方文档提到 `ExternalMemory` 与 Mem0（社区/文档提及「using an external memory like Mem0 useful when sharing memory with different crews」）；第三方 `crewai-mongodb-memory` 实现其 `StorageBackend` | ①实现 `StorageBackend` 协议类，`Memory(storage=...)`；②或走 `external_memory` 参数指向 Palimpsest 包装层 | 中。`StorageBackend` 要满足 CrewAI 的复合打分/scope 树语义；只做 external_memory 则较省 |
| **LlamaIndex** | `Memory` 类（`llama_index.core.memory.Memory`），可组合 `MemoryBlock`（如 `FactExtractionMemoryBlock`）；旧接口 `BaseMemory` / `ChatMemoryBuffer`（已废弃） | **能，官方明确**：「you can typically customize memory by using an existing `BaseMemory` class, or by creating a custom one」 | **有**：`Memory` 在 agent 每轮被读取；`node_postprocessors` 可挂检索后处理 | **官方集成包**：`llama-index-memory-mem0`（`Mem0Memory`，LlamaHub 收录、官方文档有 Mem0 示例页），替换 `ChatMemoryBuffer` 即可 | 写自定义 `BaseMemory`/`MemoryBlock`，或在 agent 的 `memory=` 槽传入 Palimpsest 适配器；也可直接 MCP 工具 | 中。`Memory` 语义较丰富（token 限额、flush 到长期记忆），适配比 AutoGen 稍重 |
| **Semantic Kernel** | 现役：**Vector Store 抽象**（`Microsoft.Extensions.VectorData.Abstractions`：`VectorStore` / `VectorStoreCollection<TKey,TRecord>`）；旧版：**Memory Store**（`Microsoft.SemanticKernel.Memory.IMemoryStore`、`ISemanticTextMemory`），官方标注建议改用 Vector Store | **能**：Vector Store 是连接器模型，官方有大量 store 连接器；也可自实现 `VectorStore` / `IMemoryStore` | **间接**：SK 无「每轮前」的记忆钩子，靠 **Function/Plugin 调用**检索（把检索包成 `[KernelFunction]` 让模型调）；Filters 可做管道拦截 | 无 mem0/Zep 官方一级集成（未能确认）；走 Vector Store 生态（Azure AI Search / Qdrant 等） | 半新半旧：Vector Store 层是 RAG 检索，不是「记忆」语义；接 Palimpsest 最干净的是把检索包成 Plugin + 自定义 `VectorStore` | **高**。SK 记忆层面向量检索，缺少记忆演化（取代链/bi-temporal）的槽位；且 SK 已进维护模式 |
| **Google ADK** | `BaseMemoryService`（Python：`google.adk.memory`），方法：`add_session_to_memory` / `add_events_to_memory` / `add_memory` / `search_memory`。短期为 `SessionService` | **能，官方明确**：官方讨论帖确认「you can implement a custom `MemoryService` and deploy it with your agent」；部署可用 `--memory_service_uri` 指定自定义服务端点 | **有**：官方内置 **Preload memory**（「Automatically retrieves memory at the beginning of each turn, similar to a callback」）与 **Load memory**（模型自主调用）两个工具；另有 callback（如 `after_tool_callback`、写记忆的 callback） | **Mem0 官方集成**（`Mem0MemoryService` 实现 `BaseMemoryService`）；Zep 官方也提供 Google ADK 集成 | 实现 `BaseMemoryService` 子类→`Runner(memory_service=...)`；或纯 MCP 工具；有 `--memory_service_uri` 部署口 | **低-中**。接口清晰（4 个方法）且天然支持「每轮前预加载」，是**最贴合 Palimpsest 语义的槽位之一** |
| **OpenAI Agents SDK** | `Session` 协议（`agents.memory.session.Session`）：`get_items` / `add_items` / `pop_item` / `clear_session`，另有 `session_id` / `session_settings` | **能，官方明确**：「You can implement your own session memory by creating a class that structurally follows the `Session` protocol」（Python；JS 版亦然，无需继承 `SessionABC`） | **有**：`RunConfig.session_input_callback`（模型调用前自定义 history+new_input 合并）；`RunConfig.call_model_input_filter`（模型调用前最后一道）；自动压缩 `OpenAIResponsesCompactionSession` | **无框架内置第三方记忆类**，但 Mem0 有官方集成文档（`MemoryClient` + `function_tool` 模式）；社区有 Django ORM session 等 | ①自定义 `Session` 类（读写走 Palimpsest）②或 Mem0 式：注册 `function_tool` 让 agent 主动调 Palimpsest MCP | **低**（作为会话存储）/ **中**（作为长期记忆）。`Session` 语义是「会话历史」，不是「长期记忆」——需自行在 `session_input_callback` 里补注入长期记忆 |

---

## 2. 哪些框架的槽位最「通用」，最值得先接

**第一梯队（接口薄、语义近、官方明确可换后端）：**

1. **AutoGen** —— `Memory` 协议只有 5 个方法（`add`/`query`/`update_context`/`clear`/`close`），其中 `update_context` 的定位就是「每步前把检索结果写进 model_context」，与 Palimpsest 的「检索三合一 + 每轮注入」几乎一一对应。官方自己就用这套协议接了 Mem0/Chroma/Redis，是**验证过的第三方记忆接入套路**。
   ⚠️ 但 AutoGen 官方已宣布进入维护模式（不再加新功能，社区托管），**接它收益会折旧** —— [AutoGen 说明](https://a8gent.com/platforms/microsoft-autogen)

2. **Google ADK** —— `BaseMemoryService` 的 `add_memory`（显式灌入）、`search_memory`（检索）、`add_session_to_memory`（会话归档）与 Palimpsest 的写入/检索/会话归档直接对应；官方内置 **Preload memory（每轮前自动检索）** 钩子，天然支持「每轮调用前注入外部记忆」——这正是 Palimpsest 想要的位置。Mem0 官方就是靠实现 `BaseMemoryService` 完成集成（**接第三方的通用套路：实现厂商的 memory 抽象类 + 用官方预加载钩子注入**）。

3. **LangGraph** —— `BaseStore` 是显式可替换接口（官方文档明说 subclass + `langgraph.json` 的 `store` 键可整体替换内置 Postgres store），且 `pre_model_hook` 是标准化的「每轮模型调用前」钩子，Zep 官方集成正是走这条路。生态最强、第三方记忆示例最多。**代价**：`BaseStore` 要实现语义检索得自己接 embedding，比 ADK/AutoGen 的抽象重。

**第二梯队（能接，语义有偏差）：**

4. **LlamaIndex** —— `Memory` / `MemoryBlock` 可自定义，Mem0 有官方集成包（`llama-index-memory-mem0`），路径成熟；但 `Memory` 概念偏向「会话内 token 管理 + flush」，要承载 Palimpsest 的取代链/bi-temporal 需额外设计。

5. **CrewAI** —— 有 `StorageBackend` 协议和 `external_memory` 双通道，官方文档明确可换后端；但 CrewAI 自带 scope 树与复合打分（semantic+recency+importance），与 Palimpsest 的检索模型重叠，适配要做取舍。

6. **OpenAI Agents SDK** —— `Session` 协议好实现、钩子（`session_input_callback`）齐全，但 `Session` 是**会话历史**抽象而非**长期记忆**抽象。官方生态里 Mem0 是作为「外挂 function_tool 层」接入的，说明其官方定位就是「长期记忆请自己叠」——接 Palimpsest 最自然的方式是注册 MCP/function_tool。

**不建议优先（维护模式 / 语义不符）：**

7. **Semantic Kernel** —— 记忆层是 **Vector Store / RAG 检索**抽象，不是「带演化与取代链的记忆」抽象；且 SK 与 AutoGen 已被微软并入 **Microsoft Agent Framework**（2026-04 GA），处于维护模式。接 SK 等于接一个面向向量检索、且正在退役的层。

**结论（一句话）**：**先接 Google ADK 与 LangGraph，AutoGen 接口最薄但已退役可作参考/低成本尝鲜。** 三者的共同通用套路是：实现厂商的 memory 抽象类（`BaseMemoryService` / `BaseStore` / `Memory` 协议），再用官方的「每轮调用前」钩子（ADK Preload memory / LangGraph `pre_model_hook`）把 Palimpsest 的检索结果注入上下文。

---

## 附：关键 URL 索引

- LangGraph Store 接口/自定义后端：https://docs.langchain.com/oss/python/langgraph/stores
- LangGraph 自定义 Store 部署（langgraph.json `store` 键）：https://docs.langchain.com/langsmith/custom-store
- LangGraph 记忆概念（store vs checkpointer）：https://docs.langchain.com/oss/python/concepts/memory
- LangMem 介绍（含 BaseStore 集成）：https://langchain-ai.github.io/langmem
- LangMem 记忆工具 API：https://langchain-ai.github.io/langmem/reference/tools
- AutoGen Memory 协议与自定义 store / Mem0Memory：https://microsoft.github.io/autogen/stable//user-guide/agentchat-user-guide/memory.html
- CrewAI 统一 Memory 与 StorageBackend：https://docs.crewai.com/edge/en/concepts/memory
- CrewAI + MongoDB StorageBackend 实现示例：https://www.mongodb.com/docs/atlas/ai-integrations/crewai/agent-memory
- LlamaIndex Memory 组件与自定义：https://developers.llamaindex.ai/python/framework/module_guides/deploying/agents/memory
- LlamaIndex Memory 示例（MemoryBlock）：https://developers.llamaindex.ai/python/examples/memory/memory
- LlamaIndex + Mem0 官方集成页：https://docs.mem0.ai/integrations/llama-index
- LlamaHub mem0 记忆集成包：https://llamahub.ai/l/memory/llama-index-memory-mem0
- Semantic Kernel 矢量存储抽象（现役）：https://learn.microsoft.com/en-us/semantic-kernel/concepts/vector-store-connectors
- Semantic Kernel 旧记忆存储（IMemoryStore，建议迁移）：https://learn.microsoft.com/en-us/semantic-kernel/concepts/vector-store-connectors/memory-stores
- SK 是否废弃 / 官方澄清：https://github.com/microsoft/semantic-kernel/discussions/12299
- Google ADK Memory（MemoryService/Preload memory）：https://adk.dev/sessions/memory
- ADK 自定义 MemoryService（官方讨论帖）：https://github.com/google/adk-python/discussions/2548
- ADK + Mem0 官方集成：https://docs.mem0.ai/integrations/google-ai-adk
- OpenAI Agents SDK Sessions（自定义 Session 协议 / session_input_callback）：https://openai.github.io/openai-agents-python/sessions
- OpenAI Agents SDK JS Sessions（Session 接口）：https://openai.github.io/openai-agents-js/guides/sessions
- Mem0 × OpenAI Agents SDK 官方集成：https://docs.mem0.ai/integrations/openai-agents-sdk
- Zep 官方框架集成总览（含 LangGraph pre-model hook / AutoGen / CrewAI / ADK）：https://help.getzep.com/memory-for-agent-frameworks
- Zep × LangGraph（ZepPreModelHook）：https://help.getzep.com/langgraph-memory
- AutoGen 维护模式与 Microsoft Agent Framework 继任：https://a8gent.com/platforms/microsoft-autogen
- LangChain 官方对 AutoGen 维护模式的说明：https://www.langchain.com/resources/langchain-vs-autogen
