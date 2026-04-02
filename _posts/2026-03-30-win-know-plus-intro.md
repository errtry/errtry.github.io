---
layout: post
title: WiNEX 智能知识库检索系统 v3.0：四路知识源统一检索
categories: [Work]
description: 介绍我们团队开发的 WiNEX 智能知识库检索系统，整合 Wiki、SVN、TFS、本地文档四路知识源，实现一键统一检索
keywords: WiNEX, 知识库, 检索, Wiki, SVN, TFS, 本地文档
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一个为 WiNEX 实施团队量身定制的智能知识库检索系统，今晚分享给同事们。

## 痛点：知识分散，查找困难

在 WiNEX 项目实施中，我们经常遇到这样的问题：

```
场景：客户问「门诊退费审核怎么配置？」
我们需要：
1. 查 Wiki → 配置说明
2. 查 SVN → 实施手册
3. 查 TFS → 历史工单
4. 查本地文档 → 项目资料

四个地方来回切换，效率低下！
```

**痛点总结：**
- 知识分散在 4 个不同的系统
- 每个系统都要单独登录、单独搜索
- 搜索结果不统一，需要人工整合
- 新人上手困难，不知道去哪找资料

---

## 解决方案：四路知识源统一检索

我们开发了 **WiNEX 智能知识库检索系统 v3.0**，核心思想：

> **一个命令，四路齐发**

### 系统架构

```
┌─────────────────────────────────────────────────────────────┐
│              WiNEX 智能知识库检索系统 v3.0                   │
├─────────────────────────────────────────────────────────────┤
│  统一入口                                                    │
│  └── node search_all.mjs "关键词" → 同时检索四路知识源       │
├─────────────────────────────────────────────────────────────┤
│  四路知识源                                                  │
│  ├─ ① Wiki (实时搜索) ✅                                    │
│  │   └── https://winwiki.winning.com.cn                      │
│  ├─ ② SVN 文档+视频 (预索引) ✅                              │
│  │   └── https://svn.winning.com.cn                          │
│  ├─ ③ TFS 工作项 (实时搜索) ✅                              │
│  │   └── http://tfs2018-web.winning.com.cn:8080/tfs        │
│  └─ ④ 本地文档 (已索引) ✅                                  │
│      └── E:\sdkq\文档 (226份, 1.7GB)                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 功能演示

### 统一检索（推荐）

```bash
# 一个命令，同时检索四路知识源
node search_all.mjs "门诊退费审核"
```

**输出示例：**

```
━━━ 📁 Wiki ━━━
[1] 门诊退药-药房申请退药医生审核
    https://winwiki.winning.com.cn/pages/viewpage.action?pageId=28708
    匹配度：95%

[2] 门诊退费流程三级配置
    https://winwiki.winning.com.cn/pages/viewpage.action?pageId=28915
    匹配度：88%

━━━ 📁 SVN文档 ━━━
📄 实施手册-门诊系统.pdf
    路径：/实施文档/门诊系统/实施手册-门诊系统.pdf
    大小：12.5MB

📄 门诊退费配置说明.docx
    路径：/项目文档/东明县中医医院/门诊退费配置说明.docx
    大小：3.2MB

━━━ 📁 TFS工作项 ━━━
📄 [1085089] 东明县中医医院-门诊退费审核对接
    状态：已完成
    创建人：张三
    创建时间：2025-11-15

📄 [1092345] 门诊退费审核流程优化
    状态：进行中
    负责人：李四
    创建时间：2026-01-20

━━━ 📁 本地文档 ━━━
📄 WN-SSJS-PZ-001《业务配置手册》-WiNEX财务.docx
    路径：E:\sdkq\文档\业务配置手册\WN-SSJS-PZ-001《业务配置手册》-WiNEX财务.docx
    大小：39.5MB
```

### 单独检索

```bash
# 只检索 Wiki
node wiki/run.mjs "门诊退费"

# 只检索 TFS 工作项
node search_tfs.mjs "门诊退费"

# 只检索 SVN 文档
python search_svn.py "医保登记"

# 只检索本地文档
python winex_search.py "退费"
```

---

## 技术实现

### 1. 统一检索入口

核心文件：`search_all.mjs`

```javascript
// 同时启动四个检索任务
const tasks = [
  { name: 'Wiki', cmd: `node wiki/run.mjs "${keyword}"` },
  { name: 'TFS', cmd: `node search_tfs.mjs "${keyword}"` },
  { name: 'SVN', cmd: `python search_svn.py "${keyword}"` },
  { name: '本地文档', cmd: `python winex_search.py "${keyword}"` }
];

// 并行执行，结果汇总
const results = await Promise.allSettled(tasks.map(task => execAsync(task.cmd)));
```

### 2. 缓存机制

```javascript
// 5分钟缓存，避免重复请求
const CACHE_TTL = 5 * 60 * 1000;

function getCache(key) {
  const cacheFile = path.join(CACHE_DIR, `${key}.json`);
  if (fs.existsSync(cacheFile)) {
    const stat = fs.statSync(cacheFile);
    if (Date.now() - stat.mtimeMs < CACHE_TTL) {
      return JSON.parse(fs.readFileSync(cacheFile, 'utf8'));
    }
  }
  return null;
}
```

### 3. 配置文件管理

所有配置集中管理：

```
config/
├── .svn.config.json      # SVN 账号配置
├── .tfs.config.json      # TFS PAT token
└── wiki/
    └── .winwiki.config.json  # Wiki 账号配置
```

---

## 性能优化

| 优化项 | 效果 |
|--------|------|
| 并行检索 | 四路同时搜索，总耗时 = 最慢的那一路 |
| 5分钟缓存 | 相同关键词第二次搜索秒出结果 |
| 结果排序 | 按相关性排序，最相关的结果排前面 |
| 错误隔离 | 一路失败不影响其他路 |

**性能数据：**
- 首次检索：~60 秒（四路并行）
- 缓存命中：~0 秒
- 单路失败：自动跳过，不影响其他路

---

## 使用场景

### 场景 1：客户咨询

```
客户：门诊退费审核怎么配置？
你：node search_all.mjs "门诊退费审核"
→ 立刻得到：Wiki配置 + 实施手册 + 历史工单 + 项目文档
→ 5分钟内给出完整答案
```

### 场景 2：新人培训

```
新人：这个功能怎么实现？
你：node search_all.mjs "功能关键词"
→ 新人自己就能找到所有相关资料
→ 减少老员工带教时间
```

### 场景 3：问题排查

```
问题：某个功能报错
你：node search_all.mjs "错误关键词"
→ 找到：Wiki解决方案 + TFS类似工单 + 相关文档
→ 快速定位问题原因
```

---

## 安装部署

### 环境要求

| 依赖 | 版本要求 | 说明 |
|-----|---------|------|
| Python | 3.12+ | 用于 SVN/本地文档检索 |
| Node.js | 16+ | 用于 Wiki/TFS 检索 |

### 快速安装

```bash
# 1. 克隆仓库
git clone https://github.com/errtry/win-know-plus.git

# 2. 安装依赖
cd win-know-plus
npm install

# 3. 配置账号（联系管理员获取配置文件）
# 复制 config/ 目录下的配置文件

# 4. 测试
node search_all.mjs "测试"
```

---

## 对比传统方式

| 方式 | 耗时 | 完整性 | 易用性 |
|------|------|--------|--------|
| 传统方式 | 10-30分钟 | ❌ 可能漏掉某些来源 | ❌ 需要切换多个系统 |
| 本系统 | 1-2分钟 | ✅ 四路全覆盖 | ✅ 一个命令搞定 |

**效率提升：5-10 倍**

---

## 未来规划

### v3.1（计划中）
- ✅ 搜索结果收藏功能
- ✅ 自动生成知识图谱

### v4.0（规划中）
- ✅ AI 智能问答（直接问，直接答）
- ✅ 知识自动更新（监控知识源变化）

---

## 总结

**WiNEX 智能知识库检索系统** 解决了我们团队的核心痛点：

1. **知识分散** → 四路统一检索
2. **查找困难** → 一个命令搞定
3. **效率低下** → 并行搜索 + 缓存
4. **新人上手难** → 简单易用

**核心价值：**
- 提升问题解决速度 5-10 倍
- 减少重复劳动
- 统一团队知识管理
- 赋能新人快速成长

---

## 配套 PPT

<iframe src="/assets/slides/win-know-plus.html" width="100%" height="500px" frameborder="0" style="border: 1px solid #ddd; border-radius: 10px;"></iframe>

[点击这里全屏查看 PPT](/assets/slides/win-know-plus.html)

---

*作者：牛马便利店一号店员*
