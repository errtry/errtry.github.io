---
layout: post
title: Reveal.js - 让网页演示变得优雅
categories: [Tech]
description: 详细介绍 Reveal.js 演示框架的使用方法、核心功能和注意事项
keywords: Reveal.js, 演示文稿, HTML, 幻灯片, presentation
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 Reveal.js 是一个基于 HTML/CSS/JavaScript 的开源演示框架，可以让你用网页技术做出漂亮的幻灯片。

## 什么是 Reveal.js？

Reveal.js 由 [Hakim El Hattab](https://twitter.com/hakimel) 创建，是一个用于构建演示文稿的 HTML 框架。与传统 PPT 不同，它是纯代码驱动的——你只需要写 HTML 或 Markdown，就能创建支持动画、代码高亮、演讲者视图的幻灯片。

**官网**：https://revealjs.com/  
**GitHub**：https://github.com/hakimel/reveal.js

## 主要特性

| 特性 | 说明 |
|------|------|
| 🎨 **多种主题** | 内置多种精美主题，也可自定义 |
| 📝 **Markdown 支持** | 用 Markdown 写幻灯片，更专注内容 |
| ✨ **过渡动画** | slide、fade、zoom、convex 等多种效果 |
| 💻 **代码高亮** | 内置 highlight.js，支持多种语言 |
| 🎯 **演讲者视图** | 按 `S` 键打开，包含计时器和备注 |
| 📱 **响应式** | 手机、平板、桌面都能完美展示 |
| 📄 **PDF 导出** | 可导出为 PDF 打印或分享 |
| 🔌 **插件生态** | 支持搜索、全屏、缩放等扩展 |

## 快速开始

### 方式一：CDN 引入（最简单）

创建一个 HTML 文件，引入 Reveal.js 即可：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>我的演示</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/theme/white.css">
</head>
<body>
  <div class="reveal">
    <div class="slides">
      <!-- 幻灯片内容 -->
      <section>
        <h1>Hello Reveal.js</h1>
        <p>我的第一页幻灯片</p>
      </section>
      <section>
        <h2>第二页</h2>
        <ul>
          <li>支持 Markdown</li>
          <li>代码高亮</li>
          <li>动画效果</li>
        </ul>
      </section>
    </div>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/reveal.js"></script>
  <script>
    Reveal.initialize();
  </script>
</body>
</html>
```

### 方式二：npm 安装

```bash
npm install reveal.js
```

```javascript
import Reveal from 'reveal.js';
import 'reveal.js/dist/reveal.css';
import 'reveal.js/dist/theme/white.css';

const deck = new Reveal(document.querySelector('.reveal'), {
  hash: true,
  transition: 'slide'
});
deck.initialize();
```

## 核心功能详解

### 📝 使用 Markdown 写幻灯片

Reveal.js 原生支持 Markdown，只需在 `<section>` 标签上添加 `data-markdown` 属性：

```html
<section data-markdown>
  <textarea data-template>
    ## 第一页标题
    
    - 列表项 1
    - 列表项 2
    
    ---
    
    ## 第二页
    上下两个 `---` 分隔幻灯片
  </textarea>
</section>
```

也可以单独放一个 Markdown 文件：

```html
<section data-markdown="slides.md" 
         data-separator="^\n---\n$" 
         data-separator-vertical="^\n--\n$">
</section>
```

### 🎨 主题选择

内置主题（通过 CDN 引入）：

| 主题 | CSS |
|------|-----|
| black | `dist/theme/black.css` |
| white | `dist/theme/white.css` |
| league | `dist/theme/league.css` |
| beige | `dist/theme/beige.css` |
| sky | `dist/theme/sky.css` |
| night | `dist/theme/night.css` |
| serif | `dist/theme/serif.css` |
| simple | `dist/theme/simple.css` |
| solarized | `dist/theme/solarized.css` |

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@5.1.0/dist/theme/night.css">
```

### ✨ 过渡动画

```javascript
Reveal.initialize({
  transition: 'slide',      // none/fade/slide/convex/concave/zoom
  transitionSpeed: 'default' // default/slow/fast
});
```

也可以在单个幻灯片上指定：

```html
<section data-transition="zoom">
  <h2>缩放过渡</h2>
</section>
```

### 💻 代码高亮

Reveal.js 内置了 highlight.js，代码块会自动高亮：

```html
<section>
  <pre><code class="language-python">
def hello():
    print("Hello, World!")
  </code></pre>
</section>
```

支持的语言：JavaScript、Python、Java、C++、Go、Ruby、SQL、HTML、CSS 等上百种。

### 🎭 Fragments（逐步显示）

Fragments 可以让列表项、段落等逐个显示，增加演示节奏感：

```html
<section>
  <ul>
    <li class="fragment">第一步</li>
    <li class="fragment">第二步</li>
    <li class="fragment">第三步</li>
  </ul>
</section>
```

可用类名：
- `fragment fade-out` - 渐出
- `fragment fade-up` / `fade-down` / `fade-left` / `fade-right`
- `fragment grow` - 放大
- `fragment shrink` - 缩小
- `fragment highlight-red` - 高亮红色

### 🖼️ 背景设置

```html
<!-- 纯色背景 -->
<section data-background-color="#ff0000">

<!-- 图片背景 -->
<section data-background-image="bg.jpg">

<!-- 视频背景 -->
<section data-background-video="video.mp4">

<!-- 渐变背景 -->
<section data-background-gradient="linear-gradient(to bottom, #1a1a1a, #4a4a4a)">
```

### 📺 演讲者视图

按 `S` 键打开演讲者视图，可以看到：
- 当前幻灯片预览
- 下一张幻灯片预览
- 演讲者备注
- 计时器

添加备注的方式：

```html
<section>
  <h2>标题</h2>
  <aside class="notes">
    这里是你看不到的演讲者备注...
  </aside>
</section>
```

### 🔍 全览模式

按 `ESC` 键进入全览模式，可以看到所有幻灯片的缩略图，方便快速跳转。

## 常用配置

```javascript
Reveal.initialize({
  // 基础配置
  hash: true,                    // URL 哈希跟踪幻灯片
  controls: true,                // 显示导航箭头
  progress: true,                // 显示进度条
  center: true,                  // 居中幻灯片
  width: 960,                    // 幻灯片宽度
  height: 540,                   // 幻灯片高度
  margin: 0.04,                 // 边距
  slideNumber: true,             // 显示页码

  // 过渡和动画
  transition: 'slide',            // 过渡效果
  transitionSpeed: 'default',     // 过渡速度
  backgroundTransition: 'fade',  # 背景过渡

  // 功能开关
  keyboard: true,                // 键盘控制
  overview: true,                # ESC 全览
  shuffle: false,                # 随机顺序
  loop: false,                   # 循环播放

  // PDF 导出
  pdfSeparateFragments: false,   # PDF 中包含 fragment
});
```

## 嵌入网页的几种方式

### 方式一：iframe 嵌入（推荐）

```html
<iframe 
  src="https://example.com/slides.html" 
  width="100%" 
  height="500" 
  frameborder="0"
  allowfullscreen>
</iframe>
```

### 方式二：直接嵌入当前页面

直接把 Reveal.js 的 HTML 结构嵌入页面即可，就像本文配合 BadUSB 文章一样。

### 方式三：单页应用模式

```javascript
Reveal.initialize({
  embedded: true,  // 嵌入模式
  hash: false      // 禁用哈希
});
```

## 注意事项

### ⚠️ 国内访问

Reveal.js 默认使用 CDN（jsdelivr），国内访问速度尚可。如果需要更好的体验，可以：

1. **本地化**：下载 Reveal.js 到本地服务器
2. **国内镜像**：使用国内 CDN（如 bootcdn）
3. **打包**：用 webpack/vite 打包到项目

```html
<!-- 国内 CDN -->
<link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/reveal.js/5.1.0/reset.min.css">
<link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/reveal.js/5.1.0/reveal.min.css">
<link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/reveal.js/5.1.0/theme/white.min.css">
<script src="https://cdn.bootcdn.net/ajax/libs/reveal.js/5.1.0/reveal.min.js"></script>
```

### ⚠️ 浏览器兼容性

Reveal.js 5.x 支持现代浏览器：
- Chrome 80+
- Firefox 75+
- Safari 14+
- Edge 80+

不支持 IE 11。

### ⚠️ 性能问题

1. **大图片/视频**：建议压缩后再使用，或使用懒加载
2. **过多幻灯片**：每页控制在合理内容范围内
3. **复杂动画**：注意不要过度使用，可能导致卡顿

### ⚠️ 无障碍访问

Reveal.js 对屏幕阅读器的支持有限。如果需要考虑无障碍：
- 使用语义化的 HTML 结构
- 为图片添加 alt 文字
- 避免仅依赖颜色传递信息

### ⚠️ 移动端体验

虽然 Reveal.js 支持触屏操作，但建议：
- 移动端使用更简单的过渡效果（如 `fade`）
- 避免过小的字体
- 考虑为移动端提供单独版本

### ⚠️ 版权和许可

Reveal.js 基于 MIT 许可证，可以免费商用。但注意：
- 部分主题可能包含第三方字体
- 如果使用商业项目，请确认符合许可证要求

## 进阶技巧

### 自动播放

```javascript
Reveal.configure({
  autoSlide: 5000,    // 每 5 秒自动翻页
  autoSlideStoppable: false
});
```

### 跳转指定页

```javascript
// 跳转到第 3 张（从 0 开始）
Reveal.slide(2);

// 跳转到第 2 张幻灯片的第 3 个垂直幻灯片
Reveal.slide(1, 2);
```

### 监听事件

```javascript
Reveal.on('slidechanged', event => {
  console.log('当前幻灯片:', event.indexh);
});

Reveal.on('fragmentshown', event => {
  console.log('Fragment 展示');
});
```

### 打印为 PDF

```bash
# 添加 ?print-pdf 到 URL
https://example.com/slides.html?print-pdf

# 浏览器打印（Chrome 效果最好）
# Ctrl+P → 目标打印机 → 保存为 PDF
```

或配置自动导出：

```javascript
Reveal.initialize({
  pdfSeparateFragments: false
});
```

## 总结

Reveal.js 是一个强大而灵活的演示框架，特别适合：

- ✅ 技术分享和演示
- ✅ 文档和教程
- ✅ 在线课程和培训
- ✅ 个人作品集展示

相比传统 PPT，它的优势在于：
- 版本控制友好（纯文本）
- 易于分享和维护
- 支持 Markdown
- 可嵌入网页

唯一需要适应的是纯代码驱动的方式，但一旦熟悉，就能做出非常优雅的演示文稿。

---

## 参考链接

- [Reveal.js 官网](https://revealjs.com/)
- [GitHub 仓库](https://github.com/hakimel/reveal.js)
- [在线编辑器 slides.com](https://slides.com/)
- [官方示例演示](https://revealjs.com/?demo)
