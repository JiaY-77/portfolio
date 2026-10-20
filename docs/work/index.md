---
title: 作品
description: 五个视角的深度文档：设计、判断、方法、验证、交付
---

# 作品总览

主作品是 [Palimpsest](https://github.com/JiaY-77/Palimpsest)（本地优先的开源 AI 长期记忆系统）。
下面五篇文档从五个不同角度把它和相关方法摊开——每一篇都能经得起追问，而不是只有结论。

<div class="grid cards" markdown>

-   :material-sitemap:{ .lg .middle } __Palimpsest 设计说明__

    ---

    **工程视角**：立项背景 → 主流方案调研 → 架构选型 → 设计决策 → 近期演进。
    关注点：为什么是这一套，而不是别的。

    [:octicons-arrow-right-24: 阅读](palimpsest-design.md)

-   :material-scale-balance:{ .lg .middle } __记忆系统的设计与取舍__

    ---

    **判断视角**：本地实测 Mem0 / Cognee，源码级定位两处上游缺陷，核查上游状态后做出「不予提交」的判断。
    关注点：什么值得做，什么不值得做。

    [:octicons-arrow-right-24: 阅读](memory-tradeoffs.md)

-   :material-fingerprint:{ .lg .middle } __大模型 API 真伪鉴别__

    ---

    **方法视角**：五层鉴别法——上游结构指纹 → 知识截止探针 → 决定性探针 → 路由漂移检测 → 真实单价实测。
    关注点：可复现的判据，不靠「问它你是谁」。

    [:octicons-arrow-right-24: 阅读](api-authenticity.md)

-   :material-package-check:{ .lg .middle } __依赖升级验证与上游贡献__

    ---

    **验证视角**：隔离 venv 的 A/B 压测、零迁移验证、性能回归定位与上游 issue/PR——以及一个被我自己撤回的结论。
    关注点：怎么保证一次依赖升级不出事。

    [:octicons-arrow-right-24: 阅读](dependency-upgrade.md)

-   :material-timer-outline:{ .lg .middle } __真实业务数据交付（30 分钟）__

    ---

    **交付视角**：脏数据体检 → 候选根因逐个证伪 → 交叉验证与稳健性检验 → 可执行阈值与工时 ROI。
    关注点：结论怎么才算可验收。

    [:octicons-arrow-right-24: 阅读](data-delivery-30min.md)

</div>

---

## 我的角色

以上均为本人**主导完成**：负责问题定义、方法论设计、技术判断与结论验收；
具体执行（探测脚本、数据整理、编码实现）由 AI 编程助手按需求实现，我负责审阅、测试验收与迭代决策。

这是我 2026 年的工作方式，也是我认为这个岗位最核心的能力：**知道该做什么，更知道什么不该做。**
