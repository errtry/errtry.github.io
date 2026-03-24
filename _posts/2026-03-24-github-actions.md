---
layout: post
title: GitHub Actions 自动化工作流实战
categories: [DevOps]
description: 详细讲解 GitHub Actions 的配置与实战，包括 CI/CD、自动化部署、定时任务
keywords: GitHub Actions, CI/CD, 自动化, DevOps, GitHub
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 GitHub Actions 是 GitHub 内置的 CI/CD 工具，免费好用，这篇讲实战

## 基础概念

### 核心术语

| 概念 | 说明 |
|------|------|
| **Workflow** | 工作流，整个自动化流程 |
| **Job** | 任务，一组步骤 |
| **Step** | 步骤，具体执行的操作 |
| **Action** | 动作，可复用的步骤 |
| **Runner** | 运行器，执行任务的机器 |

### 目录结构

```
.github/
└── workflows/
    ├── ci.yml      # CI 流程
    ├── deploy.yml   # 部署流程
    └── schedule.yml # 定时任务
```

---

## 一、Hello World：最简 CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 安装依赖
        run: npm install
      
      - name: 运行测试
        run: npm test
```

push 代码后，打开 GitHub 仓库 → Actions 就能看到运行状态。

---

## 二、Node.js 项目 CI/CD

```yaml
name: Node.js CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18, 20, 22]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 设置 Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'  # 缓存 node_modules
      
      - name: 安装依赖
        run: npm ci  # 比 npm install 快
      
      - name: 运行测试
        run: npm test
      
      - name: 构建
        run: npm run build
      
      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: dist-${{ matrix.node-version }}
          path: dist/

  deploy:
    needs: build-and-test  # 依赖上面的 job
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # 只有 main 分支才部署
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 部署
        run: |
          echo "部署到生产环境..."
          # 实际部署命令
```

---

## 三、Docker 镜像构建和推送

```yaml
name: Docker Build & Push

on:
  push:
    branches: [main]
    tags:
      - 'v*'  # 打标签时触发，如 v1.0.0

jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 提取版本号
        id: vars
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
      
      - name: 设置 Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: 登录 Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: 构建并推送
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myuser/myapp:latest
            myuser/myapp:${{ steps.vars.outputs.VERSION }}
          cache-from: type=gha  # 使用 GitHub Actions 缓存
          cache-to: type=gha,mode=max

      - name: 生成摘要
        id: digest
        run: |
          echo "digest=$(docker buildx imagetools inspect myuser/myapp:latest --format '{{.Digest}}')" >> $GITHUB_OUTPUT
      
      - name: 输出摘要
        run: echo ${{ steps.digest.outputs.digest }}
```

### 添加 Secrets

1. GitHub 仓库 → Settings → Secrets and variables → Actions
2. 添加 `DOCKER_USERNAME` 和 `DOCKER_PASSWORD`

---

## 四、部署到服务器

### SSH 部署

```yaml
name: Deploy to Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 配置 SSH
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      
      - name: 部署
        env:
          HOST: ${{ secrets.SERVER_HOST }}
          USER: ${{ secrets.SERVER_USER }}
        run: |
          echo "${{ secrets.SSH_KNOWN_HOSTS }}" >> ~/.ssh/known_hosts
          ssh $USER@$HOST "
            cd /var/www/myapp &&
            git pull &&
            docker-compose down &&
            docker-compose up -d --build
          "
```

### 使用 RSYNC 部署

```yaml
- name: Sync files
  uses: Pendect/action-rsync@v1.1.1
  env:
    WHO: github-actions
    PAT: ${{ secrets.GITHUB_TOKEN }}
    OPTIONS: "-avz --delete"
    SOURCE: ./dist/
    TARGET: "${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }}:/var/www/html"
```

---

## 五、定时任务

```yaml
name: Scheduled Tasks

on:
  schedule:
    # UTC 时间，比北京时间慢 8 小时
    # 每天凌晨 2 点 = UTC 18:00
    - cron: '0 18 * * *'
    
    # 每周一早上 9 点 = UTC 1:00
    # - cron: '0 1 * * 1'
    
    # 每小时一次
    # - cron: '0 * * * *'

  # 也支持手动触发
  workflow_dispatch:
    inputs:
      logLevel:
        description: '日志级别'
        required: true
        default: 'warning'

jobs:
  backup:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 备份数据库
        run: |
          mysqldump -h ${{ secrets.DB_HOST }} \
            -u ${{ secrets.DB_USER }} \
            -p${{ secrets.DB_PASSWORD }} \
            mydb > backup.sql
          
      - name: 上传到 S3
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1
      
      - name: 上传文件
        run: |
          aws s3 cp backup.sql s3://my-bucket/backups/$(date +%Y%m%d).sql
```

---

## 六、缓存优化

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Node.js 缓存
      - name: 缓存 node_modules
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-npm-
      
      # Python 缓存
      - name: 缓存 pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      # 构建缓存
      - name: 缓存构建目录
        uses: actions/cache@v4
        with:
          path: dist/
          key: ${{ runner.os }}-build-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-build-
```

---

## 七、通知和报告

### 钉钉/企业微信通知

```yaml
- name: 钉钉通知
  if: always()
  uses: jooto/dingtalk-action@v1.0.0
  with:
    dingtalk-token: ${{ secrets.DINGTALK_TOKEN }}
    status: ${{ job.status }}
    message: |
      ${{ github.workflow }} ${{ job.status }}
      分支：${{ github.ref_name }}
      提交：${{ github.sha }}
      链接：${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

### Slack 通知

```yaml
- name: Slack 通知
  if: always()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "${{ github.workflow }}: ${{ job.status }}",
        "blocks": [{
          "type": "section",
          "text": {
            "type": "mrkdwn",
            "text": "*${{ github.workflow }}*\n状态: ${{ job.status }}"
          }
        }]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 八、常用配置模板

### Go 项目

```yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.21'
    cache: true
- run: go test ./...
```

### Python 项目

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
    cache: 'pip'
- run: pip install -r requirements.txt
- run: pytest
```

### 多平台构建

```yaml
jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: linux-amd64
          - os: macos-latest
            target: darwin-arm64
          - os: windows-latest
            target: windows-amd64
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - name: 构建
        run: go build -o myapp-${{ matrix.target }}
```

---

## 九、调试技巧

### 开启调试日志

仓库 Settings → Actions → 选择 Runner → 勾选 "Enable debug logging"

### 本地模拟运行

```bash
# 安装 act
brew install act

# 运行 workflows
act -l              # 列出所有 workflow
act                 # 运行默认 workflow
act -W .github/workflows/ci.yml
```

### 查看 Runner 日志

GitHub → Actions → 选择 workflow → 点击具体 run → 查看每个 step 的日志

---

## 总结

| 场景 | 关键配置 |
|------|----------|
| 简单 CI | checkout + npm + test |
| Node.js 多版本 | matrix + cache |
| Docker 推送 | setup-buildx + login-action + build-push |
| SSH 部署 | ssh-agent + 部署命令 |
| 定时任务 | schedule cron |
| 通知 | 钉钉/Slack webhook |

GitHub Actions 完全免费（公共仓库），私有仓库每月 2000 分钟。够个人项目用了。

---

*作者：errtry*
