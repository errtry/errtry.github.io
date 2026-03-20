---
layout: wiki
title: Redis 常用命令
cate1: Redis
cate2: 内存数据库
description: Redis 常用命令速查，包含字符串、哈希、列表、集合、有序集合
keywords: Redis, 缓存, NoSQL, 内存数据库
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 Redis 默认端口：6379，连接：`redis-cli`

## 🔌 连接管理

```bash
# 连接
redis-cli                    # 本地
redis-cli -h 127.0.0.1 -p 6379
redis-cli -a password        # 认证

# 常用选项
redis-cli --scan            # 扫描所有 key
redis-cli --bigkeys         # 查找大 key
redis-cli --hotkeys         # 查找热 key
```

## 🔤 通用命令

```bash
PING                        # 测试连接，返回 PONG
SELECT 0                    # 切换数据库（0-15）
DBSIZE                     # 当前数据库 key 数量
KEYS pattern               # 查找 key（生产禁用！）
SCAN 0 MATCH user:*        # 游标扫描
EXISTS key                 # 判断 key 是否存在
TYPE key                   # 查看类型
DEL key                    # 删除
EXPIRE key 60              # 设置过期（秒）
TTL key                    # 查看剩余生存时间
PERSIST key                # 移除过期
RENAME key newkey          # 重命名
```

## 📝 String（字符串）

```bash
SET key value               # 设置
GET key                     # 获取
MGET key1 key2             # 批量获取
MSET key1 v1 key2 v2       # 批量设置

SETEX key 60 value         # 设置并过期
PSETEX key 60000 value     # 毫秒过期

APPEND key "abc"           # 追加
STRLEN key                 # 长度

# 数字操作
INCR counter               # +1
DECR counter               # -1
INCRBY counter 10          # +10
DECRBY counter 5           # -5
INCRBYFLOAT counter 1.5    # +1.5

# 截取
GETRANGE key 0 -1          # 全部
SETRANGE key 5 "xxx"       # 替换
```

## 📋 Hash（哈希）

```bash
HSET user:1 name tom age 20         # 设置
HGET user:1 name                    # 获取
HMGET user:1 name age               # 批量获取
HGETALL user:1                      # 获取全部

HINCRBY user:1 age 1               # 数字 +1
HEXISTS user:1 name                # 字段是否存在
HLEN user:1                        # 字段数量
HKEYS user:1                       # 所有字段
HVALUES user:1                     # 所有值
HDEL user:1 age                    # 删除字段
```

## 📜 List（列表）

```bash
LPUSH list a b c                    # 左边插入
RPUSH list 1 2 3                    # 右边插入

LRANGE list 0 -1                   # 获取全部
LINDEX list 0                       # 按索引获取
LLEN list                           # 长度

LPOP list                           # 左边弹出
RPOP list                           # 右边弹出

LSET list 0 new                     # 按索引设置
LINSERT list BEFORE a new           # 插入
LTRIM list 0 9                      # 截取保留
```

## 🎯 Set（集合）

```bash
SADD tags python java go             # 添加
SMEMBERS tags                       # 成员
SISMEMBERS tags python              # 是否存在
SCARD tags                          # 数量

SREM tags go                        # 删除
SPOP tags                          # 随机弹出
SRANDMEMBER tags 2                 # 随机获取

SUNION set1 set2                    # 并集
SINTER set1 set2                    # 交集
SDIFF set1 set2                     # 差集
```

## 🏆 Sorted Set（有序集合）

```bash
ZADD leaderboard 100 tom 90 jerry 80 bob   # 添加
ZRANGE leaderboard 0 -1 WITHSCORES         # 正序
ZREVRANGE leaderboard 0 -1 WITHSCORES      # 倒序

ZINCRBY leaderboard 50 tom                  # 增加分数
ZSCORE leaderboard tom                       # 获取分数
ZRANK leaderboard tom                        # 排名（正序）
ZREVRANK leaderboard tom                     # 排名（倒序）

ZREM leaderboard bob                         # 删除
```

## ⏰ Keyspace 通知

```bash
# 配置（redis.conf 或运行时）
notify-keyspace-events Ex

# 订阅过期事件
 PSUBSCRIBE __keyevent@0__:expired
```

## ⚙️ 运维命令

```bash
# 持久化
BGSAVE                   # 后台保存
LASTSAVE                 # 上次保存时间
BGREWRITEAOF            # 重写 AOF

# 服务器
INFO                    # 详细信息
INFO memory             # 内存信息
CONFIG GET maxmemory    # 查看配置
CONFIG SET maxmemory 1gb # 设置配置

# 客户端
CLIENT LIST             # 连接列表
CLIENT KILL ip:port    # 杀掉连接

# 内存
MEMORY USAGE key       # key 占用内存
MEMORY STATS           # 内存统计
```

## 🔄 事务

```bash
MULTI                   # 开始
SET a 1
INCR a
EXEC                    # 执行

# 管道
redis-cli --pipe < data.redis
```

## 🐍 Python 使用

```python
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# String
r.set('name', 'tom')
print(r.get('name'))

# Hash
r.hset('user:1', 'name', 'tom')
r.hgetall('user:1')

# List
r.lpush('list', 'a', 'b')
r.lrange('list', 0, -1)

# Set
r.sadd('tags', 'python', 'java')
r.smembers('tags')

# 有序集合
r.zadd('leaderboard', {'tom': 100})
r.zrevrange('leaderboard', 0, -1, withscores=True)

# 管道
pipe = r.pipeline()
pipe.set('a', 1).incr('a').get('a')
print(pipe.execute())
```
