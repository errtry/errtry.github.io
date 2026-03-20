---
layout: wiki
title: Linux 常用命令
cate1: Linux
cate2: 系统命令
description: Linux 常用命令速查，包含文件、网络、进程、文本处理
keywords: Linux, Shell, 命令, terminal, bash
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 使用 `Tab` 自动补全，`Ctrl+C` 终止执行，`Ctrl+R` 搜索历史

## 📁 文件操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `ls` | 列出文件 | `ls -lah` |
| `cd` | 切换目录 | `cd /var/log` |
| `pwd` | 当前目录 | - |
| `mkdir` | 创建目录 | `mkdir -p a/b/c` |
| `rmdir` | 删除空目录 | `rmdir folder` |
| `rm` | 删除文件 | `rm -rf folder` |
| `cp` | 复制 | `cp -r src dest` |
| `mv` | 移动/重命名 | `mv a.txt b.txt` |
| `touch` | 创建空文件 | `touch a.txt` |
| `cat` | 查看文件 | `cat a.txt` |
| `less` / `more` | 分页查看 | `less a.txt` |
| `head` / `tail` | 查看头/尾 | `tail -f log` |
| `ln` | 链接 | `ln -s src link` |
| `chmod` | 权限 | `chmod 755 file` |
| `chown` | 所有者 | `chown user:group file` |
| `find` | 查找 | `find . -name "*.txt"` |
| `locate` | 快速查找 | `locate file` |
| `which` | 命令位置 | `which python` |

## 🔍 文本处理

| 命令 | 说明 | 示例 |
|------|------|------|
| `grep` | 文本搜索 | `grep -r "word" .` |
| `sed` | 替换 | `sed 's/old/new/g' file` |
| `awk` | 文本处理 | `awk '{print $1}' file` |
| `cut` | 截取字段 | `cut -d: -f1 file` |
| `sort` | 排序 | `sort file` |
| `uniq` | 去重 | `uniq file` |
| `wc` | 统计 | `wc -l file` |
| `diff` | 比较 | `diff a.txt b.txt` |

## 📦 包管理 (Debian/Ubuntu)

| 命令 | 说明 |
|------|------|
| `apt update` | 更新源 |
| `apt upgrade` | 升级软件 |
| `apt install pkg` | 安装软件 |
| `apt remove pkg` | 卸载 |
| `apt search kw` | 搜索 |
| `dpkg -l` | 已安装列表 |
| `dpkg -i pkg.deb` | 本地安装 |

## 📦 包管理 (CentOS/RHEL)

| 命令 | 说明 |
|------|------|
| `yum update` | 升级 |
| `yum install pkg` | 安装 |
| `yum remove pkg` | 卸载 |
| `yum search kw` | 搜索 |
| `yum list installed` | 已安装 |

## 🌐 网络操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `ip a` / `ifconfig` | 查看 IP | - |
| `ip route` / `route` | 路由表 | - |
| `ping` | 连通测试 | `ping -c 4 google.com` |
| `traceroute` | 路由追踪 | - |
| `netstat` / `ss` | 端口监听 | `ss -tulnp` |
| `curl` | HTTP 请求 | `curl -X POST url` |
| `wget` | 下载 | `wget url` |
| `ssh` | 远程连接 | `ssh user@host` |
| `scp` | 远程复制 | `scp file user@host:/path` |
| `rsync` | 同步 | `rsync -av src dest` |
| `nslookup` / `dig` | DNS 查询 | - |

## ⚙️ 进程管理

| 命令 | 说明 | 示例 |
|------|------|------|
| `ps` | 进程列表 | `ps aux` |
| `top` / `htop` | 实时监控 | - |
| `pkill` | 按名杀掉 | `pkill python` |
| `kill` | 按 PID | `kill -9 1234` |
| `jobs` | 后台任务 | - |
| `bg` / `fg` | 后台/前台 | - |
| `nohup` | 后台运行 | `nohup cmd &` |

## 💾 磁盘操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `df -h` | 磁盘使用 | - |
| `du -sh` | 目录大小 | `du -sh *` |
| `mount` | 挂载 | `mount /dev/sdb1 /mnt` |
| `fdisk -l` | 分区表 | - |
| `mkfs` | 格式化 | `mkfs.ext4 /dev/sdb1` |

## 🔧 系统信息

| 命令 | 说明 |
|------|------|
| `uname -a` | 内核信息 |
| `hostname` | 主机名 |
| `whoami` | 当前用户 |
| `uptime` | 运行时间 |
| `free -h` | 内存 |
| `lscpu` | CPU 信息 |
| `lsblk` | 块设备 |
| `systemctl` | 服务管理 |
| `journalctl` | 日志 |

## �archives/压缩

| 命令 | 说明 | 示例 |
|------|------|------|
| `tar czf` | 打包 gzip | `tar czf a.tar.gz dir/` |
| `tar xf` | 解压 | `tar xf a.tar.gz` |
| `zip` / `unzip` | zip 压缩 | `zip -r a.zip dir/` |
| `7z` | 7z 压缩 | `7z x a.7z` |

## 🔄 重定向 & 管道

```bash
# 输出重定向
cmd > file      # 覆盖
cmd >> file     # 追加
cmd 2> err.log  # 错误重定向

# 管道
cmd1 | cmd2     # cmd1 输出作为 cmd2 输入
cmd &           # 后台运行
cmd && cmd2     # cmd 成功后再执行 cmd2
cmd || cmd2     # cmd 失败后执行 cmd2
```

## ⚡ 常用快捷键

| 快捷键 | 作用 |
|--------|------|
| `Ctrl + C` | 终止 |
| `Ctrl + Z` | 暂停 |
| `Ctrl + D` | 退出 |
| `Ctrl + L` | 清屏 |
| `Ctrl + A` | 行首 |
| `Ctrl + E` | 行尾 |
| `Ctrl + R` | 搜索历史 |
| `!!` | 上一条命令 |
| `!$` | 上一条命令的参数 |
