# 何嘉琪 · 个人站

> **在线阅读：<https://jiay-77.github.io/portfolio/>**
> 这里不是作品清单，是一个关于「记忆」的长期项目现场：我在想什么、做了什么、怎么验证。

# 🧠 Agent + Memory = Assistant
> *Stimulating the potential of memory to build true AI assistants.*

## 这个站点有什么

| 栏目 | 内容 |
|---|---|
| [思考](https://jiay-77.github.io/portfolio/writing/) | 三条我正在琢磨的判断：agent + memory = assistant；记忆与涌现；本地优先的记忆持久化 |
| [作品](https://jiay-77.github.io/portfolio/work/) | 五篇文档，五个视角：设计 / 判断 / 方法 / 验证 / 交付 |
| [调研](https://jiay-77.github.io/portfolio/research/) | 为立项做的调研：五家 Agent 的记忆机制、四款记忆工具对比、宿主接入优先级 |
| [经历](https://jiay-77.github.io/portfolio/experience/hanlink-itbp/) | ITBP 与 AI 产品售前——方法是从现场长出来的 |
| [关于](https://jiay-77.github.io/portfolio/about/) | 技能、教育、联系方式 |

主作品：[**Palimpsest**](https://github.com/JiaY-77/Palimpsest) —— 本地优先的开源 AI 长期记忆系统。

## 技术说明

- 静态站点，用 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 构建
- 内容全部是 Markdown，位于 `docs/`；站点配置见 `mkdocs.yml`
- 推送到 `main` 后由 GitHub Actions 自动构建并发布到 GitHub Pages（`.github/workflows/deploy.yml`）

本地预览：

```bash
uv run --with "mkdocs-material>=9.5,<10" mkdocs serve
```

## 关于内容

- 所有技术结论标来源、标时间；查不到就写「查不到」
- 个人角色如实标注：本人负责问题定义、架构决策、方案验证与成果验收，编码由 AI 编程助手按需求实现
- 涉及前公司 / 客户的内容已脱敏，只保留方法与角色
