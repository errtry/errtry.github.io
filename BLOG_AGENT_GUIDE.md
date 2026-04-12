# 博客运营 AI Agent 标准操作手册

> 目标：让接手博客的 AI 能独立完成从「选题」到「推送」的全流程，无需人工干预。
>
> 作者署名：**牛马便利店一号店员**
> 博客路径：`D:\Chuangyu_VNet\blog\errtry.github.io`
> 域名：geekhappy.com

---

## 一、核心工作流（标准流程）

```
选题 → 调研 → 写文章 → [制作PPT] → 提交 Git → 推送
```

### 1.1 选题

**选题来源**（优先级从高到低）：

1. 用户明确指定主题 → 直接进入调研
2. 热点追踪 → 定期搜索以下来源：
   - ArXiv cs.AI / cs.CL 最新论文
   - GitHub Trending（AI / Python / DevOps 相关）
   - 技术社区：Hacker News、掘金、InfoQ
   - 行业新闻：AI 圈大事件、开源项目动态

3. 主动发现 → 当有以下情况时主动提案：
   - 博客长时间（>7天）没有新文章
   - 用户提到了一个值得深挖的话题
   - 发现了适合博客定位的优质内容

**博客定位**：
- 目标读者：开发者、技术爱好者
- 内容风格：实用、落地、不假大空
- 技术方向：AI/LLM、DevOps、后端开发、效率工具

### 1.2 调研

写文章前必须完成调研：

```bash
# 搜索相关资料
web_search "主题关键词 site:github.com"
web_search "主题关键词 教程"
web_fetch "官网文档URL"
```

**调研清单**：
- [ ] 找到官方文档/最新版本说明
- [ ] 搜索 2-3 篇高质量参考文章
- [ ] 确认代码示例能否直接运行
- [ ] 了解常见坑和最佳实践

### 1.3 写文章

**文件路径**：`D:\Chuangyu_VNet\blog\errtry.github.io\_posts\`

**文件名格式**：`YYYY-MM-DD-文章标题英文.md`

**Front Matter 模板**：

```yaml
---
layout: post
title: 文章标题
categories: [分类]
description: 一句话描述文章内容
keywords: 关键词1, 关键词2, 关键词3
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
---

> 💡 简短导语，吸引读者继续往下看。
```

**文章结构标准**：

| 章节 | 内容要求 |
|------|----------|
| 导语 | 1-2 句话说明文章背景和价值 |
| 什么是/为什么 | 基础概念 + 应用场景 |
| 快速开始 | 5-10 行可运行的代码 |
| 核心功能详解 | 功能逐一讲解，配合代码示例 |
| 进阶用法 | 高级配置、常见坑 |
| 总结 | 要点回顾 |

**写作原则**：
- ✅ 每一行代码都要能直接复制运行
- ✅ 每个功能点都要有具体示例
- ✅ 遇到不确定的参数去官方文档核实
- ❌ 不要写「大家应该都知道了」之类的废话
- ❌ 不要堆砌概念，要以实操为主

### 1.4 制作 PPT（可选）

**条件**：文章适合演示讲解，或者用户要求

**文件路径**：`D:\Chuangyu_VNet\blog\errtry.github.io\assets\slides\`

**文件名**：`文章主题.html`（与文章名对应）

**模板**：参考已有的 PPT 文件：
- `badusb-digispark.html`
- `revealjs-tutorial.html`
- `win-know-plus.html`

**嵌入文章**：

```markdown
{% raw %}
<!-- 幻灯片演示 -->
<iframe 
    src="/assets/slides/文件名.html"
    width="100%"
    height="500"
    frameborder="0"
    allowfullscreen>
</iframe>
{% endraw %}
```

### 1.5 Git 提交推送

**关键**：锁文件问题（PowerShell 执行多命令时易触发）

**推荐做法**（分步执行，避免锁冲突）：

```powershell
# 第一步：暂存
cd D:\Chuangyu_VNet\blog\errtry.github.io
git add -A

# 第二步：等待锁释放（如果报锁文件错误）
Start-Sleep -Seconds 3

# 第三步：提交
git commit -m "feat: 文章标题"

# 第四步：推送
git push
```

**提交流程规范**：

| 类型 | commit 前缀 |
|------|-------------|
| 新文章 | `feat:` |
| 优化文章 | `fix:` / `perf:` |
| SEO/配置 | `chore:` |
| 删除文件 | `chore:` |

**commit 消息示例**：
```
feat: 新增 RustDesk 自建服务器完整部署指南
fix: 修复文章内的代码错误
chore: 优化 SEO 配置
```

---

## 二、知识库资源

写技术文章时善用这些资源：

### 2.1 内部知识库（最优先）

```bash
# WiNEX 知识库检索（内网）
node "D:\Chuangyu_VNet\blog\errtry.github.io\..\win-know-plus\search_all.mjs" "搜索关键词"

# 本地文档检索
python "D:\Chuangyu_VNet\blog\errtry.github.io\..\win-know-plus\winex_search.py" "搜索关键词"
```

### 2.2 外部资料获取

```bash
# 搜索技术文档
web_search "主题 site:官方文档域名"

# 获取页面内容
web_fetch "https://官方文档URL"
```

### 2.3 参考文章格式

写文章时如果参考了多篇资料，在文章末尾加上：

```markdown
## 参考资料

- [官方文档](URL)
- [参考文章1](URL)
- [参考文章2](URL)
```

---

## 三、选题库（日常储备）

当没有明确选题时，可从以下方向选择：

### 高价值方向（容易出爆款）
- AI/LLM 工具使用、比较、评测
- 自建服务教程（NAS、远程桌面、代理等）
- 开发效率工具对比和配置
- 实战踩坑记录和解决方案

### 中等价值方向（稳定输出）
- 某个技术的系统性教程
- 开发环境搭建指南
- 命令行工具使用手册
- 某个开源项目的深度使用

### 低价值方向（避免）
- 纯概念科普（网上已经很多）
- 过于小众的工具
- 与博客定位无关的内容

---

## 四、质量检查清单

写完文章后逐项检查：

- [ ] 文章能解决一个具体问题吗？
- [ ] 代码能直接复制运行吗？
- [ ] 有没有遗漏的关键步骤？
- [ ] 截图/演示是否有说明？
- [ ] 作者署名正确吗？（**牛马便利店一号店员**）
- [ ] Git 提交信息规范吗？

---

## 五、Git 推送失败处理

### 锁文件问题

```
fatal: Unable to create 'xxx/.git/index.lock': Permission denied
```

**原因**：SourceTree、VSCode 或其他 Git 工具正在访问仓库。

**解决方法**：
1. 关闭所有可能占用 Git 的程序
2. 或等待 5 秒后重试
3. 或手动运行 PowerShell 分步推送

### 推送冲突

```
error: failed to push some refs
```

**解决方法**：
```powershell
git pull --rebase
git push
```

---

## 六、快速参考

| 操作 | 命令/路径 |
|------|-----------|
| 文章目录 | `D:\Chuangyu_VNet\blog\errtry.github.io\_posts\` |
| PPT 目录 | `D:\Chuangyu_VNet\blog\errtry.github.io\assets\slides\` |
| Wiki 目录 | `D:\Chuangyu_VNet\blog\errtry.github.io\_wiki\` |
| 博客地址 | https://geekhappy.com |
| Git 仓库 | https://github.com/errtry/errtry.github.io |

### 推送脚本（备用）

如果 Git 命令经常出问题，用这个脚本：

```powershell
# 文件：git-push.ps1
cd D:\Chuangyu_VNet\blog\errtry.github.io
git add -A
git commit -m $args[0]
git push
```

运行：`.\git-push.ps1 "feat: 文章标题"`

---

## 七、自动化触发条件

**AI 应在以下情况下主动行动**：

| 触发条件 | 行动 |
|---------|------|
| 博客超过 7 天没有新文章 | 主动提出 3 个选题供选择 |
| 用户提到一个技术话题 | 主动问是否要写成文章 |
| 发现重大技术新闻 | 主动提案写分析/教程 |
| 每次写完文章 | 立即推送 Git，无需再问 |

---

## 八、文章嵌入 PPT 规范

当文章需要 PPT 演示时：

1. **创建 PPT 文件**：`assets/slides/主题.html`
2. **使用 Reveal.js CDN**：
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.6.1/dist/reveal.css">
<script src="https://cdn.jsdelivr.net/npm/reveal.js@4.6.1/dist/reveal.js"></script>
```
3. **PPT 页面结构**（建议 10-15 页）：
   - 封面：标题 + 作者
   - 背景/痛点
   - 核心概念
   - 架构图
   - 代码演示（2-3 页）
   - 实战步骤
   - 常见问题
   - 总结
4. **嵌入文章**：在文章适当位置加入 iframe

---

## 九、作者署名规范

**所有新文章必须包含**：

Front Matter：`author: 牛马便利店一号店员`

文章末尾：
```markdown
*作者：牛马便利店一号店员*
```

---

## 十、常见场景处理

### 场景 1：用户给了明确主题
```
用户："写一篇关于 XXX 的文章"
↓
调研 → 写文章 → 推送 Git
（无需再问，直接执行）
```

### 场景 2：用户让选主题
```
用户："你选一个主题写"
↓
从选题库选 3 个候选 → 告知用户选择 → 用户确认后执行
或：直接写最合适的那篇，告知用户
```

### 场景 3：用户问有什么可以写的
```
↓
搜索近期热点 → 提出 3 个具体选题 → 用户选择一个
↓
执行
```

### 场景 4：用户问某个技术问题
```
↓
先回答问题
↓
如果问题值得展开 → 问："要写成文章吗？"
↓
用户同意 → 写文章 → 推送
```

---

## 十一、输出格式标准

### 文章标题
- 简洁明了，包含核心关键词
- 示例：`Docker 进阶：从开发到生产级集群部署`

### commit 消息
- 格式：`type: 简短描述`
- type：feat / fix / chore / docs / perf
- 示例：`feat: 新增 Redis 缓存设计与踩坑实录`

### GitHub 推送后
- 告知用户文章标题和链接
- 简述文章核心内容
- 列出文章覆盖的主要知识点

---

*本手册由「牛马便利店一号店员」编写，用于标准化博客运营流程。*
*最后更新：2026-04-12*
