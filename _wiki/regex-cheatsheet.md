---
layout: wiki
title: 正则表达式速查
cate1: 正则
cate2: 表达式
description: 正则表达式常用模式速查，包含字符类、量词、锚点、分组、断言
keywords: 正则, Regex, 正则表达式, 模式匹配
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 常用工具：regex101.com、regexr.com

## 🔤 字符类

| 模式 | 说明 | 示例 |
|------|------|------|
| `.` | 任意字符（换行符除外） | `a.c` → abc, aXc |
| `\d` | 数字 | `\d{3}` → 123 |
| `\D` | 非数字 | `\D+` → abc |
| `\w` | 字母数字下划线 | `\w+` → hello_1 |
| `\W` | 非字母数字 | `\W` → @#$ |
| `\s` | 空白字符（空格 tab 换行） | `a\sb` → a b |
| `\S` | 非空白 | `\S+` → abc |
| `[abc]` | 字符集 | `[aeiou]` → 元音 |
| `[^abc]` | 排除 | `[^0-9]` → 非数字 |
| `[a-z]` | 范围 | 小写字母 |

## ⏱️ 量词

| 量词 | 说明 | 示例 |
|------|------|------|
| `*` | 0 次或多次 | `ab*` → a, ab, abb |
| `+` | 1 次或多次 | `ab+` → ab, abb |
| `?` | 0 次或 1 次 | `colou?r` → color, colour |
| `{n}` | 恰好 n 次 | `\d{4}` → 2024 |
| `{n,}` | 至少 n 次 | `\d{2,}` → 12, 123 |
| `{n,m}` | n 到 m 次 | `\d{2,4}` → 12, 123, 1234 |
| `*?` | 非贪婪（最少） | `.*?` → 尽可能少匹配 |
| `+?` | 非贪婪 | `.+?` | 

## 🪝 锚点

| 锚点 | 说明 | 示例 |
|------|------|------|
| `^` | 字符串开头 | `^hello` → hello world |
| `$` | 字符串结尾 | `world$` → hello world |
| `\b` | 单词边界 | `\bword\b` → 完整单词 |
| `\B` | 非单词边界 | |

## 👥 分组

| 模式 | 说明 | 示例 |
|------|------|------|
| `(abc)` | 捕获组 | `(ab)+` → abab |
| `(?:abc)` | 非捕获组 | `(?:ab)+` |
| `(?<name>abc)` | 命名组 | `(?<year>\d{4})` |
| `\1` | 反向引用 | `(\w)\1` → aa, bb |
| `(a|b)` | 选择 | `(cat|dog)` → cat, dog |

## 🔍 断言

| 断言 | 说明 | 示例 |
|------|------|------|
| `(?=abc)` | 正向先行 | `\d(?=px)` → 5p → 5 |
| `(?!abc)` | 负向先行 | `\d(?!px)` → 5em → 5 |
| `(?<=abc)` | 正向后行 | `(?<=\$)\d+` → $100 → 100 |
| `(?<!abc)` | 负向后行 | `(?<!\$)\d+` → 100$ |

## 🛠️ 常用模式

```regex
# 中国手机号
1[3-9]\d{9}

# 邮箱
[\w.-]+@[\w.-]+\.\w+

# 身份证号（18位）
[1-9]\d{5}(19|20)\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\d|3[01])\d{3}[\dXx]

# URL
https?://[\w.-]+(/[\w./-]*)?)

# IPv4
\b(?:\d{1,3}\.){3}\d{1,3}\b

# 日期 YYYY-MM-DD
\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\d|3[01])

# 时间 HH:MM:SS
(?:[01]\d|2[0-3]):[0-5]\d:[0-5]\d

# 中文
[\u4e00-\u9fa5]+

# 密码（字母+数字+特殊字符，8位以上）
^(?=.*[a-zA-Z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}

# QQ号
[1-9]\d{4,10}

# 微信号
^[a-zA-Z][\w-]{5,19}$

# 银行卡号
\d{16,19}

# HTML 标签
<([a-z]+)[^>]*>.*?</\1>
```

## 🖥️ 常用命令

```bash
# grep 过滤
grep -E '\d+' file.txt
grep -o '[0-9]\+' file.txt

# sed 替换
sed -E 's/(\d{4})-(\d{2})-(\d{2})/\3\/\2\1/' file.txt

# find
find . -name '*.txt' -regex '.*[0-9]+\.txt'

# 批量重命名
rename 's/(\d+)_(\d+)/$2_$1/' *.txt
```

## 🐍 Python 示例

```python
import re

text = '手机: 13812345678, 邮箱: test@example.com'

# 匹配
phone = re.search(r'1[3-9]\d{9}', text)
print(phone.group())  # 13812345678

# 查找所有
emails = re.findall(r'[\w.-]+@[\w.-]+\.\w+', text)
print(emails)  # ['test@example.com']

# 替换
new_text = re.sub(r'\d{3}\d{4}\d{4}', '***-****-****', text)

# 分割
parts = re.split(r'[,，]', 'a,b,c，d')

# 命名组
m = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})', '2024-01-15')
print(m.group('year'))  # 2024

# 编译复用
pattern = re.compile(r'\d+')
pattern.findall(text)
```
