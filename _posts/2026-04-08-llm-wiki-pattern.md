---
layout: post
title: Karpathy 的 LLM Wiki 模式：让 AI 构建你的个人知识库
categories: [AI]
description: 介绍 Andrej Karpathy 提出的 LLM Wiki 模式，用 AI 增量构建和维护个人知识库，实现知识的累积和复用
keywords: LLM, AI, 知识库, RAG, Wiki, Obsidian, 个人知识管理
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是 Karpathy 提出的一个革命性的个人知识管理思路：用 LLM 构建一个会自动进化、累积、交叉引用的 Wiki

## 传统 RAG 的问题

我们熟悉的使用 LLM 处理文档的方式是这样的：

```
上传文档 → RAG 检索 → LLM 生成答案 → 结束
```

这很常用，但有一个根本问题：

> **LLM 每次都在从零开始「重新发现」知识**
>
> 问一个需要综合 5 份文档的问题？LLM 每次都要找出并拼凑相关片段。
> 什么都没积累下来。下次问类似问题，还是重来一遍。

---

## Karpathy 的新思路：LLM Wiki

### 核心理念

> **不要在查询时检索原始文档，而是让 LLM 增量构建和维护一个持久的 Wiki**

```
新文档进入
    ↓
LLM 读取文档 → 提取关键信息 → 整合到现有 Wiki
    ↓
更新实体页面 + 修订主题摘要 + 标记矛盾点 + 增强交叉引用
    ↓
Wiki 越来越丰富，每次查询都在已有的基础上工作
```

**关键区别：Wiki 是一个持久的、不断累积的产物。**

- 交叉引用已经就位
- 矛盾点已经被标记
- 综合结论已经反映了你读过的所有内容
- 每添加一个源，Wiki 就会变得更有价值

---

## 三层架构

LLM Wiki 包含三个层次：

### 1. Raw Sources（原始来源）

```
Articles / Papers / Images / Data Files
```

这是你的精选源文档集合。**不可变** — LLM 只读取，不修改。这是你的真理来源。

### 2. The Wiki（知识库）

```
Summaries / Entity Pages / Concept Pages / Comparisons / Overview
```

LLM 完全拥有的层。它创建页面、当新源到达时更新它们、维护交叉引用、保持一切一致。**你读它，LLM 写它。**

### 3. The Schema（规范）

```markdown
# CLAUDE.md 或 AGENTS.md

# Wiki 结构规范
# 约定规则
# 工作流程
```

告诉 LLM Wiki 是如何组织的、约定是什么、在摄取源、回答问题或维护 Wiki 时要遵循什么工作流程。

**这是关键配置文件** — 它让 LLM 成为有纪律的 Wiki 维护者，而不是通用聊天机器人。

---

## 核心操作

### Ingest（摄取）

```
你：把新文档丢进原始收集
LLM：
  - 读取源文档
  - 与你讨论关键要点
  - 在 Wiki 中写摘要页面
  - 更新索引
  - 更新相关实体和概念页面
  - 在日志中追加条目

一份源文档可能涉及 10-15 个 Wiki 页面
```

### Query（查询）

```
你：向 Wiki 提问
LLM：
  - 搜索相关页面
  - 读取它们
  - 综合答案并给出引用

答案可以是不同形式：
  - Markdown 页面
  - 对比表格
  - 幻灯片（Marp）
  - 图表（matplotlib）
  - Canvas
```

**重要洞察：好的答案可以作为新页面存档回 Wiki。**

你问过的一个对比、你做的一个分析、你发现的一个联系 — 这些都是有价值的，不应该消失在聊天历史中。这样你的探索就像摄取源一样在知识库中累积。

### Lint（检查）

```
定期让 LLM 做 Wiki 健康检查：

检查项：
  - 页面之间的矛盾
  - 被新源取代的不新鲜声明
  - 没有入站链接的孤立页面
  - 提到但缺少自己页面的重要概念
  - 缺失的交叉引用
  - 可以通过网络搜索填补的数据空白

LLM 擅长建议新的要调查的问题和要寻找的新来源
```

---

## 两个特殊文件

### index.md — 内容导向

```
# Wiki Index

## Entities（实体）
- [[Alice]] — 项目负责人，专注机器学习
- [[Project Alpha]] — 2024年启动的内部项目

## Concepts（概念）
- [[Attention Mechanism]] — Transformer 的核心组件
- [[RAG]] — 检索增强生成模式

## Sources（来源）
- [[Paper: Attention Is All You Need]]
- [[Blog: LLM Wiki Pattern]]

## Overviews（概述）
- [[Project Alpha 进展总结]]
- [[Q1 技术趋势分析]]
```

LLM 每次摄取后更新它。回答查询时，LLM 先读索引找相关页面，再深入阅读。**在中等规模（~100 源，~数百页）下效果出奇地好，避免了对基于嵌入的 RAG 基础设施的需求。**

### log.md — 时间导向

```markdown
# Wiki Log（按时间顺序的只追加记录）

## [2026-04-08] ingest | LLM Wiki 原始 Gist
## [2026-04-08] query | LLM Wiki 和 RAG 有什么区别
## [2026-04-08] lint | Wiki 健康检查
## [2026-04-07] ingest | Karpathy nanoGPT 文档
## [2026-04-07] query | nanoGPT 和 nanochat 的区别
```

**技巧：** 如果每个条目以一致的前缀开始（例如 `## [YYYY-MM-DD] ingest |`），日志可以用简单的 Unix 工具解析：

```bash
grep "^## \[" log.md | tail -5  # 最后 5 条记录
```

---

## 工作流程演示

### 场景：研究一个新主题

```
Day 1:
你：帮我建立一个 RustDesk 的 Wiki
LLM：创建 Wiki 基础结构，创建 index.md 和 log.md

Day 2:
你：摄取这篇 RustDesk 部署文章
LLM：
  - 创建 [[RustDesk 部署指南]] 摘要页
  - 更新 [[远程桌面软件]] 概念页
  - 添加到 [[待研究]] 列表
  - 更新 log.md

Day 3:
你：再摄取这篇 RustDesk 安全配置文章
LLM：
  - 创建 [[RustDesk 安全配置]] 摘要页
  - 发现与之前部署文章有矛盾点 → 标记
  - 更新 [[RustDesk 部署指南]] 添加安全注意事项
  - 更新 log.md

Day 4:
你：RustDesk 和向日葵对比如何？
LLM：
  - 读取相关页面
  - 生成 [[RustDesk vs 向日葵]] 对比页
  - 存档到 Wiki（重要知识）
  - 更新 log.md

Day 5:
你：帮我检查 Wiki 健康状况
LLM：
  - 发现 [[Port 配置]] 页面孤立
  - 发现 [[21116 端口]] 需要更新（官方有新说明）
  - 建议：补充 [[NAT 类型测试]] 相关知识
  - 更新 log.md
```

---

## 工具推荐

### Obsidian — Wiki 的 IDE

Karpathy 的个人工作流：

```
┌─────────────────┐     ┌─────────────────┐
│  LLM Agent     │ ←→  │    Obsidian     │
│  (Claude Code) │     │   (Wiki 编辑)   │
└─────────────────┘     └─────────────────┘
     LLM 做编辑            你实时浏览结果
     根据对话进行           跟随链接
     操作                   查看图视图
                           读更新的页面
```

**Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。**

### Obsidian Web Clipper

浏览器扩展，将网页文章转换为 Markdown。**快速将源获取到原始收集的必备工具。**

### 本地图片下载

在 Obsidian 设置中：
1. 设置 → 文件和链接 → 附件文件夹路径设为固定目录（如 `raw/assets/`）
2. 设置 → 热键 → 搜索「下载」→ 绑定快捷键（如 `Ctrl+Shift+D`）
3. 剪藏文章后按快捷键，所有图片下载到本地

### 搜索工具：qmd

当 Wiki 变大时，需要 proper search：

> [qmd](https://github.com/tobi/qmd) — 本地 Markdown 文件搜索引擎，支持 BM25/向量混合搜索和 LLM 重排，全部设备上运行。它有 CLI（LLM 可以 shell 调用）和 MCP server（LLM 可以作为原生工具使用）。

---

## 适用场景

| 场景 | 说明 |
|------|------|
| **个人成长** | 追踪自己的目标、健康、心理、自我提升 — 整理日记条目、文章、播客笔记 |
| **学术研究** | 数周或数月深入一个主题 — 读论文、文章、报告，逐步构建综合 Wiki |
| **读书笔记** | 每读一章就整理一页，构建角色、主题、情节线的页面。到最后你有一个丰富的伴侣 Wiki |
| **团队知识库** | 内部 Wiki 由 LLM 维护，来源包括 Slack 线程、会议记录、项目文档、客户电话 |
| **竞品分析** | 持续跟踪竞争对手动态 |
| **尽职调查** | 系统化整理调研信息 |
| **旅行规划** | 积累行程、攻略、笔记 |
| **课程笔记** | 构建学科知识的深度 Wiki |

---

## vs 传统 RAG

| 对比项 | 传统 RAG | LLM Wiki |
|--------|----------|----------|
| 知识状态 | 每次重新发现 | 持久累积 |
| 交叉引用 | 每次重新建立 | 已有 |
| 矛盾处理 | 无 | 已标记 |
| 综合分析 | 临时生成 | 已有存档 |
| 维护成本 | 低 | 需要 LLM 持续维护 |
| 适用场景 | 简单问答 | 深度研究积累 |

---

## 快速开始

### 1. 创建基础结构

```markdown
# 你的 Wiki

/root
├── README.md          # Wiki 介绍
├── index.md          # 内容索引
├── log.md            # 操作日志
├── CLAUDE.md         # LLM 规范（必填！）
├── raw/               # 原始文档
│   ├── papers/
│   ├── articles/
│   └── notes/
└── wiki/             # LLM 生成的知识库
    ├── entities/
    ├── concepts/
    ├── summaries/
    └── analyses/
```

### 2. 创建 CLAUDE.md 规范

```markdown
# CLAUDE.md

## Wiki 概述
这是一个关于 [你的主题] 的个人 Wiki。

## Wiki 结构
- `raw/` — 原始源文档（不可变）
- `wiki/` — LLM 生成的知识库
  - `entities/` — 实体页面（人、项目、产品）
  - `concepts/` — 概念页面（技术、方法论）
  - `summaries/` — 源文档摘要
  - `analyses/` — 分析和综合

## 工作流程

### 摄取新源
1. 读取源文档
2. 与用户讨论关键要点
3. 在 `wiki/summaries/` 创建摘要页
4. 更新相关实体/概念页面
5. 更新 `index.md`
6. 在 `log.md` 追加条目

### 回答查询
1. 读取 `index.md` 找到相关页面
2. 深入阅读相关页面
3. 综合答案，给出引用
4. 好的答案存档到 `wiki/analyses/`

### Lint 检查
1. 检查矛盾
2. 检查过时信息
3. 检查孤立页面
4. 建议新的研究问题

## 约定
- 页面标题用 [[WikiLinks]]
- 使用 YYYY-MM-DD 时间格式
- 每个概念页至少 3 个交叉引用
```

### 3. 开始使用

```
1. 打开 Obsidian，创建 Vault
2. 按上述结构创建目录
3. 配置 CLAUDE.md
4. 打开 Claude Code / Cursor 等 LLM Agent
5. 把 Obsidian 目录作为工作目录
6. 开始摄取你的第一个源文档
```

---

## 总结

LLM Wiki 模式的核心洞察：

> **知识应该被编译一次，然后保持最新，而不是每次查询时重新派生。**

| 特点 | 说明 |
|------|------|
| 持久累积 | Wiki 随每个源和每个问题变得更加丰富 |
| LLM 维护 | 你负责来源、探索、提问；LLM 做所有工作 |
| 交叉引用 | 矛盾被标记，综合已完成 |
| 工具支持 | Obsidian + Web Clipper + LLM Agent |

这不是要取代 RAG，而是为**需要深度积累和持续研究**的场景提供了一种更可持续的模式。

---

## 参考资料

- [Karpathy LLM Wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Obsidian 官网](https://obsidian.md/)
- [qmd - 本地 Markdown 搜索](https://github.com/tobi/qmd)

---

*作者：牛马便利店一号店员*
