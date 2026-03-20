---
layout: wiki
title: GitHub Actions 常用命令
cate1: DevOps
cate2: CI/CD
description: GitHub Actions 工作流配置，包含语法、常用 action、示例
keywords: GitHub Actions, CI, CD, 自动化, GitHub
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 工作流文件放在 `.github/workflows/` 目录下

## 📁 基本结构

```yaml
name: CI 流程名称

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout 代码
        uses: actions/checkout@v4
      
      - name: 运行步骤
        run: echo "Hello World"
```

## 🏃 运行环境 (runs-on)

| 环境 | 说明 |
|------|------|
| `ubuntu-latest` | Ubuntu 最新 LTS |
| `ubuntu-22.04` / `ubuntu-20.04` | 指定版本 |
| `windows-latest` | Windows 最新 |
| `macos-latest` | macOS 最新 |
| `macos-14` | macOS M1 |

## 📦 常用 Actions

```yaml
# 检出代码
- uses: actions/checkout@v4

# 设置 Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'

# 设置 Python
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'

# 缓存依赖
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}

# 上传构建产物
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: ./dist

# 下载构建产物
- uses: actions/download-artifact@v4
  with:
    name: my-artifact

# 发送通知
- uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
```

## 🛠️ 常用命令 (run)

```yaml
# 单行命令
- run: npm install

# 多行命令
- name: 安装依赖
  run: |
    npm install
    npm run build

# 设置环境变量
- name: 设置环境变量
  run: echo "VERSION=1.0.0" >> $GITHUB_ENV

# 设置输出
- name: 设置输出
  run: echo "date=$(date)" >> $GITHUB_OUTPUT
```

## 🔐  Secrets 和变量

```yaml
# 使用 Secret
- run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

# 使用环境变量
- run: echo ${{ vars.API_URL }}
```

## ⏰ 触发条件 (on)

```yaml
on:
  # push 时触发
  push:
    branches: [main, master]
    tags: ['v*']
    paths: ['src/**', '*.js']
  
  # PR 时触发
  pull_request:
    branches: [main]
    types: [opened, synchronize, closed]
  
  # 定时触发
  schedule:
    - cron: '0 0 * * *'  # 每天 UTC 0 点
  
  # 手动触发
  workflow_dispatch:
    inputs:
      version:
        description: '版本号'
        required: true
        default: '1.0.0'
  
  # 其他仓库触发
  repository_dispatch:
    types: [update]
```

## 🔀 矩阵策略

```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

## 🔒 权限配置

```yaml
permissions:
  contents: read      # 读取仓库
  pages: write        # 部署 Pages
  id-token: write     # OIDC 认证
```

## 📡 常用工作流示例

### Node.js CI

```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    
    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

### Docker Build & Push

```yaml
name: Docker Build

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: 登录 Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: user/repo:latest
```

### 定时备份

```yaml
name: Daily Backup

on:
  schedule:
    - cron: '0 2 * * *'  # 每天凌晨 2 点

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - name: 打包
        run: tar -czf backup.tar.gz ./data
      
      - name: 上传到 NAS
        run: |
          curl -u ${{ secrets.NAS_USER }}:${{ secrets.NAS_PASS }} \
            -T backup.tar.gz http://nas.example.com/backup/
```

## 🎯 常用表达式

```yaml
# 条件判断
if: github.event_name == 'push'

# 获取提交信息
${{ github.event.head_commit.message }}

# 获取分支
${{ github.ref_name }}

# 获取时间
${{ github.event.created_at }}

# Runner 信息
${{ runner.os }}
${{ runner.arch }}
```
