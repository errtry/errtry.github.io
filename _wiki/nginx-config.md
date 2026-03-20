---
layout: wiki
title: Nginx 常用配置
cate1: Nginx
cate2: Web 服务器
description: Nginx 常用配置速查，包含反向代理、负载均衡、静态站点、HTTPS
keywords: Nginx, 反向代理, 负载均衡, 配置, Web服务器
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 配置文件：`/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/`

## 🏠 基础配置

```nginx
# 运行用户
user www-data;

# 工作进程数（auto 为 CPU 核心数）
worker_processes auto;

# 错误日志
error_log /var/log/nginx/error.log;

# PID 文件
pid /run/nginx.pid;

events {
    worker_connections 1024;  # 单进程最大连接数
    use epoll;                 # Linux 高性能模式
    multi_accept on;
}

http {
    # 基础设置
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # 性能优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
}
```

## 📂 静态站点

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;

    # 访问日志
    access_log /var/log/nginx/example.log;

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # 默认请求处理
    location / {
        try_files $uri $uri/ =404;
    }
}
```

## 🔄 反向代理

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        
        # 代理头设置
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # 缓冲设置
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }
}
```

## ⚖️ 负载均衡

```nginx
upstream backend {
    least_conn;              # 最少连接数
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080 backup;  # 备用
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🔒 HTTPS/SSL 配置

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL 证书
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # HSTS（强制 HTTPS）
    add_header Strict-Transport-Security "max-age=31536000" always;
}

# HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

## 🌐 路径匹配规则

| 前缀 | 说明 | 示例 |
|------|------|------|
| `location /` | 通用匹配 | 所有请求 |
| `location = /` | 精确匹配 | 仅根路径 |
| `location ^~ /api/` | 前缀匹配，不正则 | /api/* |
| `location ~ /api/` | 正则区分大小写 | /api/*
| `location ~* \.jpg$` | 正则不区分大小写 | .jpg |
| `location /images/` | 普通前缀 | /images/* |

## 📜 常用指令

```nginx
# 重写 URL
rewrite ^/old/(.*)$ /new/$1 permanent;  # 301 重定向
rewrite ^/old/(.*)$ /new/$1 redirect;  # 302 重定向

# 条件判断
if ($http_user_agent ~* "Mobile") {
    # 移动端处理
}

# 限流
limit_req_zone $binary_remote_addr zone=req_limit:10m rate=10r/s;

location / {
    limit_req zone=req_limit burst=20 nodelay;
}

# 禁止 IP 访问
location / {
    deny 192.168.1.100;
    allow all;
}
```

## 🐳 Docker 常用命令

```bash
# 启动 Nginx
docker run -d -p 80:80 -v /data/nginx.conf:/etc/nginx/nginx.conf nginx

# 重新加载配置
docker exec -it nginx nginx -s reload

# 测试配置
docker exec -it nginx nginx -t
```
