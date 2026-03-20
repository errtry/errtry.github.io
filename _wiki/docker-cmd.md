---
layout: wiki
title: Docker 常用命令
cate1: Docker
cate2: 容器技术
description: Docker 常用命令速查，包含镜像、容器、网络、数据卷操作
keywords: Docker, 容器, DockerHub, 镜像, 容器化
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 Docker 镜像站（国内加速）：`https://docker.mirrors.ustc.edu.cn/` 或 `https://mirror.ccsggg.com/`

## 🔧 基础操作

```bash
# 查看版本
docker version
docker info

# 登录 Docker Hub
docker login
docker logout
```

## 🖼️ 镜像操作

```bash
# 搜索镜像
docker search ubuntu

# 拉取镜像
docker pull ubuntu:latest
docker pull ubuntu:22.04

# 查看镜像列表
docker images
docker image ls
docker images -a

# 删除镜像
docker rmi ubuntu
docker rmi image_id
docker rmi $(docker images -q)     # 删除所有镜像

# 构建镜像
docker build -t myapp:latest .
docker build -t myapp:latest -f Dockerfile.dev .

# 标记镜像（推送到私有仓库）
docker tag myapp:latest registry.example.com/myapp:latest

# 推送镜像
docker push registry.example.com/myapp:latest

# 清理未使用镜像
docker image prune -a
```

## 📦 容器操作

```bash
# 运行容器
docker run -d nginx                    # 后台运行
docker run -it ubuntu bash             # 交互式
docker run -p 8080:80 nginx            # 端口映射
docker run -v /data:/data nginx        # 挂载卷
docker run --name mynginx nginx        # 指定容器名
docker run --restart=always nginx     # 自动重启
docker run -e VAR=value nginx          # 环境变量
docker run --network mynet nginx       # 指定网络

# 查看容器
docker ps                              # 运行中
docker ps -a                            # 所有容器
docker ps -l                            # 最近容器

# 启动/停止/重启容器
docker start mynginx
docker stop mynginx
docker restart mynginx

# 进入容器
docker exec -it mynginx bash
docker exec -it mynginx sh
docker attach mynginx                   # 进入正在运行的（共享终端）

# 查看容器日志
docker logs mynginx
docker logs -f mynginx                  # 实时
docker logs --tail 100 mynginx          # 最近 100 行

# 查看容器详情
docker inspect mynginx

# 删除容器
docker rm mynginx
docker rm -f mynginx                    # 强制删除（运行中）
docker rm $(docker ps -aq)              # 删除所有容器
```

## 📊 容器其他操作

```bash
# 复制文件
docker cp mynginx:/etc/nginx/nginx.conf ./
docker cp ./file mynginx:/tmp/

# 查看容器进程
docker top mynginx

# 查看资源使用
docker stats
docker stats mynginx

# 查看容器 IP
docker inspect -f '{{.NetworkSettings.IPAddress}}' mynginx

# 暂停/恢复容器
docker pause mynginx
docker unpause mynginx
```

## 🕸️ 网络操作

```bash
# 查看网络
docker network ls

# 创建网络
docker network create mynet
docker network create -d bridge mynet

# 查看网络详情
docker network inspect mynet

# 连接/断开容器
docker network connect mynet mynginx
docker network disconnect mynet mynginx
```

## 💾 数据卷操作

```bash
# 查看卷
docker volume ls

# 创建卷
docker volume mydata

# 查看卷详情
docker volume inspect mydata

# 删除卷
docker volume rm mydata
docker volume prune                        # 清理未使用
```

## 🛠️ Docker Compose

```bash
# 启动服务
docker-compose up -d
docker-compose up -d --build              # 重新构建

# 停止服务
docker-compose down
docker-compose down -v                    # 同时删除卷

# 查看服务
docker-compose ps

# 查看日志
docker-compose logs -f

# 重启服务
docker-compose restart

# 构建镜像
docker-compose build
```

## 📝 Dockerfile 示例

```dockerfile
# 基础镜像
FROM node:18-alpine

# 设置工作目录
WORKDIR /app

# 复制文件
COPY package*.json ./
RUN npm install

COPY . .

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["npm", "start"]
```

## ⚡ 常用技巧

```bash
# 一键清理
docker system prune -a                    # 清理所有未使用数据
docker system prune --volumes              # 包括卷

# 查看 Docker 占用
docker system df

# 给容器指定内存限制
docker run -m 512m nginx

# 限制 CPU
docker run --cpus=0.5 nginx

# 多平台构建
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# 查看实时事件
docker events
```

## 🔗 国内加速配置

```json
// /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

然后执行：
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
