---
layout: post
title: Nginx 性能调优：让网站快 10 倍的实战指南
categories: [DevOps]
description: 从基础配置到生产级优化，详细讲解 Nginx 性能调优的各个维度
keywords: Nginx, 性能优化, Web服务器, 高并发, 缓存
author: errtry
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇实战型 Nginx 调优指南，基于真实生产环境经验

## 先跑个基准测试

调优之前，先有个基线：

```bash
# 安装 ab 或 wrk
# macOS
brew install wrk

# 测试
wrk -t12 -c400 -d30s https://yourdomain.com

# 或者用 Apache Bench
ab -n 10000 -c 100 https://yourdomain.com/
```

记住这几个数字：**QPS（每秒请求）**、**平均响应时间**、**失败率**。调优后对比。

---

## 一、系统层面优化

### 1. 文件描述符

Linux 默认 1024，远不够用。

```bash
# 查看当前限制
ulimit -n

# 临时修改
ulimit -n 65535

# 永久修改 /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```

### 2. Nginx worker 进程数

```nginx
# /etc/nginx/nginx.conf

worker_processes auto;  # 自动等于 CPU 核心数
worker_rlimit_nofile 65535;

events {
    worker_connections 65535;  # 单 worker 最大连接数
    use epoll;                  # Linux 高性能模型
    multi_accept on;            # 一次接受多个连接
}
```

计算公式：
```
最大并发 = worker_processes × worker_connections
```

比如 4 核 CPU、65535 连接数 = 最多 26 万并发。

### 3. TCP 优化

```nginx
http {
    # 开启高效传输
    sendfile on;
    tcp_nopush on;      # 等待数据包填满再发送
    tcp_nodelay on;     # 禁用 Nagle 算法
    
    # 连接复用
    keepalive_timeout 65;
    keepalive_requests 10000;
    
    # 缓冲区
    client_body_buffer_size 16k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
    client_max_body_size 50m;
    
    # 超时
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
}
```

---

## 二、Gzip 压缩

压缩前后流量可能差 5-10 倍：

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;           # 小于 1KB 不压缩
    gzip_proxied any;               # 代理请求也压缩
    gzip_comp_level 6;              # 压缩级别 1-9，越高越省流量但耗 CPU
    gzip_buffers 16 8k;
    
    gzip_types 
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml
        application/xml+rss
        image/svg+xml;
    
    gzip_disable "MSIE [1-6]\.";    # IE6 不支持
}
```

验证压缩生效：

```bash
curl -I -H "Accept-Encoding: gzip" https://yourdomain.com
# 应该看到 Content-Encoding: gzip
```

---

## 三、静态资源缓存

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/html;

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;  # 日志关闭省 IO
    }
}
```

按类型设置不同缓存时间：

```nginx
# 图片缓存 30 天
location ~* \.(jpg|jpeg|png|gif|ico)$ {
    expires 30d;
}

# CSS/JS 缓存 7 天（加 hash 的可以更长）
location ~* \.(css|js)$ {
    expires 7d;
}

# 字体缓存 1 年（因为加 hash）
location ~* \.(woff2|woff|ttf|eot)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### 禁止缓存（动态页面）

```nginx
location ~ \.php$ {
    expires -1;
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

---

## 四、反向代理优化

### 基本配置

```nginx
upstream backend {
    server 127.0.0.1:8080;
    keepalive 64;  # 保持连接数
}

server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        
        # 代理头设置（必须加！）
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 性能
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # 缓冲
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
        
        # 缓存（可选）
        proxy_cache my_cache;
    }
}
```

### 代理缓存

```nginx
http {
    # 缓存配置
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

    server {
        location / {
            proxy_cache my_cache;
            proxy_cache_valid 200 60m;        # 200 响应缓存 60 分钟
            proxy_cache_valid 404 1m;         # 404 缓存 1 分钟
            proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
            add_header X-Cache-Status $upstream_cache_status;  # 查看命中状态
        }
    }
}
```

查看命中状态：
```
X-Cache-Status: HIT    # 命中
X-Cache-Status: MISS    # 未命中
X-Cache-Status: BYPASS  # 跳过缓存
```

---

## 五、HTTP/2 和 SSL 优化

### HTTP/2

```nginx
server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # Session 缓存
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
}
```

### 强制 HTTPS

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

---

## 六、负载均衡策略

```nginx
upstream backend {
    # 最少连接（适合短连接）
    least_conn;
    
    server 192.168.1.10:8080 weight=3;  # 权重 3
    server 192.168.1.11:8080 weight=2;  # 权重 2
    server 192.168.1.12:8080 backup;     # 备用机
    
    # 健康检查（商业版 NGINX Plus）
    # health_check;
}
```

| 策略 | 说明 |
|------|------|
| `round_robin`（默认） | 轮询 |
| `least_conn` | 最少连接优先 |
| `ip_hash` | 同 IP 一直访问同一台 |
| `hash` | 按 key 哈希 |

---

## 七、限流防护

### 连接数限制

```nginx
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

server {
    location / {
        limit_conn conn_limit 10;  # 单 IP 最多 10 连接
    }
}
```

### 请求速率限制

```nginx
limit_req_zone $binary_remote_addr zone=req_limit:10m rate=10r/s;

server {
    location / {
        limit_req zone=req_limit burst=20 nodelay;  # 突发 20 个
    }
}
```

### 带宽限制

```nginx
location /download/ {
    limit_rate 500k;          # 单连接限速 500KB/s
    limit_rate_after 2m;       # 前 2MB 不限速
}
```

---

## 八、图片处理

减少图片大小比任何优化都有效：

```nginx
# 自动转 WebP（需要 ngx_http_image_filter_module）
location ~* \.(jpg|jpeg|png)$ {
    expires 30d;
    add_header Vary Accept;
    try_files $uri$suffix @webp $uri;
}

location @webp {
    internal;
    proxy_pass http://127.0.0.1:8080/convert?format=webp&quality=80&source=$uri;
}
```

简单方案：用 [cwebp](https://developers.google.com/speed/webp/docs/precompiled) 提前转换：

```bash
# 批量转换
find . -name "*.jpg" -exec cwebp -q 80 {} -o {}.webp \;
```

---

## 九、日志优化

### 关闭不需要的日志

```nginx
location /static/ {
    access_log off;  # 静态资源不记日志
}
```

### 日志切片

```bash
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
    endscript
}
```

### 简化日志格式

```nginx
http {
    log_format main '$remote_addr - $request_time $status $body_bytes_sent "$request"';
    access_log /var/log/nginx/access.log main;
}
```

---

## 十、压测验证

调优完必须压测验证：

```bash
# 单机压测
wrk -t12 -c400 -d30s https://yourdomain.com

# 分布式压测（多台机器）
# 安装 locust
pip install locust
locust -f locustfile.py --headless -u 10000 -r 100 -t 1h --host https://yourdomain.com
```

关注指标：
- **QPS** — 每秒请求数
- **Latency** — 响应时间（P50、P95、P99）
- **Error Rate** — 错误率

---

## 总结：调优清单

| 类别 | 优化项 | 预期效果 |
|------|--------|----------|
| 系统 | 文件描述符 | 支持更多并发 |
| 系统 | worker 数 | 充分利用 CPU |
| 传输 | Gzip | 流量减少 5-10 倍 |
| 缓存 | 静态资源缓存 | 减少回源 |
| 代理 | HTTP/2 | 减少延迟 |
| 安全 | HTTPS | 加密传输 |
| 防护 | 限流 | 防止攻击 |
| 日志 | 切片/关闭 | 减少 IO |

**先测基准 → 逐项优化 → 再测对比 → 保留效果好的**

不要一次性全改，改一个测一个。

---

*作者：errtry*
