---
layout: wiki
title: CMD 常用命令
cate1: Windows
cate2: 系统命令
description: Windows CMD 常用命令速查，包含文件、网络、系统操作
keywords: Windows, CMD, 命令, dos, 批处理
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 快捷键 `Win + R` 输入 `cmd` 快速打开命令提示符

## 📁 文件操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `dir` | 列出当前目录文件 | `dir /w` 紧凑格式 |
| `cd` | 切换目录 | `cd D:\Tools` |
| `md` 或 `mkdir` | 创建目录 | `md folder` |
| `rd` 或 `rmdir` | 删除目录 | `rd /s /q folder` |
| `copy` | 复制文件 | `copy a.txt b.txt` |
| `move` | 移动/重命名 | `move a.txt folder\` |
| `del` | 删除文件 | `del /f /q a.txt` |
| `type` | 查看文件内容 | `type a.txt` |
| `attrib` | 修改文件属性 | `attrib +h a.txt` 隐藏 |
| `xcopy` | 递归复制 | `xcopy /s /e folder\` |

## 🌐 网络操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `ipconfig` | 查看 IP 配置 | `ipconfig /all` |
| `ping` | 测试网络连通 | `ping google.com` |
| `tracert` | 路由追踪 | `tracert google.com` |
| `netstat` | 查看端口 | `netstat -ano` |
| `nslookup` | DNS 查询 | `nslookup google.com` |
| `arp` | ARP 表 | `arp -a` |
| `route` | 路由表 | `route print` |
| `netsh` | 网络配置 | `netsh wlan show all` |
| `telnet` | 远程连接 | `telnet 192.168.1.1 23` |

## 💻 系统操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `systeminfo` | 系统详细信息 | - |
| `tasklist` | 进程列表 | `tasklist /v` |
| `taskkill` | 结束进程 | `taskkill /PID 1234 /F` |
| `hostname` | 计算机名 | - |
| `whoami` | 当前用户 | `whoami /all` |
| `shutdown` | 关机/重启 | `shutdown -s -t 60` |
| `reg` | 注册表 | `reg query HKLM\` |
| `msinfo32` | 系统信息 | - |
| `devmgmt.msc` | 设备管理器 | - |
| `services.msc` | 服务管理 | - |
| `diskmgmt.msc` | 磁盘管理 | - |
| `compmgmt.msc` | 计算机管理 | - |

## 🔧 快捷启动

| 命令 | 打开 |
|------|------|
| `calc` | 计算器 |
| `notepad` | 记事本 |
| `mspaint` | 画图 |
| `explorer` | 文件资源管理器 |
| `taskmgr` | 任务管理器 |
| `control` | 控制面板 |
| `ncpa.cpl` | 网络连接 |
| `devmgmt.msc` | 设备管理器 |
| `dcomcnfg` | 组件服务 |
| `gpedit.msc` | 本地组策略 |
| `regedit` | 注册表 |
| `msconfig` | 系统配置 |
| `cleanmgr` | 磁盘清理 |

## 📝 批处理常用

```batch
@echo off    # 关闭命令回显
setlocal      # 本地变量
echo Hello   # 输出
pause        # 暂停等待按键
goto label   # 跳转
if exist file (echo exists)  # 条件判断
for %%i in (*.*) do echo %%i  # 循环
```

## 🔗 常用组合

```batch
# 查看 IP 并保存
ipconfig /all > ipinfo.txt

# 批量重命名
ren *.txt *.bak

# 杀掉指定端口进程
netstat -ano | findstr :8080
taskkill /PID <PID> /F

# 远程重启
shutdown /r /m \\192.168.1.1 -c "reboot" -f
```
