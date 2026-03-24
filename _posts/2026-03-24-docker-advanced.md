---
layout: post
title: Docker 进阶：从开发到生产级集群部署
categories: [DevOps]
description: 详细讲解 Docker Compose、Docker Swarm、镜像优化、生产级部署实践
keywords: Docker, Docker Compose, Docker Swarm, 容器编排, 部署
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇 Docker 进阶指南，覆盖从本地开发到生产集群部署的完整链路

## 前置知识

本文假设你已经：
- 会用 `docker run`、`docker ps`、`docker exec` 等基础命令
- 了解镜像、容器、网络、数据卷的概念
- 有过 Docker 开发经验

如果还没用过 Docker，先去看官方入门文档。

---

## 一、用 Docker Compose 编排多容器

单个容器用 `docker run`，多个容器协作就得用 **Docker Compose**。

### 一个真实案例：Node.js + MongoDB + Redis

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongo
      - redis
    restart: unless-stopped
    networks:
      - backend

  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped
    networks:
      - backend

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - backend
    command: redis-server --appendonly yes

volumes:
  mongo_data:
  redis_data:

networks:
  backend:
    driver: bridge
```

关键点说明：

| 配置 | 作用 |
|------|------|
| `depends_on` | 启动顺序，确保依赖服务先启动 |
| `restart: unless-stopped` | 容器异常退出后自动重启 |
| `volumes` | 持久化数据，容器删除不丢失 |
| `networks` | 网络隔离，不同 stack 互不影响 |
| `command` | 覆盖默认启动命令 |

### 常用 Compose 命令

```bash
# 启动（-d 后台运行）
docker-compose up -d

# 带构建启动
docker-compose up -d --build

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f app
docker-compose logs --tail=100 app

# 停止并删除
docker-compose down

# 停止并删除数据卷（清空数据！）
docker-compose down -v

# 重新构建（不看缓存）
docker-compose build --no-cache
```

### 多环境配置

```yaml
# .env 文件
NODE_ENV=production
MONGODB_URI=mongodb://mongo:27017/myapp

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    environment:
      - NODE_ENV=${NODE_ENV}
      - MONGODB_URI=${MONGODB_URI}
    # 生产环境限制资源
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

```bash
# 使用生产配置
docker-compose -f docker-compose.prod.yml up -d
```

---

## 二、构建优化：让镜像小 10 倍

镜像大的问题：
- 构建慢
- 部署慢
- 占用磁盘
- 拉取失败率高

### 多阶段构建

```dockerfile
# Stage 1: 构建阶段
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Stage 2: 运行阶段
FROM node:20-alpine AS runtime
WORKDIR /app

# 只复制构建产物
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# 创建非 root 用户
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

效果对比：

| 方式 | 镜像大小 | 构建时间 |
|------|----------|----------|
| 单阶段 `node:20` | ~900MB | 3分钟 |
| 单阶段 `node:20-alpine` | ~170MB | 2分钟 |
| 多阶段 `alpine` | ~50MB | 2分钟 |

### .dockerignore

```gitignore
node_modules
.git
.env*
dist
*.log
README.md
coverage
.vscode
.DS_Store
```

### 并行构建

```bash
# Docker BuildKit 加速
export DOCKER_BUILDKIT=1
docker-compose build --parallel
```

---

## 三、Docker Swarm：开箱即用的集群

不需要 Kubernetes 就能跑集群。

### 初始化 Swarm

```bash
# 在管理节点执行
docker swarm init

# 查看节点
docker node ls

# 获取 worker token（其他机器加入用）
docker swarm join-token worker

# 获取 manager token
docker swarm join-token manager
```

### 部署服务

```yaml
# docker-stack.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        max_attempts: 3
    placement:
      constraints:
        - "node.role==worker"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  api:
    image: myregistry.com/myapp:latest
    ports:
      - "8080:8080"
    environment:
      - ENV=production
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
    networks:
      - backend

networks:
  backend:
    driver: overlay
```

```bash
# 部署 stack
docker stack deploy -c docker-stack.yml myapp

# 查看 stack
docker stack ls
docker stack ps myapp

# 扩缩容
docker service scale myapp_web=5

# 更新服务
docker service update --image myapp:v2 myapp_api

# 回滚
docker service rollback myapp_api

# 删除 stack
docker stack rm myapp
```

### 健康检查与滚动更新

```yaml
services:
  api:
    image: myapp:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      update_config:
        parallelism: 1          # 每次更新 1 个
        delay: 10s              # 间隔 10 秒
        failure_action: pause   # 失败暂停
        monitor: 30s            # 监控 30 秒
```

---

## 四、生产环境必做清单

### 1. 资源限制（必须加）

```yaml
deploy:
  resources:
    limits:
      cpus: '1.0'
      memory: 512M
    reservations:
      cpus: '0.25'
      memory: 128M
```

### 2. 日志管理

```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"    # 单文件最大 10MB
        max-file: "3"       # 最多 3 个文件
```

### 3. 安全：非 root 用户

```dockerfile
RUN addgroup -S app && adduser -S app -G app
USER app
```

### 4. 信号处理（优雅关闭）

容器收到 `SIGTERM` 时需要优雅退出，而不是被 `SIGKILL` 强杀：

```dockerfile
# 使用 exec 表单式，PID=1 收到信号
CMD ["node", "server.js"]
# 不要用：CMD node server.js（shell 表单会丢失信号）
```

Node.js 示例：

```javascript
process.on('SIGTERM', () => {
  console.log('收到 SIGTERM，准备关闭...');
  server.close(() => {
    console.log('关闭完成');
    process.exit(0);
  });
});
```

### 5. 健康检查

```yaml
healthcheck:
  test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
  interval: 30s
  timeout: 5s
  retries: 3
```

---

## 五、监控与排查

### 查看资源使用

```bash
# 实时资源
docker stats

# 限制输出
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### 排查问题

```bash
# 查看容器详情
docker inspect container_name

# 查看网络
docker network inspect backend

# 查看磁盘占用
docker system df

# 清理未使用资源
docker system prune -a
```

### 推荐监控方案

| 工具 | 用途 |
|------|------|
| cAdvisor | 容器级别监控 |
| Prometheus + Grafana | 指标收集和可视化 |
| Loki + Grafana | 日志收集 |
| Docker Swarm Visualizer | 集群可视化 |

---

## 总结

生产级 Docker 部署的关键点：

1. ✅ **多阶段构建** — 镜像小 10 倍
2. ✅ **资源限制** — 防止容器占满机器
3. ✅ **健康检查** — 自动发现异常容器
4. ✅ **滚动更新** — 零停机部署
5. ✅ **日志轮转** — 防止磁盘写满
6. ✅ **非 root 用户** — 安全加固
7. ✅ **优雅关闭** — 不丢请求

下一步可以学 **Kubernetes**，但 Docker Swarm 对中小项目完全够用，先把它用熟再说。

---

*作者：牛马便利店一号店员*
