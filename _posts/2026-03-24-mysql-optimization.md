---
layout: post
title: 一次 MySQL 慢查询优化实战全过程
categories: [Database]
description: 记录一次真实的慢查询优化过程，从分析到优化再到验证的完整流程
keywords: MySQL, 慢查询, SQL优化, 索引, EXPLAIN
author: 牛马便利店一号店员
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 这是一篇真实的慢查询优化记录，从问题发现到解决的完整过程

## 问题背景

用户反馈：后台订单列表页面加载要 **8 秒**，必须优化。

表数据量：
- 订单表 `orders`：约 **500 万** 条
- 用户表 `users`：约 **50 万** 条
- 商品表 `products`：约 **10 万** 条

---

## 第一步：找到慢 SQL

### 开启慢查询日志

```sql
-- 查看慢查询配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- 开启慢查询（临时）
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 超过 1 秒记录

-- 查看慢查询日志位置
SHOW VARIABLES LIKE 'slow_query_log_file';
```

### 找到最慢的查询

```sql
-- 查看当前所有连接和正在执行的 SQL
SHOW FULL PROCESSLIST;

-- MySQL 8.0+ 使用 performance_schema
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

### 问题 SQL 定位

用户说的慢 SQL：

```sql
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20 OFFSET 200;
```

这个查询花了 **8.3 秒**。

---

## 第二步：分析 SQL

### EXPLAIN 分析执行计划

```sql
EXPLAIN 
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20 OFFSET 200;
```

结果分析：

| 字段 | 值 | 说明 |
|------|-----|------|
| type | ALL | **全表扫描！** |
| key | NULL | 没走索引 |
| rows | 5000000 | 扫描了 500 万行！ |
| Extra | Using filesort | **文件排序** |

问题找到了：
1. `status` 字段没索引
2. `ORDER BY created_at` 也没索引
3. `OFFSET 200` 大偏移量很慢

---

## 第三步：加索引

### 分析 WHERE 和 ORDER BY

```sql
-- 经常查询 status=1 的订单，按时间倒序
-- 需要复合索引：(status, created_at)
ALTER TABLE orders ADD INDEX idx_status_created (status, created_at DESC);
```

### 验证索引

```sql
EXPLAIN 
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20 OFFSET 200;
```

优化后：

| 字段 | 优化前 | 优化后 |
|------|--------|--------|
| type | ALL | range |
| key | NULL | idx_status_created |
| rows | 5000000 | 约 50000 |
| Extra | Using filesort | Using index condition |

**但是 OFFSET 还是问题。**

---

## 第四步：解决 OFFSET 问题

### 为什么要避免大 OFFSET

```sql
LIMIT 20 OFFSET 1000000;
-- MySQL 会扫描前 1000020 行，然后扔掉前 1000000 行
```

### 方案一：游标分页（推荐）

```sql
-- 第一次查询
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20;

-- 记住最后一行的 created_at
-- 假设是 2024-01-15 10:30:00

-- 第二次查询（下一页）
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
  AND o.created_at < '2024-01-15 10:30:00'  -- 用时间做游标
ORDER BY o.created_at DESC
LIMIT 20;
```

### 方案二：用主键做游标

```sql
-- 如果时间可能重复，用复合游标
WHERE o.status = 1
  AND (o.created_at, o.id) < ('2024-01-15 10:30:00', 99999)
ORDER BY o.created_at DESC, o.id DESC
LIMIT 20;
```

---

## 第五步：优化 JOIN

### EXPLAIN 分析 JOIN 成本

```sql
EXPLAIN 
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20;
```

发现 `users` 和 `products` 也是全表扫描，给它们加索引：

```sql
ALTER TABLE users ADD INDEX idx_id (id);        -- 一般都有
ALTER TABLE products ADD INDEX idx_id (id);      -- 一般都有
```

### 用 STRAIGHT_JOIN 强制顺序

如果 JOIN 顺序不对，可以用 `STRAIGHT_JOIN`：

```sql
SELECT STRAIGHT_JOIN 
    o.*, u.username, p.name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20;
```

---

## 第六步：验证结果

### 优化后测试

```sql
-- 用游标分页
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
  AND o.created_at < '2024-01-15 10:30:00'
ORDER BY o.created_at DESC
LIMIT 20;

-- 对比执行时间
SELECT BENCHMARK(1, <SQL>);
```

### APM 工具监控

部署 [PMM](https://www.percona.com/doc/percona-monitoring-and-management/index.html) 或 [SkyWalking](https://skywalking.apache.org/) 持续监控慢 SQL。

---

## 完整优化后的 SQL

```sql
-- 第一页
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
ORDER BY o.created_at DESC
LIMIT 20;

-- 后续页面（带游标）
SELECT o.*, u.username, u.email, p.name as product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.status = 1
  AND o.created_at < :last_created_at
ORDER BY o.created_at DESC
LIMIT 20;
```

**结果：从 8.3 秒 → 0.02 秒，快了 400 倍。**

---

## 经验总结

### 常见慢查询原因

| 原因 | 解决方案 |
|------|----------|
| WHERE 字段无索引 | 加索引 |
| ORDER BY 字段无索引 | 加索引 |
| 大 OFFSET 分页 | 改用游标分页 |
| SELECT * | 只查需要的字段 |
| JOIN 无索引字段 | 确保 JOIN 字段有索引 |
| 函数操作字段 | 改写 SQL，避免函数 |

### 加索引的原则

1. **高频 WHERE 字段** — 优先加
2. **ORDER BY 字段** — 如果是 filesort
3. **复合索引顺序** — 等于 > 范围 > 排序 > 覆盖
4. **不要太多** — 每个索引都占用空间，写入变慢

### 养成习惯

```sql
-- 上线前必查执行计划
EXPLAIN your_query;

-- 监控慢查询
SHOW FULL PROCESSLIST;
SELECT * FROM performance_schema.events_statements_summary_by_digest;
```

---

## 附：常用分析命令

```sql
-- 查看表状态
SHOW TABLE STATUS FROM db_name LIKE 'orders';

-- 查看索引
SHOW INDEX FROM orders;

-- 查看锁等待
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 查看连接数
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

---

*作者：牛马便利店一号店员*
