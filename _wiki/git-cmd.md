---
layout: wiki
title: Git 常用命令
cate1: Git
cate2: 版本控制
description: Git 常用命令速查，包含配置、提交、分支、远程操作
keywords: Git, 版本控制, 命令, VCS, github, gitee
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 初始化：`git init`，克隆：`git clone url`

## ⚙️ 基础配置

```bash
# 设置用户名和邮箱
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 查看配置
git config --list
git config --global --list

# 设置默认分支名
git config --global init.defaultBranch main

# 设置合并工具
git config --global merge.tool vimdiff

# 加速克隆（国内）
git config --global url."https://ghproxy.com/".insteadOf https://github.com/
```

## 📥 仓库操作

```bash
# 初始化仓库
git init

# 克隆仓库
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git myfolder   # 指定目录名

# 查看状态
git status
git status -s    # 简洁模式
```

## 📝 提交操作

```bash
# 查看变更
git diff                    # 工作区 vs 暂存区
git diff --staged           # 暂存区 vs 版本库
git diff HEAD               # 工作区 vs 版本库
git diff commit1 commit2    # 比较两个版本

# 添加文件
git add filename           # 单个文件
git add .                  # 所有文件
git add -p                 # 交互式添加

# 提交
git commit -m "message"
git commit -am "message"   # 自动 add 已跟踪文件（不包含新文件）
git commit --amend         # 修改最后一次提交
git commit --amend -m "new message"  # 修改提交信息
```

## 🔙 回退操作

```bash
# 撤销工作区修改
git checkout -- filename
git restore filename      # Git 2.23+

# 撤销暂存区
git reset HEAD filename
git restore --staged filename

# 回退版本（慎用！）
git reset --hard HEAD^     # 回退到上一个版本
git reset --hard HEAD~3    # 回退到前 3 个版本
git reset --hard commit_id # 回退到指定版本

# 撤销某次提交（保留修改）
git revert commit_id       # 会创建新提交

# 恢复删除的文件
git checkout HEAD~1 -- filename
```

## 🌿 分支操作

```bash
# 查看分支
git branch                 # 本地分支
git branch -r              # 远程分支
git branch -a              # 所有分支

# 创建分支
git branch feature/new     # 创建但不会切换

# 切换分支
git checkout feature/new
git switch feature/new     # Git 2.23+

# 创建并切换
git checkout -b feature/new
git switch -c feature/new

# 合并分支
git merge feature/new      # 合并到当前分支
git merge feature/new --no-ff  # 禁止快速合并

# 删除分支
git branch -d feature/new  # 安全删除
git branch -D feature/new  # 强制删除

# 删除远程分支
git push origin --delete feature/new
git push origin :feature/new
```

## 🔀 储藏操作

```bash
# 储藏修改
git stash
git stash push -m "message"

# 查看储藏
git stash list

# 恢复储藏
git stash apply             # 恢复但不删除
git stash pop               # 恢复并删除

# 删除储藏
git stash drop              # 删除最近一个
git stash clear             # 清空所有
```

## ☁️ 远程操作

```bash
# 查看远程
git remote -v

# 添加远程
git remote add origin https://github.com/user/repo.git

# 拉取代码
git pull origin main
git pull --rebase origin main

# 推送代码
git push origin main
git push -u origin main     # -u 设置默认上游

# 强制推送（慎用！）
git push -f origin main
git push --force origin main

# 拉取远程分支
git checkout -b feature/new origin/feature/new
git switch origin/feature/new
```

## 📜 历史操作

```bash
# 查看日志
git log
git log --oneline           # 简洁模式
git log -n 3                # 最近 3 条
git log --graph             # 图形化
git log --author="name"     # 按作者筛选
git log --grep="keyword"    # 按提交信息筛选

# 查看具体文件历史
git log filename
git log -p filename         # 详细变更

# 查看某次提交
git show commit_id

# 查看文件某行谁修改的
git blame filename

# 清理未跟踪文件
git clean -fd               # 目录和文件
git clean -fdx              # 包括忽略的文件
```

## 🏷️ 标签操作

```bash
# 创建标签
git tag v1.0.0
git tag -a v1.0.0 -m "release"

# 查看标签
git tag
git tag -l "v1.*"

# 推送标签
git push origin v1.0.0
git push origin --tags       # 推送所有标签

# 删除标签
git tag -d v1.0.0           # 本地
git push origin --delete v1.0.0  # 远程
```

## ⚡ 实用技巧

```bash
# 忽略文件不生效时
git rm -r --cached .
git add .
git commit -m "update .gitignore"

# 解决冲突后
git add filename
git commit -m "merge"

# 暂存密码（临时）
git config credential.helper store

# 使用 SSH 避免每次输入密码
# 1. 生成密钥：ssh-keygen -t ed25519 -C "your@email.com"
# 2. 添加公钥到 GitHub

# 合并多个 commit
git rebase -i HEAD~3
# 将 pick 改为 squash 或 s
```

## 🔗 常用别名

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.df diff
git config --global alias.lg log --graph --oneline --decorate
git config --global alias.last log -1 HEAD
```
