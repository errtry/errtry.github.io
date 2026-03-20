---
layout: wiki
title: PowerShell 常用命令
cate1: Windows
cate2: PowerShell
description: Windows PowerShell 常用命令速查，包含文件、进程、服务、网络
keywords: PowerShell, PS, Windows, 命令, 脚本
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 快捷键 `Win + X` → `I` 打开 PowerShell，`Win + R` 输入 `powershell`

## 🔌 基础操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `Get-Help` | 查看帮助 | `Get-Help Get-Process` |
| `Get-Command` | 查看命令 | `Get-Command -Verb Get` |
| `Get-Member` | 查看对象成员 | `obj \| Get-Member` |
| `Get-Alias` | 查看别名 | - |

## 📁 文件操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `Get-ChildItem` / `ls` / `dir` | 列出文件 | `ls -Recurse` |
| `Set-Location` / `cd` | 切换目录 | `cd C:\` |
| `New-Item` | 创建 | `New-Item -ItemType Directory folder` |
| `Remove-Item` / `rm` | 删除 | `Remove-Item -Recurse -Force` |
| `Copy-Item` / `cp` | 复制 | `Copy-Item src dest` |
| `Move-Item` / `mv` | 移动 | `Move-Item a.txt folder\` |
| `Get-Content` / `cat` | 读取 | `Get-Content a.txt` |
| `Set-Content` | 写入 | `Set-Content a.txt "hello"` |
| `Test-Path` | 检查路径 | `Test-Path a.txt` |

## ⚙️ 进程管理

| 命令 | 说明 | 示例 |
|------|------|------|
| `Get-Process` | 查看进程 | `Get-Process python` |
| `Stop-Process` | 结束进程 | `Stop-Process -Name notepad -Force` |
| `Start-Process` | 启动进程 | `Start-Process notepad` |
| `Wait-Process` | 等待进程 | - |

## 🔧 服务管理

| 命令 | 说明 | 示例 |
|------|------|------|
| `Get-Service` | 查看服务 | `Get-Service` |
| `Start-Service` | 启动服务 | `Start-Service Spooler` |
| `Stop-Service` | 停止服务 | `Stop-Service Spooler` |
| `Restart-Service` | 重启服务 | `Restart-Service Spooler` |
| `Set-Service` | 修改服务 | - |

## 🌐 网络操作

| 命令 | 说明 | 示例 |
|------|------|------|
| `Test-Connection` / `ping` | ping 测试 | `Test-Connection google.com` |
| `Resolve-DnsName` / `nslookup` | DNS 查询 | `Resolve-DnsName google.com` |
| `Get-NetIPAddress` | 查看 IP | - |
| `Get-NetAdapter` | 网卡信息 | - |
| `New-NetFirewallRule` | 创建防火墙规则 | - |
| `Test-NetConnection` | 端口测试 | `Test-NetConnection google.com -Port 443` |

## 💾 系统信息

| 命令 | 说明 |
|------|------|
| `Get-ComputerInfo` | 系统详细信息 |
| `Get-WmiObject` | WMI 信息 |
| `Get-EventLog` | 事件日志 |
| `Get-EventLog -LogName System` | 系统日志 |
| `systeminfo` | 兼容命令 |

## 📦 模块操作

```powershell
# 查看已加载模块
Get-Module

# 查看可安装模块
Find-Module -Name *

# 安装模块
Install-Module -Name

# 导入模块
Import-Module
```

## 🔧 管道和筛选

```powershell
# 筛选
Get-Process | Where-Object CPU -gt 10
Get-Process | Where-Object { $_.CPU -gt 10 -and $_.Name -eq 'chrome' }

# 排序
Get-Process | Sort-Object CPU -Descending

# 选择
Get-Process | Select-Object Name, CPU, Id

# 统计
(Get-Process).Count

# 格式化输出
Get-Process | Format-Table Name, CPU
Get-Process | Format-List *
```

## 📝 常用快捷变量

| 变量 | 说明 |
|------|------|
| `$_` | 当前管道对象 |
| `$PSVersionTable` | PS 版本 |
| `$env:VAR` | 环境变量 |
| `$HOME` | 用户目录 |
| `$PWD` | 当前目录 |

## 🛠️ 常用环境变量

```powershell
# 查看环境变量
$env:Path

# 添加到 PATH
$env:Path += ";C:\Tools\bin"

# 临时设置
$env:JAVA_HOME = "C:\Java\jdk17"
```

## 🔄 循环和条件

```powershell
# ForEach
1..10 | ForEach-Object { $_ * 2 }

# ForEach-Object
Get-Process | ForEach-Object { $_.Name }

# 条件
if ($a -eq 1) { "one" }
switch ($status) { 1 { "OK" } 2 { "Fail" } }

# For
for ($i=0; $i -lt 10; $i++) { $i }

# While
while ($true) { break }
```

## ⚡ 常用快捷命令

```powershell
# 快速打开文件位置
(Get-Item file.txt).DirectoryName

# 管理员权限
Start-Process powershell -Verb RunAs

# 定时任务
schtasks /create /sc minute /mo 1 /tn "MyTask" /tr "powershell -file script.ps1"

# 字符串操作
"hello" .ToUpper()
"hello" .Replace("l", "r")
"a,b,c" -split ","
```

## 📜 一行命令

```powershell
# 杀掉所有 python 进程
Get-Process python -ErrorAction SilentlyContinue | Stop-Process

# 批量重命名
Get-ChildItem *.txt | Rename-Item -NewName { $_.Name -replace 'old', 'new' }

# 查找大文件
Get-ChildItem -Recurse | Sort-Object Length -Descending | Select -First 10

# 端口占用
netstat -ano | Select-String ":8080"
```
