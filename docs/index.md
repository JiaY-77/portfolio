---
title: 何嘉琪
hide:
  - navigation
  - toc
---

<div class="hero" markdown>

<div class="hero-head">
<img class="avatar" src="assets/avatar.jpg" alt="何嘉琪">
<div class="hero-title">
<h1>何嘉琪</h1>
<p class="tagline">🧠 Agent + Memory = Assistant</p>
<p class="tagline-en">Stimulating the potential of memory to build true AI assistants.</p>
</div>
</div>

我做的一件事：**给 agent 装上记得住、想得起、说得清的记忆**。

它们的总称是 [Palimpsest](https://github.com/JiaY-77/Palimpsest)——一个本地优先的开源 AI 长期记忆系统，
1047 项自动化测试、24 个正式版本，用 MCP 标准协议对外提供 21 个工具。

[我在想什么](writing/index.md){ .md-button .md-button--primary }
[看作品](work/index.md){ .md-button }
[:fontawesome-brands-github: GitHub](https://github.com/JiaY-77){ .md-button }

</div>

## 我目前在琢磨的三件事

<div class="hypothesis" markdown>

### 〔01〕agent + memory = assistant

模型给能力，工具给手脚，**记忆给连续性**。没有记忆的 agent，每次见面都是第一次见你。

[读这篇](writing/agent-memory-assistant.md)

</div>

<div class="hypothesis" markdown>

### 〔02〕高质量的记忆，是 AI 涌现的土壤

涌现常被说成「参数大到一定程度自然发生」。我怀疑另一件事：**在长程任务里，模型能不能表现出超出单轮的能力，取决于它有多少可信、可检索、可追溯的上下文**。

[读这篇](writing/memory-and-emergence.md)

</div>

<div class="hypothesis" markdown>

### 〔03〕记忆的所有权，应该属于使用它的人

记忆是最私人的数据，但主流方案把它放在别人的云上。本地优先不是洁癖，是**换模型、换工具、换机器时不失忆**的前提。

[读这篇](writing/local-first-memory.md)

</div>

---

## 主作品：Palimpsest

**本地优先的 AI 长期记忆系统** —— 覆而不失，改写可溯。

语义向量 + 全文检索 + 知识图谱三路混合检索，RRF 融合排序；新事实覆盖旧事实，但旧版本通过 `REVISED_BY` 版本链可追溯；
REST / MCP / CLI 三接入共用一内核，21 个工具能力对等；本地优先、无云层。

- 我的角色：**产品定义者与设计者**——需求定义、架构决策、方案验证、成果验收；编码由 AI 编程助手按需求实现，我负责审阅与验收
- 规模：产品代码约 1.06 万行、测试代码约 1.93 万行（测试比产品代码多）、**1047 项测试**、266 次提交、24 个正式版本
- [设计说明](work/palimpsest-design.md) ｜ [源码仓库](https://github.com/JiaY-77/Palimpsest)

---

## 想更了解我

<div class="grid cards" markdown>

-   :material-file-document-outline:{ .lg .middle } __作品__

    ---

    五篇文档，五个视角：设计、判断、方法、验证、交付。

    [:octicons-arrow-right-24: 看作品](work/index.md)

-   :material-flask-outline:{ .lg .middle } __调研__

    ---

    为立项做的调研：五家 Agent 的记忆机制、四款记忆工具对比、宿主接入优先级。

    [:octicons-arrow-right-24: 看调研](research/index.md)

-   :material-briefcase-outline:{ .lg .middle } __经历__

    ---

    ITBP 与 AI 产品售前——我的工作方法是从这些现场里长出来的。

    [:octicons-arrow-right-24: 看经历](experience/hanlink-itbp.md)

-   :material-account-circle-outline:{ .lg .middle } __关于__

    ---

    技能、教育，以及我信的四条规矩。

    [:octicons-arrow-right-24: 关于我](about.md)

</div>
