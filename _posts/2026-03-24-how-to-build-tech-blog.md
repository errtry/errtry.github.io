---
layout: post
title: 从零开始搭建个人技术博客指南
categories: [Tech]
description: 详细介绍了如何使用 GitHub Pages + Jekyll 从零搭建个人技术博客，包括主题定制、域名绑定、SEO 优化等
keywords: 技术博客, GitHub Pages, Jekyll, 博客搭建, 个人网站
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇从零搭建技术博客的完整指南，适合想要建立个人品牌或分享技术内容的朋友。

## 为什么需要个人技术博客？

在开始之前，先回答一个问题：为什么我们需要个人技术博客？

- **沉淀知识** — 写下来才是真正的学会
- **个人品牌** — 技术圈的个人名片
- **帮助他人** — 你的踩坑经验可能救别人一命
- **求职加分** — 面试官眼中的亮点
- **记录成长** — 回头看自己进步的过程很有趣

---

## 方案选择

### 为什么选 GitHub Pages + Jekyll？

| 方案 | 优点 | 缺点 |
|------|------|------|
| WordPress | 功能强大 | 需要服务器、维护成本高 |
| Hexo | 速度快、主题多 | 需要 Node.js 环境 |
| **GitHub Pages + Jekyll** | 免费、静态、无需服务器、版本控制 | 国内访问略慢、需要懂一点 Git |
| Notion + Super | 写作体验好 | 功能有限、免费版有限制 |

**本文选择 Jekyll**，因为：
1. GitHub 原生支持，无需配置
2. 纯静态页面，加载速度快
3. 支持 Markdown 写作
4. 主题丰富，易于定制

---

## 搭建步骤

### 1. 准备环境

你需要安装：

```bash
# 安装 Git
# Windows: https://git-scm.com/download/win
# macOS: brew install git
# Linux: sudo apt install git

# 安装 Ruby (Jekyll 需要)
# Windows: https://rubyinstaller.org/
# macOS: 已预装，或 brew install ruby
# Linux: sudo apt install ruby ruby-dev

# 安装 Jekyll
gem install jekyll bundler
```

> 💡 **Windows 用户**：推荐使用 [RubyInstaller](https://rubyinstaller.org/)，安装时勾选 "Add Ruby executables to your PATH"

### 2. 创建 GitHub 仓库

1. 登录 [GitHub](https://github.com/)
2. 点击右上角 `+` → `New repository`
3. Repository name 填写：`yourname.github.io`
4. 选择 `Public`
5. 点击 `Create repository`

> ⚠️ 必须用 `yourname.github.io` 这个格式，才能通过 `https://yourname.github.io` 访问

### 3. 初始化 Jekyll

```bash
# 克隆仓库
git clone https://github.com/yourname/yourname.github.io.git
cd yourname.github.io

# 创建 Jekyll 网站
jekyll new .

# 本地预览
bundle exec jekyll serve
# 访问 http://localhost:4000
```

### 4. 选择主题

Jekyll 有丰富的主题可以选择：

| 主题 | 特点 | GitHub Stars |
|------|------|--------------|
| [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) | 功能最全，文档完善 | 12k+ |
| [Chirpy](https://chirpy.cotes.co/) | 现代化，支持暗色主题 | 5k+ |
| [TeXt Theme](https://tianqi.name/jekyll-TeXt-theme/) | 文档友好，适合团队 | 3k+ |
| [Jekyll Clean](https://scotte.github.io/jekyll-clean/) | 简洁干净 | 2k+ |

推荐使用 **Minimal Mistakes**，功能最全，社区活跃。

#### 安装主题（以 Minimal Mistakes 为例）

```bash
# 添加主题到 Gemfile
echo 'gem "minimal-mistakes-jekyll"' >> Gemfile

# 安装
bundle install

# 配置 _config.yml
theme: minimal-mistakes-jekyll
```

---

## 主题配置

### _config.yml 基础配置

```yaml
# 站点基本信息
title: 牛马假日
name: 你的名字
description: 技术分享与个人成长
url: https://yourname.github.io
repository: yourname/yourname.github.io

# 作者信息
author:
  name: 你的名字
  bio: 软件工程师 / 技术爱好者
  location: 中国
  email: your@email.com
  links:
    - title: GitHub
      url: https://github.com/yourname
    - title: Twitter
      url: https://twitter.com/yourname

# Jekyll 配置
timezone: Asia/Shanghai
markdown: kramdown
highlighter: rouge

# 分页
paginate: 5
paginate_path: "/page:num/"

# 社交分享（可选）
twitter:
  username: yourname
github:
  username: yourname
```

### 目录结构

```
├── _config.yml          # 配置文件
├── _posts/              # 博客文章
│   └── 2024-01-01-hello-world.md
├── _pages/              # 页面（如关于页面）
├── _layouts/            # 布局文件
├── _includes/          # 可复用组件
├── _drafts/            # 草稿（不公开发布）
├── assets/             # 静态资源（CSS、图片等）
├── images/             # 图片文件夹
└── index.html          # 首页
```

---

## 写文章

### 创建第一篇文章

在 `_posts` 目录下创建文件，命名格式：`年-月-日-标题.md`

```markdown
---
layout: post
title: 我的第一篇博客
categories: [Tech]
description: 这是我的第一篇博客
keywords: 博客, 入门
---

正文内容在这里...

## 二级标题

### 三级标题

代码示例：

```python
print("Hello World")
```
```

### Front Matter 说明

| 字段 | 说明 | 必填 |
|------|------|------|
| `layout` | 布局类型 | ✅ |
| `title` | 文章标题 | ✅ |
| `categories` | 分类 | ✅ |
| `description` | 描述（SEO 用） | 推荐 |
| `keywords` | 关键词 | 推荐 |
| `tags` | 标签 | 可选 |
| `mathjax` | 是否加载 MathJax | 可选 |
| `mermaid` | 是否加载 Mermaid | 可选 |

---

## 进阶功能

### 1. 添加评论系统

推荐使用 **Giscus**（基于 GitHub Discussions）：

```html
<!-- 在 _layouts/post.html 或文章底部添加 -->
<script src="https://giscus.app/client.js"
        data-repo="yourname/yourname.github.io"
        data-repo-id="xxx"
        data-category="Comments"
        data-category-id="xxx"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```

### 2. 添加搜索功能

使用 Algolia 或本地搜索：

```yaml
# _config.yml
search: true
search_full_content: true
```

### 3. SEO 优化

```yaml
# _config.yml
seo:
  title: 你的网站标题
  description: 网站描述
  thumbnail: /images/logo.png
  name: 你的名字
  domain: yourname.github.io
```

### 4. 添加百度统计

在 `_includes/head.html` 中添加：

```html
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?你的百度统计ID";
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
</script>
```

---

## 部署上线

### 方式一：GitHub Actions（推荐）

创建 `.github/workflows/jekyll.yml`：

```yaml
name: Deploy Jekyll Site

on:
  push:
    branches: [master]

jobs:
  jekyll:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
      
      - name: Build and Deploy
        uses: jeffreytse/jekyll-deploy-action@v0.1.0
        env:
          JEKYLL_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          JEKYLL_BUILD_OPTIONS: --config _config.yml
```

### 方式二：手动部署

```bash
# 构建静态文件
jekyll build

# _site 目录就是构建结果
# 手动上传到 GitHub Pages
```

---

## 域名绑定（可选）

### 购买域名

推荐在国内服务商购买：
- 阿里云万网
- 腾讯云 DNSPod

### 配置 DNS

添加两条记录：

| 类型 | 主机记录 | 记录值 |
|------|----------|--------|
| CNAME | @ | yourname.github.io |
| CNAME | www | yourname.github.io |

### 在 GitHub 添加自定义域名

1. 打开仓库设置 → Pages
2. 在 Custom domain 输入你的域名
3. 勾选 Enforce HTTPS

### 配置 Jekyll

```yaml
# _config.yml
url: "https://yourdomain.com"
baseurl: ""
```

---

## 常用插件

| 插件 | 用途 |
|------|------|
| [Jekyll SEO Tag](https://github.com/jekyll/jekyll-seo-tag) | SEO 优化 |
| [Jekyll Sitemap](https://github.com/jekyll/jekyll-sitemap) | 生成 sitemap.xml |
| [Jekyll Feed](https://github.com/jekyll/jekyll-feed) | RSS 订阅 |
| [MathJax](https://github.com/p光华/ jekyll-mathjax) | 数学公式 |
| [Mermaid](https://github.com/yFMcQ/jekyll-mermaid) | 流程图 |

安装插件：

```ruby
# Gemfile
group :jekyll_plugins do
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
  gem "jekyll-feed"
  gem "jekyll-paginate"
end
```

```yaml
# _config.yml
plugins:
  - jekyll-seo-tag
  - jekyll-sitemap
  - jekyll-feed
  - jekyll-paginate
```

---

## 写作技巧

### 1. 使用图片

```markdown
![图片描述](/images/post-image.jpg)
```

建议：
- 图片压缩后再上传
- 使用图床（如 GitHub 本身）减轻仓库体积
- 添加 alt 文字便于 SEO

### 2. 代码高亮

Jekyll 内置支持多种语言：

````markdown
```python
def hello():
    print("Hello")
```
````

### 3. 使用 Mermaid 流程图

```mermaid
graph TD
    A[开始] --> B{判断}
    B -->|是| C[执行]
    B -->|否| D[结束]
```

### 4. 数学公式

开启 MathJax：

```yaml
mathjax: true
```

使用：

```latex
$$E = mc^2$$
```

---

## 持续优化

### 加载速度优化

1. **图片优化** — 使用 WebP 格式，压缩图片
2. **CDN 加速** — 使用 jsDelivr 等 CDN
3. **减少请求** — 合并 CSS/JS
4. **懒加载** — 图片延迟加载

```html
<img src="image.jpg" loading="lazy" alt="描述">
```

### 内容优化

1. 坚持定期更新
2. 注重内容质量
3. 添加合适的标签和分类
4. 积极参与社区互动

---

## 总结

搭建个人技术博客的关键步骤：

1. ✅ 选择 GitHub Pages + Jekyll 方案
2. ✅ 创建 GitHub 仓库并初始化
3. ✅ 选择和配置主题
4. ✅ 撰写第一篇文章
5. ✅ 添加评论和搜索功能
6. ✅ 配置域名（可选）
7. ✅ 持续更新内容

> 🚀 **最重要的是：开始写！** 不需要等到"完全准备好"才上线，先跑起来再迭代。

---

## 参考资源

- [Jekyll 官方文档](https://jekyllrb.com/)
- [Minimal Mistakes 主题文档](https://mmistakes.github.io/minimal-mistakes/)
- [GitHub Pages 文档](https://docs.github.com/en/pages)
- [GitHub Actions 入门](https://docs.github.com/en/actions/learn-github-actions)

---

*如果你觉得这篇文章有帮助，欢迎在评论区留言讨论！*
