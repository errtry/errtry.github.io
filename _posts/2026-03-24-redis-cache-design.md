---
layout: post
title: Redis 缓存设计与踩坑实录
categories: [Database]
description: 从实际项目出发，讲 Redis 缓存的设计方法与常见坑，包括缓存穿透、击穿、雪崩、热key问题
keywords: Redis, 缓存, 缓存穿透, 缓存击穿, 缓存雪崩, 热key
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇 Redis 缓存实战指南，基于真实踩坑经验

## 为什么要用缓存？

先看个数据：

| 操作 | 耗时 |
|------|------|
| 查 MySQL | 10-50ms |
| 查 Redis | 0.1-0.5ms |
| 差了 | **50-500 倍** |

缓存就是把热点数据放内存里，让读取快到飞起。

---

## 一、缓存读写模式

### 模式 1：Cache Aside（最常用）

```
读：
  查询缓存 → 有 → 直接返回
            ↓ 没有
          查数据库 → 写入缓存 → 返回

写：
  更新数据库 → 删除缓存（不是更新！）
```

代码示例：

```python
def get_user(user_id):
    # 1. 先查缓存
    cache_key = f"user:{user_id}"
    user = redis.get(cache_key)
    
    if user:
        return json.loads(user)
    
    # 2. 缓存没有，查数据库
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    if user:
        # 3. 写入缓存，过期时间 1 小时
        redis.setex(cache_key, 3600, json.dumps(user))
    
    return user


def update_user(user_id, data):
    # 1. 更新数据库
    db.execute("UPDATE users SET name = ? WHERE id = ?", 
               data['name'], user_id)
    
    # 2. 删除缓存（不是更新！）
    # 为什么删而不是更新？因为更新可能不一致
    redis.delete(f"user:{user_id}")
```

**为什么写的时候删缓存而不是更新？**

```
场景：ABC三个操作同时来
  A: 更新数据库 name='张三'
  B: 更新数据库 name='李四'
  C: 查缓存没有，查数据库拿到 name='张三'
  A: 更新缓存 name='张三'
  C: 写缓存 name='张三'
  B: 删除缓存

最终：缓存删了，下次查是最新值 ✓
```

---

### 模式 2：Read Through

缓存自动加载，代码更简单：

```python
def get_user(user_id):
    cache_key = f"user:{user_id}"
    
    # 缓存不存在时，缓存层自动加载
    user = cache.get(cache_key, lambda: db.query_user(user_id))
    
    return user
```

### 模式 3：Write Through

写的时候同步更新缓存：

```python
def update_user(user_id, data):
    db.execute("UPDATE users SET ...", ...)
    cache.set(f"user:{user_id}", data)  # 同步写缓存
```

---

## 二、缓存过期策略

### Redis 的 8 种淘汰策略

| 策略 | 说明 |
|------|------|
| `noeviction` | 不淘汰，内存满返回错误（默认） |
| `volatile-lru` | 过期键中淘汰最近最少使用 |
| `volatile-lfu` | 过期键中淘汰使用频率最低 |
| `volatile-ttl` | 过期键中淘汰剩余时间最短 |
| `volatile-random` | 过期键中随机淘汰 |
| `allkeys-lru` | 所有键中淘汰最近最少使用 |
| `allkeys-lfu` | 所有键中淘汰使用频率最低 |
| `allkeys-random` | 所有键中随机淘汰 |

**配置示例：**

```redis
# redis.conf
maxmemory 2gb              # 最大内存 2GB
maxmemory-policy allkeys-lru  # 内存满时淘汰最少使用的键
```

**怎么选？**

```
大部分场景：allkeys-lru
读多写少、热点明显：allkeys-lru
所有 key 都一样重要：allkeys-random
需要精确控制过期：volatile-lru + 过期时间
```

---

## 三、三大经典问题

### 1. 缓存穿透

**问题**：查询不存在的数据，每次都打到数据库

```
查询 user_id=999999（不存在）
  → Redis 没有
  → MySQL 也没有
  → 返回空
  → 但每次都查 MySQL！
  
如果有人恶意刷：MySQL 直接崩
```

**解决方案：**

#### 方案 A：缓存空值

```python
def get_user(user_id):
    cache_key = f"user:{user_id}"
    user = redis.get(cache_key)
    
    if user is not None:
        if user == '__NULL__':
            return None  # 缓存的空值
        return json.loads(user)
    
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    if user:
        redis.setex(cache_key, 3600, json.dumps(user))
    else:
        # 缓存空值，过期时间短一点
        redis.setex(cache_key, 60, '__NULL__')
    
    return user
```

#### 方案 B：布隆过滤器

```python
from bloom_filter import BloomFilter

bf = BloomFilter(max_elements=1000000, error_rate=0.01)

# 初始化：把所有存在的 user_id 加入布隆过滤器
for user_id in db.query_all_user_ids():
    bf.add(str(user_id))

# 查询时先判断
def get_user(user_id):
    if str(user_id) not in bf:
        return None  # 一定不存在，直接返回
    
    # 布隆过滤器说可能存在，再查缓存和数据库
    ...
```

#### 方案 C：接口限流

```python
def rate_limit(user_id):
    key = f"rate_limit:get_user:{user_id}"
    count = redis.incr(key)
    if count == 1:
        redis.expire(key, 60)  # 60秒内最多60次
    return count > 60

def get_user(user_id):
    if rate_limit(user_id):
        raise Exception("请求太频繁")
    ...
```

---

### 2. 缓存击穿

**问题**：热点 key 过期瞬间，大量请求打到数据库

```
热点文章 A（100万人在看）
  → Redis 过期了
  → 100万个请求同时查 MySQL
  → MySQL 扛不住，直接崩
```

**解决方案：**

#### 方案 A：互斥锁（分布式锁）

```python
import redis
import time

def get_user_with_lock(user_id):
    cache_key = f"user:{user_id}"
    
    # 先查缓存
    user = redis.get(cache_key)
    if user:
        return json.loads(user)
    
    # 获取锁
    lock_key = f"lock:user:{user_id}"
    lock = redis.set(lock_key, '1', nx=True, ex=10)
    
    if lock:
        try:
            # 拿到锁的请求查数据库
            user = db.query("SELECT * FROM users WHERE id = ?", user_id)
            if user:
                redis.setex(cache_key, 3600, json.dumps(user))
            return user
        finally:
            redis.delete(lock_key)
    else:
        # 没拿到锁，等一下再试
        time.sleep(0.1)
        return get_user_with_lock(user_id)  # 递归重试
```

#### 方案 B：逻辑过期

不设真正的过期时间，而是在读取时"软判断"：

```python
def get_user(user_id):
    cache_key = f"user:{user_id}"
    data = redis.get(cache_key)
    
    if not data:
        return db.query_user(user_id)
    
    user = json.loads(data)
    
    # 软过期：超过30分钟就返回过期数据，同时异步重建
    if time.time() - user['cache_time'] > 1800:
        # 返回旧数据（不阻塞）
        Thread(target=rebuild_cache, args=(user_id,)).start()
        return user
    
    return user


def rebuild_cache(user_id):
    user = db.query_user(user_id)
    user['cache_time'] = time.time()
    redis.setex(f"user:{user_id}", 3600, json.dumps(user))
```

#### 方案 C：永不过期

热点数据直接不过期，靠后台任务更新：

```python
# 定时任务，每小时刷新缓存
def refresh_user_cache():
    user_ids = get_hot_user_ids()  # 获取热点用户ID
    
    for user_id in user_ids:
        user = db.query_user(user_id)
        redis.set(f"user:{user_id}", json.dumps(user))
```

---

### 3. 缓存雪崩

**问题**：大量 key 同时过期，瞬间大量请求打到数据库

```
双十一零点
  → 10000 个商品缓存同时过期
  → 10000 个请求同时查 MySQL
  → MySQL 原地爆炸
```

**解决方案：**

#### 方案 A：过期时间加随机值

```python
# 不要这样：统一过期
redis.setex("product:1", 3600, data)
redis.setex("product:2", 3600, data)
# 10000个都3600秒，同时过期！

# 这样做：过期时间加随机
redis.setex("product:1", 3600 + random.randint(0, 600), data)
redis.setex("product:2", 3600 + random.randint(0, 600), data)
```

#### 方案 B：多级缓存

```
L1: Caffeine（本机缓存，100ms TTL）
L2: Redis（分布式缓存，10分钟 TTL）
L3: MySQL
```

```python
from caffeine import Caffeine

local_cache = Caffeine(max_size=10000, ttl=100)

def get_product(product_id):
    # L1: 本机缓存
    product = local_cache.get(product_id)
    if product:
        return product
    
    # L2: Redis
    product = redis.get(f"product:{product_id}")
    if product:
        local_cache.set(product_id, product)
        return json.loads(product)
    
    # L3: MySQL
    product = db.query("SELECT * FROM products WHERE id = ?", product_id)
    if product:
        redis.setex(f"product:{product_id}", 600, json.dumps(product))
        local_cache.set(product_id, product)
    
    return product
```

---

## 四、热 Key 问题

**问题**：某个 key 访问量太大，一个 Redis 实例扛不住

```
某个明星发微博
  → 几亿人查这个明星的资料
  → 单个 Redis 的网卡打满
  → 瘫痪
```

**解决方案：**

### 1. Redis Cluster 打散

```
key = "user:12345"
→ 根据 key 哈希分到不同节点
→ 100个节点，每个节点只扛 1%
```

```python
def get_hot_data(key):
    # 一致性哈希或 Redis Cluster 自动路由
    return redis_cluster.get(key)
```

### 2. 本地缓存

```python
from functools import lru_cache

@lru_cache(maxsize=10000)
def get_hot_user(user_id):
    return redis.get(f"user:{user_id}")
```

### 3. Key 打散

```
热点 key: "product:10086"
→ 拆成 10 个 key
  "product:10086:0"
  "product:10086:1"
  ...
  "product:10086:9"
→ 查的时候随机选一个
```

```python
def get_product(product_id):
    # 随机选一个分片
    shard = random.randint(0, 9)
    key = f"product:{product_id}:{shard}"
    return redis.get(key)
```

---

## 五、缓存与数据库一致性

这是最容易出问题的地方。

### 先删缓存再更新数据库

```python
# 场景：线程A更新，线程B查询
def update_user(user_id, data):
    redis.delete(f"user:{user_id}")      # A: 删缓存
    db.execute("UPDATE users SET ...",)  # A: 更新数据库
    
    # B: 此时缓存为空，查到旧数据，写入缓存
    # A和B都没发现问题，但数据不一致了！
```

### 先更新数据库再删缓存

```python
def update_user(user_id, data):
    db.execute("UPDATE users SET ...",)  # A: 更新数据库
    redis.delete(f"user:{user_id}")     # A: 删缓存
    
    # B: 查到旧数据，写入缓存（缓存已删，可能发生）
    # 概率低，但存在
```

### 推荐：延迟双删

```python
def update_user(user_id, data):
    # 1. 先删缓存
    redis.delete(f"user:{user_id}")
    
    # 2. 更新数据库
    db.execute("UPDATE users SET ...",)
    
    # 3. 延迟500ms再删一次（解决并发问题）
    time.sleep(0.5)
    redis.delete(f"user:{user_id}")
```

### 最终方案：订阅 binlog

用 Canal 监听 MySQL binlog，异步更新 Redis：

```
MySQL 更新
    ↓
Canal 监听 binlog
    ↓
解析并更新 Redis
    ↓
最终一致（延迟 < 1秒）
```

```python
# Canal 伪代码
canal = CanalClient(host='127.0.0.1', port=11111)
canal.subscribe()

for entry in canal.entries():
    if entry.event_type == 'UPDATE':
        table = entry.table_name
        data = entry.after_data
        
        if table == 'users':
            redis.set(f"user:{data['id']}", json.dumps(data))
```

---

## 六、实战配置

### 配置文件

```redis
# redis.conf

# 内存
maxmemory 4gb
maxmemory-policy allkeys-lru

# 持久化（RDB + AOF）
save 900 1      # 15分钟内有1次写就保存
save 300 10     # 5分钟内有10次写就保存
save 60 10000   # 1分钟内有10000次写就保存

appendonly yes
appendfsync everysec  # 每秒同步，性能和安全性平衡

# 网络
timeout 300           # 客户端idle 5分钟断开
tcp-keepalive 60      # TCP保活

# 日志
loglevel notice
```

### 监控指标

```bash
# 查看 Redis 状态
redis-cli info

# 关键指标
redis-cli info stats | grep -E "keyspace_hits|keyspace_misses"
# hit rate = hits / (hits + misses)

redis-cli info memory | grep -E "used_memory_human|maxmemory_human"

redis-cli info stats | grep -E "instantaneous_ops_per_sec"
```

---

## 七、总结

| 问题 | 解决方案 |
|------|----------|
| 缓存穿透 | 缓存空值 / 布隆过滤器 / 限流 |
| 缓存击穿 | 分布式锁 / 逻辑过期 / 永不过期 |
| 缓存雪崩 | 过期时间+随机 / 多级缓存 |
| 热 Key | 打散 / 本地缓存 / Cluster |
| 一致性 | 延迟双删 / binlog 同步 |

**记住：**

```
1. 缓存是提升性能用的，不要为了缓存而缓存
2. 一致性和性能是矛盾的，做好取舍
3. 先保证可用，再考虑性能
4. 监控比优化更重要
```

---

*作者：牛马便利店一号店员*
