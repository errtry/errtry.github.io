---
layout: wiki
title: Vim 常用命令
cate1: Vim
cate2: 编辑器
description: Vim 常用命令速查，包含模式、快捷键、光标移动、搜索替换
keywords: Vim, Vi, 编辑器, 快捷键, Linux, 文本编辑
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 模式：Normal（默认）→ `i` 进入插入 → `Esc` 返回 → `:` 进入命令

## 🌀 模式切换

| 按键 | 说明 |
|------|------|
| `Esc` | 返回 Normal 模式 |
| `i` | 当前光标插入 |
| `I` | 行首插入 |
| `a` | 光标后插入 |
| `A` | 行尾插入 |
| `o` | 下方新建行 |
| `O` | 上方新建行 |
| `s` | 删除光标字符并插入 |
| `S` | 删除整行并插入 |
| `:` | 命令行模式 |
| `v` | 可视模式（选中文本） |
| `V` | 可视模式（选中整行） |
| `Ctrl+v` | 可视模式（块选择） |

## 🖱️ 光标移动

| 按键 | 说明 |
|------|------|
| `h` `j` `k` `l` | 左/下/上/右 |
| `w` | 下一个词开头 |
| `b` | 上一个词开头 |
| `e` | 下一个词结尾 |
| `0` | 行首 |
| `^` | 行首（非空白） |
| `$` | 行尾 |
| `gg` | 文件开头 |
| `G` | 文件末尾 |
| `:n` | 跳转到第 n 行 |
| `nG` | 跳转到第 n 行 |
| `%` | 配对括号跳转 |
| `Ctrl+o` | 退回上一个位置 |
| `Ctrl+i` | 前进下一个位置 |

## ✂️ 编辑操作

| 按键 | 说明 |
|------|------|
| `x` | 删除光标字符 |
| `X` | 删除光标前字符 |
| `dd` | 删除整行 |
| `dw` | 删除到词尾 |
| `db` | 删除到词头 |
| `d$` | 删除到行尾 |
| `d0` | 删除到行首 |
| `D` | 删除到行尾 |
| `cc` | 替换整行 |
| `c$` | 替换到行尾 |
| `r` | 替换单个字符 |
| `R` | 替换模式 |
| `u` | 撤销 |
| `Ctrl+r` | 重做 |
| `.` | 重复上次操作 |
| `yy` | 复制整行 |
| `yw` | 复制到词尾 |
| `y$` | 复制到行尾 |
| `p` | 粘贴到光标后 |
| `P` | 粘贴到光标前 |
| `~` | 大小写切换 |

## 🔍 搜索替换

| 命令 | 说明 |
|------|------|
| `/word` | 向下搜索 |
| `?word` | 向上搜索 |
| `n` | 下一个匹配 |
| `N` | 上一个匹配 |
| `*` | 搜索当前词（向下） |
| `#` | 搜索当前词（向上） |
| `:s/old/new` | 替换当前行第一个 |
| `:s/old/new/g` | 替换当前行所有 |
| `:%s/old/new/g` | 替换文件所有 |
| `:n,m s/old/new/g` | 替换 n 到 m 行 |
| `:s/old/new/gc` | 逐个确认替换 |

## 📂 文件操作

| 命令 | 说明 |
|------|------|
| `:w` | 保存 |
| `:w file` | 另存为 |
| `:q` | 退出 |
| `:q!` | 不保存强制退出 |
| `:wq` / `:x` | 保存并退出 |
| `:wqa` | 保存所有并退出 |
| `:e file` | 打开文件 |
| `:e!` | 重新加载 |
| `:r file` | 插入文件内容 |
| `:r!cmd` | 插入命令输出 |

## 🪟 窗口操作

| 命令 | 说明 |
|------|------|
| `:sp file` | 水平分屏 |
| `:vsp file` | 垂直分屏 |
| `Ctrl+w h/j/k/l` | 切换窗口 |
| `Ctrl+w w` | 循环切换 |
| `Ctrl+w =` | 等分窗口 |
| `Ctrl+w _` | 最大化高度 |
| `Ctrl+w |` | 最大化宽度 |
| `:only` | 仅保留当前窗口 |
| `:close` | 关闭窗口 |

## 📑 折叠操作

| 命令 | 说明 |
|------|------|
| `zf` | 创建折叠 |
| `zd` | 删除折叠 |
| `zE` | 删除所有折叠 |
| `zo` | 打开折叠 |
| `zc` | 关闭折叠 |
| `za` | 切换折叠 |
| `zr` | 打开所有 |
| `zm` | 关闭所有 |
| `zR` | 打开所有（递归） |
| `zM` | 关闭所有（递归） |

## ⚙️ 高级技巧

```vim
" 批量注释（Ctrl+v 选中行后）
:I--

" 数字增减
Ctrl+a  数字加
Ctrl+x  数字减

" 宏录制
q a    # 开始录制到寄存器 a
...    # 操作
q      # 停止
@ a    # 执行宏
@@     # 重复执行

" 交换两个字符
xp

" 交换两行
ddp

" 复制当前行到下一行
yyp

" 快速编辑
:%!xxd      # 十六进制查看
:%!xxd -r   # 恢复

" 格式化代码
=           # 格式化选中
gg=G        # 格式化全文
```

## 🔧 配置建议

```vim
" ~/.vimrc 常用配置
set nu              " 显示行号
set relativenumber  " 相对行号
set cursorline      " 高亮当前行
set hlsearch        " 高亮搜索
set incsearch       " 增量搜索
set ignorecase      " 忽略大小写
set smartcase       " 智能大小写
set tabstop=4       " Tab 宽度
set expandtab       " Tab 转空格
set autoindent      " 自动缩进
set smartindent     " 智能缩进
set nowrap          " 不换行
set scrolloff=8     " 滚动保留行数
set laststatus=2    " 始终显示状态栏

" 配色
colorscheme gruvbox

" 插件管理（使用 vim-plug）
call plug#begin('~/.vim/plugged')
Plug 'preservim/nerdtree'
Plug 'yggdroot/indentline'
call plug#end()
```

## 🧠 记忆技巧

```
移动：h j k l （方向键）
词：w b e （word, back, end）
行：0 ^ $ （首， 首字符，尾）
删：d + 移动（d2w, d$, dd）
改：c + 移动（同 d 但进入插入）
复制：y + 移动（yy, yw）
```

> 💡 入门练习：`vimtutor` 命令，30 分钟教程
