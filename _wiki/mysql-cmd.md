---
layout: wiki
title: MySQL 常用命令
cate1: MySQL
cate2: 数据库命令
description: MySQL 常用命令速查，包含连接、库表操作、用户权限
keywords: MySQL, SQL, 数据库, MariaDB
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

> 💡 常用工具：mysql CLI、MySQL Workbench、Navicat、DBeaver

## 🔌 连接管理

```bash
# 命令行连接
mysql -u root -p              # 本地连接
mysql -h 127.0.0.1 -P 3306 -u root -p   # 指定主机和端口
mysql -u root -p -e "SQL"    # 执行单条 SQL 后退出

# 断开连接
EXIT 或 QUIT 或 \q
```

## 📊 数据库操作

```sql
-- 查看数据库
SHOW DATABASES;

-- 创建数据库
CREATE DATABASE db_name;
CREATE DATABASE db_name DEFAULT CHARSET utf8mb4;

-- 删除数据库
DROP DATABASE db_name;

-- 使用数据库
USE db_name;

-- 查看当前数据库
SELECT DATABASE();
```

## 📋 表操作

```sql
-- 查看表
SHOW TABLES;
SHOW TABLES FROM db_name;

-- 创建表
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status TINYINT DEFAULT 1
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 查看表结构
DESC table_name;
DESCRIBE table_name;
SHOW CREATE TABLE table_name;

-- 删除表
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;

-- 修改表
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users MODIFY COLUMN username VARCHAR(100);
ALTER TABLE users RENAME TO user_info;
```

## 📝 数据的增删改查 (CRUD)

```sql
-- 插入数据
INSERT INTO users (username, password, email) VALUES ('tom', '123456', 'tom@email.com');
INSERT INTO users VALUES (NULL, 'tom', '123456', 'tom@email.com', NOW(), 1);
INSERT INTO users (username, email) VALUES 
    ('a', 'a@email.com'),
    ('b', 'b@email.com');

-- 查询数据
SELECT * FROM users;
SELECT username, email FROM users;
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE status = 1 AND username LIKE 't%';
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users LIMIT 10 OFFSET 20;
SELECT DISTINCT status FROM users;

-- 更新数据
UPDATE users SET email = 'new@email.com' WHERE id = 1;
UPDATE users SET status = 0, email = '' WHERE username = 'tom';

-- 删除数据
DELETE FROM users WHERE id = 1;
DELETE FROM users;  -- 删除全部（谨慎！）

-- 清空表（自增重置）
TRUNCATE TABLE users;
```

## 🔍 条件查询

```sql
-- WHERE 条件
SELECT * FROM users WHERE age >= 18;
SELECT * FROM users WHERE age BETWEEN 18 AND 30;
SELECT * FROM users WHERE status IN (1, 2, 3);
SELECT * FROM users WHERE name IS NULL;
SELECT * FROM users WHERE name IS NOT NULL;

-- 聚合函数
SELECT COUNT(*) FROM users;
SELECT SUM(price) FROM orders;
SELECT AVG(age) FROM users;
SELECT MAX(salary) FROM emp;
SELECT MIN(age) FROM users;

-- 分组
SELECT status, COUNT(*) FROM users GROUP BY status;
SELECT category, SUM(price) FROM products GROUP BY category HAVING SUM(price) > 1000;

-- 子查询
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);
SELECT * FROM users WHERE age > (SELECT AVG(age) FROM users);

-- JOIN 查询
SELECT u.username, o.order_no 
FROM users u 
INNER JOIN orders o ON u.id = o.user_id;

SELECT u.username, o.order_no 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id;
```

## 👤 用户与权限

```sql
-- 查看用户（MySQL 5.7+）
SELECT user, host FROM mysql.user;

-- 创建用户
CREATE USER 'tom'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'tom'@'%' IDENTIFIED BY 'password123';  -- 允许远程

-- 删除用户
DROP USER 'tom'@'localhost';

-- 授权
GRANT ALL PRIVILEGES ON db_name.* TO 'tom'@'localhost';
GRANT SELECT, INSERT, UPDATE ON db_name.table TO 'tom'@'localhost';
GRANT ALL ON *.* TO 'tom'@'localhost' WITH GRANT OPTION;  -- 管理员

-- 刷新权限
FLUSH PRIVILEGES;

-- 查看权限
SHOW GRANTS FOR 'tom'@'localhost';

-- 撤销权限
REVOKE INSERT ON db_name.* FROM 'tom'@'localhost';
```

## ⚙️ 索引操作

```sql
-- 查看索引
SHOW INDEX FROM table_name;

-- 创建索引
CREATE INDEX idx_username ON users(username);
CREATE UNIQUE INDEX idx_email ON users(email);
CREATE INDEX idx_composite ON orders(user_id, created_at);

-- 删除索引
DROP INDEX idx_username ON users;
ALTER TABLE users DROP INDEX idx_email;
```

## 🔧 导入导出

```bash
# 导出 SQL 文件
mysqldump -u root -p db_name > db_name.sql
mysqldump -u root -p db_name table1 table2 > tables.sql
mysqldump -u root -p --all-databases > all.sql

# 导入 SQL 文件
mysql -u root -p db_name < db_name.sql
```

## 📈 常用运维

```sql
-- 查看状态
SHOW STATUS;
SHOW VARIABLES;

-- 查看进程
SHOW PROCESSLIST;
KILL 12345;

-- 查看慢查询
SHOW VARIABLES LIKE 'slow%';
SHOW VARIABLES LIKE 'long_query_time';

-- 修复表
REPAIR TABLE table_name;
OPTIMIZE TABLE table_name;

-- 分析表
ANALYZE TABLE table_name;
```
