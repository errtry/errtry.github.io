---
layout: post
title: 使用 Tmux 和 Neovim 打造终极终端开发环境
categories: [Tools]
description: 详细讲解 Tmux 终端多路复用和 Neovim 配置，打造高效终端开发环境
keywords: Tmux, Neovim, 终端, 开发环境, Vim
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇实战的终端环境配置指南，用好这两个工具效率翻倍

## 为什么是 Tmux + Neovim？

- **Tmux** — 终端多路复用，分屏、会话、后台运行
- **Neovim** — 现代 Vim，功能扩展强，Lua 配置
- **组合起来** — 一个终端窗口搞定所有事

---

## 一、Tmux 入门

### 安装

```bash
# macOS
brew install tmux

# Ubuntu/Debian
sudo apt install tmux

# CentOS
sudo yum install tmux

# Windows WSL
sudo apt install tmux
```

### 核心概念

| 概念 | 说明 |
|------|------|
| **Session** | 会话，一组窗口的集合 |
| **Window** | 窗口，相当于一个标签页 |
| **Pane** | 面板，一个窗口里的分屏 |

### 基本操作

```bash
# 启动
tmux

# 快捷键前缀 Ctrl+b（之后简写为 C-b）
# C-b ?   # 查看所有快捷键

# 窗口操作
C-b c      # 创建新窗口
C-b n      # 下一个窗口
C-b p      # 上一个窗口
C-b 数字   # 跳到第 N 个窗口
C-b w      # 列出所有窗口
C-b ,      # 重命名窗口

# 面板操作
C-b %      # 左右分屏
C-b "      # 上下分屏
C-b 方向键  # 切换面板
C-b o      # 切换到下一个面板
C-b x      # 关闭面板
C-b z      # 当前面板最大化/还原
C-b C-方向键  # 调整面板大小

# 会话操作
C-b d      # 断开当前会话（后台运行）
tmux ls    # 列出所有会话
tmux attach -t 0   # 重新连接会话 0
tmux new -s work   # 新建名为 work 的会话
```

---

## 二、Tmux 配置文件

```bash
# ~/.tmux.conf

# 鼠标支持
set -g mouse on

# 鼠标选择复制（需要 mouse on）
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-selection-and-cancel

# 重新加载配置
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# 从 1 开始编号（更习惯）
set -g base-index 1
setw -g pane-base-index 1

# 更短的延迟
set -sg escape-time 0

# 主题：状态栏
set -g status-bg black
set -g status-fg white
set -g status-left-length 40
set -g status-left "#[fg=green]#S #[fg=white]|"
set -g status-right "#[fg=cyan]%Y-%m-%d %H:%M"

# 当前窗口高亮
setw -g window-status-current-bg yellow
setw -g window-status-current-fg black

# 分隔线
set -g pane-border-style fg=colour235
set -g pane-active-border-style fg=colour240

# 消息样式
set -g message-style bg=yellow
set -g message-style fg=black
```

加载配置：

```bash
tmux source-file ~/.tmux.conf
```

---

## 三、Tmux 实战技巧

### 会话管理

```bash
# 命名会话
tmux new -s project1

# 分离并重命名
# 在 tmux 内：C-b D，然后选要分离的客户端

# 快速切换会话
# 安装 tmux-resurrect（插件）
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# 自动保存和恢复
set -g @continuum-restore 'on'
set -g @continuum-save '15'  # 每 15 分钟保存
```

### 常用脚本

```bash
#!/bin/bash
# 启动开发环境

SESSION="dev"

# 如果会话已存在，连接它
tmux has-session -t $SESSION 2>/dev/null
if [ $? -eq 0 ]; then
    echo "Session $SESSION exists, attaching..."
    tmux attach -t $SESSION
    exit
fi

# 创建新会话
tmux new-session -d -s $SESSION -n editor

# 第一个窗口：Neovim
tmux send-keys 'cd ~/projects/myapp' C-m
tmux send-keys 'nvim .' C-m

# 第二个窗口：服务器
tmux new-window -n server -t $SESSION
tmux send-keys 'cd ~/projects/myapp' C-m
tmux send-keys 'npm run dev' C-m

# 第三个窗口：数据库
tmux new-window -n db -t $SESSION
tmux send-keys 'mysql -u root -p' C-m

# 第四个窗口：日志
tmux new-window -n logs -t $SESSION
tmux send-keys 'cd ~/projects/myapp && tail -f logs/app.log' C-m

# 切换到第一个窗口
tmux select-window -t $SESSION:editor

# 连接
tmux attach -t $SESSION
```

---

## 四、Neovim 安装与配置

### 安装

```bash
# macOS
brew install neovim

# Ubuntu
sudo apt install neovim

# Windows WSL
sudo apt install neovim

# 检查版本（需要 0.8+）
nvim --version
```

### 配置文件位置

```bash
# Neovim 配置目录
~/.config/nvim/
├── init.lua          # 主配置
├── lua/
│   └── user/         # 自定义模块
├── plugin/           # 插件目录
└── init.vim          # 兼容模式（可选）
```

### init.lua 基础配置

```lua
-- ~/.config/nvim/init.lua

-- 基本设置
vim.opt.number = true          -- 行号
vim.opt.relativenumber = true  -- 相对行号
vim.opt.cursorline = true      -- 高亮当前行
vim.opt.signcolumn = "yes"     -- 侧边栏显示符号
vim.opt.wrap = false           -- 不自动换行
vim.opt.swapfile = false        -- 不生成 swap 文件
vim.opt.backup = false          -- 不备份
vim.opt.writebackup = false

-- 缩进
vim.opt.tabstop = 4            -- Tab 4 空格
vim.opt.shiftwidth = 4         -- 缩进 4 空格
vim.opt.expandtab = true       -- Tab 转空格
vim.opt.smartindent = true    -- 智能缩进

-- 搜索
vim.opt.hlsearch = true        -- 高亮搜索
vim.opt.incsearch = true       -- 增量搜索
vim.opt.ignorecase = true      -- 忽略大小写
vim.opt.smartcase = true        -- 智能大小写

-- 外观
vim.opt.termguicolors = true   -- 真彩色
vim.opt.background = "dark"    -- 暗色主题
vim.opt.showmode = false        -- 底部不显示模式

-- 性能
vim.opt.updatetime = 50         -- 更快的更新（让 cursorhold 事件更频繁）
vim.opt.timeoutlen = 300        -- 更快超时

-- 快捷键
vim.g.mapleader = " "
vim.keymap.set("n", "<leader>w", ":w<CR>")           -- 保存
vim.keymap.set("n", "<leader>q", ":q<CR>")            -- 退出
vim.keymap.set("n", "<leader>s", ":w<CR>:source %<CR>") -- 保存并重载配置

-- 快速移动（Vim 风格）
vim.keymap.set("n", "J", "mzJ`z")  -- J 合并行但保持光标位置
vim.keymap.set("n", "<C-d>", "<C-d>zz")  -- 上下滚动居中
vim.keymap.set("n", "<C-u>", "<C-u>zz")

-- 分屏
vim.keymap.set("n", "sv", ":vsplit<CR>")   -- 左右分
vim.keymap.set("n", "sh", ":split<CR>")     -- 上下分
vim.keymap.set("n", "sc", "<C-w>c")        -- 关闭当前
vim.keymap.set("n", "so", "<C-w>o")        -- 只保留当前

-- 分屏移动
vim.keymap.set("n", "<leader>h", "<C-w>h")
vim.keymap.set("n", "<leader>j", "<C-w>j")
vim.keymap.set("n", "<leader>k", "<C-w>k")
vim.keymap.set("n", "<leader>l", "<C-w>l")

-- Terminal 模式
vim.keymap.set("n", "<leader>t", ":split | terminal<CR>")
vim.keymap.set("t", "<Esc>", "<C-\\><C-n>")
```

---

## 五、插件管理：Lazy.nvim

### 安装

```bash
# 安装 lazy.nvim
git clone --filter=blob:none https://github.com/folke/lazy.nvim.git --depth 1 ~/.local/share/nvim/site/pack/lazy/start/lazy.nvim
```

### 配置插件

```lua
-- ~/.config/nvim/init.lua 末尾添加

require("lazy").setup({
    -- 主题
    { "folke/tokyonight.nvim" },

    -- 状态栏
    { "nvim-lualine/lualine.nvim" },

    -- 文件树
    { "nvim-tree/nvim-tree.lua" },

    -- 语法高亮
    { "nvim-treesitter/nvim-treesitter", build = ":TSUpdate" },

    -- LSP
    { "neovim/nvim-lspconfig" },
    { "williamboman/mason.nvim" },

    -- 自动补全
    { "hrsh7th/nvim-cmp" },
    { "hrsh7th/cmp-nvim-lsp" },
    { "L3MON4D3/LuaSnip" },

    -- 模糊搜索
    { "nvim-telescope/telescope.nvim" },

    -- 注释
    { "numToStr/Comment.nvim" },
})
```

### 主题配置

```lua
-- ~/.config/nvim/lua/user/colorscheme.lua
require("tokyonight").setup({
    style = "night",
    transparent = false,
})

vim.cmd("colorscheme tokyonight")
```

### LSP 配置

```lua
-- ~/.config/nvim/lua/user/lsp.lua
local lsp = require("lspconfig")

-- Python
lsp.pyright.setup({})

-- TypeScript
lsp.tsserver.setup({})

-- Go
lsp.gopls.setup({})

-- Rust
lsp.rust_analyzer.setup({})

-- 显示 LSP 图标
require("lspkind").init({})

-- 快捷键
vim.keymap.set("n", "gd", vim.lsp.buf.definition)
vim.keymap.set("n", "K", vim.lsp.buf.hover)
vim.keymap.set("n", "<leader>rn", vim.lsp.buf.rename)
vim.keymap.set("n", "<leader>ca", vim.lsp.buf.code_action)
```

---

## 六、Tmux + Neovim 联动

### 配置 Neovim 内嵌终端

```lua
-- ~/.config/nvim/lua/user/terminal.lua

vim.keymap.set("n", "<C-t>", ":split | resize 10 | terminal<CR>")
vim.keymap.set("t", "<Esc>", "<C-\\><C-n>")
```

### 分屏同步

让所有面板同步输入相同命令：

```bash
# 在 tmux.conf 中
bind e set synchronize-panes \; display "Sync: $(if [[ $(tmux show-options -w synchronize-panes) == *on ]]; then echo ON; else echo OFF; fi)"
```

---

## 七、懒人配置包

不想自己配？用现成的：

| 发行版 | 说明 |
|--------|------|
| [LazyVim](https://www.lazyvim.org/) | Neovim 完整发行版 |
| [AstroVim](https://astronvim.com/) | AstroNVIM，快速上手 |
| [NvChad](https://nvchad.com/) | 现代化 Vim 配置 |

直接用 LazyVim：

```bash
# 安装 LazyVim Starter
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
nvim
```

---

## 总结

| 工具 | 核心功能 | 学会后收益 |
|------|----------|------------|
| Tmux | 分屏、会话、后台运行 | 终端效率翻倍 |
| Neovim | 强大的文本编辑 | 代码写到手抽筋 |
| 组合 | 终端 + 编辑器一体化 | 一个窗口搞定所有事 |

**建议**：先单独用一周，熟悉基本操作后再组合使用。

---

*作者：errtry*
