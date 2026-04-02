---
layout: post
title: RustDesk 自建服务器完整部署指南
categories: [Tool]
description: 从零开始搭建 RustDesk 自建服务器，支持 Docker、Linux、Windows 三种部署方式，包含端口配置、客户端设置、常见问题解决
keywords: RustDesk, 远程桌面, 自建服务器, Docker, 内网穿透
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇保姆级教程，跟着做就能搭建自己的远程桌面服务器

## 为什么自建 RustDesk？

RustDesk 是一款开源的远程桌面软件，类似 TeamViewer、向日葵。自建服务器的优势：

| 对比项 | 官方服务器 | 自建服务器 |
|--------|-----------|-----------|
| 隐私 | 数据经过第三方 | 数据完全自控 |
| 速度 | 依赖官方中继 | 局域网直连更快 |
| 费用 | 免费版有限制 | 完全免费 |
| 稳定性 | 依赖官方服务 | 自己掌控 |
| 合规 | 可能不满足企业要求 | 满足数据不出境 |

---

## 一、准备工作

### 服务器要求

RustDesk 服务器要求极低：

| 配置项 | 最低要求 | 推荐配置 |
|--------|---------|---------|
| CPU | 1核 | 2核+ |
| 内存 | 512MB | 1GB+ |
| 硬盘 | 1GB | 10GB+ |
| 带宽 | 1Mbps | 10Mbps+ |

> 💡 树莓派、NAS、低配云服务器都能跑

### 端口要求

**核心端口（必须开放）：**

| 端口 | 协议 | 用途 | 服务 |
|------|------|------|------|
| 21115 | TCP | NAT 类型测试 | hbbs |
| 21116 | TCP/UDP | ID 注册、心跳、TCP 打洞 | hbbs |
| 21117 | TCP | 中继服务 | hbbr |

**可选端口：**

| 端口 | 协议 | 用途 |
|------|------|------|
| 21118 | TCP | Web 客户端支持 |
| 21119 | TCP | Web 客户端支持 |
| 21114 | TCP | Pro 版 Web 控制台 |

> ⚠️ **21116 必须同时开放 TCP 和 UDP**

---

## 二、部署方式选择

| 方式 | 适用场景 | 推荐指数 |
|------|----------|----------|
| **Docker Compose** | Linux 服务器（推荐） | ⭐⭐⭐⭐⭐ |
| **一键脚本** | 快速部署 | ⭐⭐⭐⭐ |
| **Debian 包** | Debian/Ubuntu | ⭐⭐⭐ |
| **Windows + NSSM** | Windows Server | ⭐⭐⭐ |
| **Windows + PM2** | Windows 开发机 | ⭐⭐ |

---

## 三、Docker Compose 部署（推荐）

### 3.1 安装 Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | bash

# 启动 Docker
systemctl start docker
systemctl enable docker

# 验证安装
docker --version
```

### 3.2 创建目录

```bash
mkdir -p /opt/rustdesk-server
cd /opt/rustdesk-server
mkdir -p data
```

### 3.3 创建 docker-compose.yml

```yaml
version: '3'

services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server:latest
    command: hbbs
    volumes:
      - ./data:/root
    network_mode: "host"
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    network_mode: "host"
    restart: unless-stopped
```

> ⚠️ `network_mode: "host"` 仅 Linux 可用，Windows 请使用端口映射

### 3.4 启动服务

```bash
# 拉取镜像
docker compose pull

# 启动服务
docker compose up -d

# 查看状态
docker compose ps

# 查看日志
docker compose logs -f
```

### 3.5 获取 Key

```bash
# 查看公钥
cat ./data/id_ed25519.pub
```

输出类似：
```
O7mKxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
```

**记下这个 Key，客户端配置要用！**

---

## 四、一键脚本部署（最快）

适合快速体验，自动配置一切。

### 4.1 运行安装脚本

```bash
# 下载脚本
wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/install.sh

# 添加执行权限
chmod +x install.sh

# 运行安装
./install.sh
```

### 4.2 安装过程

脚本会自动：
1. 下载 hbbs 和 hbbr
2. 创建 systemd 服务
3. 配置防火墙
4. 生成客户端配置页面

安装完成后会显示：

```
========================================
RustDesk Server Installation Complete!
========================================
Your IP/DNS: 192.168.1.100
Your Key: O7mKxxxxxxxxxxxxxxxxxxxxxxxx=
========================================
```

### 4.3 管理服务

```bash
# 查看状态
systemctl status rustdeskservice

# 重启服务
systemctl restart rustdeskservice

# 停止服务
systemctl stop rustdeskservice

# 查看日志
journalctl -u rustdeskservice -f
```

---

## 五、Debian 包安装

适合 Debian/Ubuntu 系统，用包管理器管理。

### 5.1 下载安装包

```bash
# 进入临时目录
cd /tmp

# 下载最新版本（访问 GitHub Releases 获取最新链接）
wget https://github.com/rustdesk/rustdesk-server/releases/download/1.1.14/rustdesk-server-hbbs_1.1.14_amd64.deb
wget https://github.com/rustdesk/rustdesk-server/releases/download/1.1.14/rustdesk-server-hbbr_1.1.14_amd64.deb

# 安装
sudo dpkg -i rustdesk-server-hbbs_1.1.14_amd64.deb
sudo dpkg -i rustdesk-server-hbbr_1.1.14_amd64.deb

# 如果有依赖问题，修复
sudo apt-get install -f
```

### 5.2 启动服务

```bash
# 启动 hbbs
sudo systemctl start hbbs
sudo systemctl enable hbbs

# 启动 hbbr
sudo systemctl start hbbr
sudo systemctl enable hbbr

# 查看状态
sudo systemctl status hbbs hbbr
```

### 5.3 获取 Key

```bash
# 公钥位置
cat /var/lib/rustdesk-server/id_ed25519.pub
```

---

## 六、Windows 部署

### 6.1 方式一：NSSM（推荐）

适合 Windows Server，以系统服务运行。

#### 步骤 1：下载文件

1. 下载 RustDesk Server：https://github.com/rustdesk/rustdesk-server/releases
   - 选择 `rustdesk-server-windows-x86_64.zip`

2. 下载 NSSM：https://github.com/dkxce/NSSM/releases
   - 解压后把 `nssm.exe` 放到 `C:\Program Files\NSSM\`

#### 步骤 2：解压 RustDesk Server

```
解压到：C:\Program Files\RustDesk Server\
目录结构：
C:\Program Files\RustDesk Server\
├── hbbs.exe
├── hbbr.exe
└── ...
```

#### 步骤 3：安装服务

打开 **管理员权限** 的 CMD 或 PowerShell：

```powershell
# 安装 hbbs 服务
nssm install "RustDesk hbbs" "C:\Program Files\RustDesk Server\hbbs.exe"

# 安装 hbbr 服务
nssm install "RustDesk hbbr" "C:\Program Files\RustDesk Server\hbbr.exe"

# 启动服务
nssm start "RustDesk hbbs"
nssm start "RustDesk hbbr"
```

#### 步骤 4：配置防火墙

```powershell
# 开放端口
New-NetFirewallRule -DisplayName "RustDesk hbbs" -Direction Inbound -Protocol TCP -LocalPort 21115-21116,21118 -Action Allow
New-NetFirewallRule -DisplayName "RustDesk hbbs UDP" -Direction Inbound -Protocol UDP -LocalPort 21116 -Action Allow
New-NetFirewallRule -DisplayName "RustDesk hbbr" -Direction Inbound -Protocol TCP -LocalPort 21117,21119 -Action Allow
```

#### 步骤 5：获取 Key

```powershell
# 查看公钥
Get-Content "C:\Program Files\RustDesk Server\id_ed25519.pub"
```

### 6.2 方式二：PM2

适合 Windows 开发机，更简单。

#### 步骤 1：安装 Node.js

下载安装：https://nodejs.org/

#### 步骤 2：安装 PM2

```powershell
npm install -g pm2
npm install -g pm2-windows-startup
pm2-startup install
```

#### 步骤 3：运行服务

```powershell
# 进入 RustDesk Server 目录
cd "C:\rustdesk-server-windows-x64"

# 启动服务
pm2 start hbbs.exe --name hbbs
pm2 start hbbr.exe --name hbbr

# 保存配置
pm2 save
```

#### 步骤 4：查看日志

```powershell
pm2 logs hbbs
pm2 logs hbbr
```

---

## 七、防火墙配置

### Ubuntu UFW

```bash
# 开放端口
sudo ufw allow 21115:21119/tcp
sudo ufw allow 21116/udp

# 启用防火墙
sudo ufw enable

# 查看状态
sudo ufw status
```

### CentOS firewalld

```bash
# 开放端口
sudo firewall-cmd --permanent --add-port=21115-21119/tcp
sudo firewall-cmd --permanent --add-port=21116/udp

# 重载配置
sudo firewall-cmd --reload

# 查看已开放端口
sudo firewall-cmd --list-ports
```

### 云服务器安全组

如果你用的是云服务器（阿里云、腾讯云、AWS 等），还需要在**安全组**中放行端口：

| 方向 | 端口 | 协议 |
|------|------|------|
| 入站 | 21115-21119 | TCP |
| 入站 | 21116 | UDP |

---

## 八、客户端配置

### 8.1 下载客户端

官网下载：https://rustdesk.com/

支持：
- Windows
- macOS
- Linux
- Android
- iOS

### 8.2 配置服务器

打开 RustDesk 客户端：

1. 点击 **设置** ⚙️
2. 选择 **网络**
3. 填写 **ID 服务器**：`你的服务器IP`
4. 填写 **Key**：`刚才记下的公钥`
5. 其他选项留空

![客户端配置示意](/assets/images/rustdesk-client-config.png)

### 8.3 验证连接

配置完成后：

1. 在客户端首页，点击 ID 旁边的刷新按钮
2. 如果显示 **就绪**，说明配置成功
3. 如果显示 **未就绪**，检查网络和 Key

---

## 九、Nginx 反向代理（可选）

如果需要通过域名访问，可以配置 Nginx 反向代理。

### 9.1 配置示例

```nginx
# /etc/nginx/sites-available/rustdesk.conf

server {
    listen 80;
    server_name rustdesk.example.com;

    # 证书配置（推荐）
    # listen 443 ssl;
    # ssl_certificate /path/to/cert.pem;
    # ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:21114;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket 支持
    location /ws {
        proxy_pass http://127.0.0.1:21118;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 9.2 启用配置

```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/rustdesk.conf /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重载 Nginx
sudo nginx -s reload
```

---

## 十、常见问题

### Q1：客户端显示「未就绪」

**可能原因：**
1. 防火墙未开放端口
2. Key 配置错误
3. 服务未启动

**解决方法：**
```bash
# 检查服务状态
docker compose ps
# 或
systemctl status hbbs hbbr

# 检查端口
netstat -tlnp | grep 2111

# 检查 Key
cat ./data/id_ed25519.pub
```

### Q2：内网无法连接

**原因：** 内网穿透问题

**解决方法：**
1. 确保 21116/UDP 开放
2. 检查 NAT 类型
3. 如果两端都在同一局域网，会自动直连

### Q3：Windows 服务无法启动

**解决方法：**
```powershell
# 检查服务状态
sc query "RustDesk hbbs"

# 手动启动
sc start "RustDesk hbbs"

# 查看事件日志
Get-EventLog -LogName Application -Source "RustDesk*" -Newest 10
```

### Q4：如何更新服务

**Docker 方式：**
```bash
cd /opt/rustdesk-server
docker compose pull
docker compose up -d
```

**脚本方式：**
```bash
wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/update.sh
chmod +x update.sh
./update.sh
```

### Q5：如何查看日志

**Docker：**
```bash
docker compose logs -f
# 或单独查看
docker logs hbbs -f
docker logs hbbr -f
```

**Systemd：**
```bash
journalctl -u hbbs -f
journalctl -u hbbr -f
```

### Q6：连接速度慢

**优化方案：**
1. 确保两端都在服务器同一区域
2. 检查带宽是否足够
3. 强制使用中继（如果直连失败）：
   ```yaml
   # docker-compose.yml
   environment:
     - ALWAYS_USE_RELAY=Y
   ```

---

## 十一、安全建议

1. **定期更新** — 保持服务器和客户端最新版本
2. **使用 Key** — 不要禁用 Key 验证
3. **限制访问** — 通过防火墙限制访问 IP
4. **启用日志** — 定期检查异常连接
5. **备份 Key** — 丢失 Key 需要重新配置所有客户端

---

## 十二、总结

| 步骤 | 操作 |
|------|------|
| 1 | 准备服务器，开放端口 |
| 2 | 选择部署方式（推荐 Docker） |
| 3 | 启动 hbbs 和 hbbr 服务 |
| 4 | 获取公钥 Key |
| 5 | 客户端配置服务器 IP 和 Key |
| 6 | 验证连接 |

**核心要点：**
- 21116 端口必须同时开放 TCP 和 UDP
- Key 是安全验证的关键，妥善保管
- Docker 部署最简单，推荐使用
- 内网环境也能用，会自动直连

---

## 参考资料

- [RustDesk 官方文档](https://rustdesk.com/docs/)
- [RustDesk Server OSS 文档](https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/)
- [Docker 部署指南](https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/docker/)
- [GitHub Releases](https://github.com/rustdesk/rustdesk-server/releases)

---

*作者：牛马便利店一号店员*
