---
layout: wiki
title: Python 常用代码
cate1: Python
cate2: 语言基础
description: Python 常用代码片段速查，包含文件操作、正则、日期、JSON、异步等
keywords: Python, 代码片段, 脚本, 编程
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 Python 3.10+ 支持 match-case

## 📁 文件操作

```python
# 读取文件
with open('file.txt', 'r', encoding='utf-8') as f:
    content = f.read()

# 按行读取
with open('file.txt', 'r') as f:
    lines = f.readlines()
    # 或
    for line in f:
        print(line)

# 写入文件
with open('file.txt', 'w', encoding='utf-8') as f:
    f.write('hello')

# 追加写入
with open('file.txt', 'a') as f:
    f.write('new line\n')

# JSON 文件
import json

with open('data.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# CSV 文件
import csv

with open('data.csv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

with open('data.csv', 'w', encoding='utf-8', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['name', 'age'])
    writer.writerows([['tom', 20], ['jerry', 25]])
```

## 🔢 字符串操作

```python
# 格式化
name = 'tom'
age = 20
msg = f'我叫{name}, 今年{age}岁'  # f-string
msg = '我叫{}, 今年{}岁'.format(name, age)
msg = '我叫%s, 今年%s岁' % (name, age)

# 切片
s = 'hello world'
s[0:5]      # 'hello'
s[::2]      # 'hlo wrd'  隔一个取
s[::-1]     # 'dlrow olleh'  翻转

# 常用方法
'hello'.upper()          # 'HELLO'
'HELLO'.lower()          # 'hello'
'  hello  '.strip()      # 'hello'
'hello world'.split()    # ['hello', 'world']
','.join(['a', 'b'])     # 'a,b'
'hello'.replace('l', 'x')  # 'hexxo'

# 判断
'hello'.startswith('he')  # True
'hello'.endswith('lo')    # True
'hello'.isalpha()         # True
'123'.isdigit()          # True

# 填充
'5'.zfill(3)              # '005'
```

## 📅 日期时间

```python
from datetime import datetime, timedelta

# 当前时间
now = datetime.now()
today = datetime.today()

# 格式化
now.strftime('%Y-%m-%d %H:%M:%S')   # '2024-01-15 10:30:00'
datetime.strptime('2024-01-15', '%Y-%m-%d')

# 时间加减
tomorrow = today + timedelta(days=1)
yesterday = today - timedelta(days=1)

# 时区
from datetime import timezone
utc_now = datetime.now(timezone.utc)
```

## 🔍 正则表达式

```python
import re

text = '我的电话是 138-1234-5678，邮箱是 test@example.com'

# 匹配
pattern = r'\d{3}-\d{4}-\d{4}'
match = re.search(pattern, text)
if match:
    print(match.group())  # '138-1234-5678'

# 查找所有
re.findall(r'\w+@\w+\.\w+', text)  # ['test@example.com']

# 替换
re.sub(r'\d{3}', 'XXX', text)  # '我的电话是 XXX-XXXX-XXXX'

# 分割
re.split(r'[,，]', 'a,b,c，d')  # ['a', 'b', 'c', 'd']

# 常用模式
r'\d+'       # 数字
r'\w+'       # 字母数字下划线
r'\s+'       # 空白字符
r'^hello'    # 开头
r'world$'    # 结尾
```

## 🌐 HTTP 请求

```python
import requests

# GET
resp = requests.get('https://api.example.com/data')
data = resp.json()

# POST
resp = requests.post('https://api.example.com/login', json={
    'username': 'tom',
    'password': '123456'
})

# 带参数
resp = requests.get('https://api.example.com/search', params={
    'q': 'python',
    'page': 1
})

# 设置 Headers
headers = {'Authorization': 'Bearer token'}
resp = requests.get(url, headers=headers)

# 下载文件
resp = requests.get(url, stream=True)
with open('file.zip', 'wb') as f:
    for chunk in resp.iter_content(chunk_size=8192):
        f.write(chunk)
```

## 📊 列表/字典操作

```python
# 列表推导
[x * 2 for x in range(10)]
[x for x in items if x > 5]

# 字典推导
{k: v for k, v in items.items() if v > 5}

# 合并字典
d1 = {'a': 1}
d2 = {'b': 2}
d = {**d1, **d2}        # {'a': 1, 'b': 2}
d = d1 | d2              # Python 3.9+

# 排序
sorted(items, key=lambda x: x['age'])           # 正序
sorted(items, key=lambda x: x['age'], reverse=True)  # 倒序

# 分组
from itertools import groupby
for key, group in groupby(items, key=lambda x: x['category']):
    print(key, list(group))
```

## ⚡ 常用技巧

```python
# 一行代码交换变量
a, b = b, a

# 多个变量赋值
a, b, c = 1, 2, 3

# 列表扁平化
from itertools import chain
flat = list(chain.from_iterable([[1,2], [3,4]]))

# 计时
import time
start = time.time()
# do something
print(f'耗时: {time.time() - start:.2f}秒')

# 快速去重（保持顺序）
seen = set()
result = [x for x in items if not (x in seen or seen.add(x))]

# 优雅的判空
data = get_data() or {}

# 默认字典
from collections import defaultdict
d = defaultdict(list)
d['key'].append(1)
```

## 🔄 异步编程

```python
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.json()

async def main():
    tasks = [fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)

asyncio.run(main())
```

## 🧪 单元测试

```python
import unittest

class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(1 + 1, 2)
    
    def test_list(self):
        self.assertIn(1, [1, 2, 3])
        self.assertTrue(True)

if __name__ == '__main__':
    unittest.main()
```
