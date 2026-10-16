# 🗄️ SQL 速查大全（中文版）

> SQL 一站速查：从入门到精通，200+ 常用 SQL 语句模板，覆盖增删改查/索引/事务/优化/窗口函数/多表查询，附 MySQL / PostgreSQL / SQLite 语法差异对照。

![stars](https://img.shields.io/badge/⭐-给个Star吧-red)
![license](https://img.shields.io/badge/license-MIT-blue)
![status](https://img.shields.io/badge/status-持续更新-green)

## 为什么需要这份速查？

你是不是也有这样的经历：

- 写个 SQL 查询，**百度了半天语法**，还写不对？
- LEFT JOIN 和 INNER JOIN **分不清区别**，查出来的数据不对？
- 面试问 SQL 优化，**只会说"加索引"**，一问细节就懵？
- 每次换数据库（MySQL/PostgreSQL/SQLite），**语法差异记不住**？

**SQL 是程序员和数据人的基本功，也是面试必考点。**

这份速查把 SQL 常用知识一次性整理全：

- ✅ **语法速查**：增删改查/聚合/多表/窗口函数，一表看懂
- ✅ **200+ 实战模板**：复制即用，覆盖日常 90% 场景
- ✅ **多数据库对照**：MySQL / PostgreSQL / SQLite 差异都标了
- ✅ **面试考点**：索引原理/事务/优化，直接背答案
- ✅ **持续更新**：欢迎补充更多实战场景

## 📚 目录

### 第一篇：基础语法速查

#### 1. 查询（SELECT）

**基础查询**

```sql
-- 查所有列
SELECT * FROM users;

-- 查指定列
SELECT id, name, email FROM users;

-- 去重
SELECT DISTINCT city FROM users;

-- 条件查询
SELECT * FROM users WHERE age > 18 AND city = '北京';

-- 排序
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 分页（MySQL）
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;  -- 第1页
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10; -- 第2页

-- 模糊查询
SELECT * FROM users WHERE name LIKE '张%';   -- 以张开头
SELECT * FROM users WHERE name LIKE '%三%';  -- 包含三
```

**聚合查询**

```sql
-- 计数
SELECT COUNT(*) FROM users;

-- 求和/平均/最大/最小
SELECT SUM(amount), AVG(amount), MAX(amount), MIN(amount) FROM orders;

-- 分组统计
SELECT city, COUNT(*) AS cnt FROM users GROUP BY city;

-- 分组后过滤（HAVING）
SELECT city, COUNT(*) AS cnt FROM users GROUP BY city HAVING COUNT(*) > 10;

-- 不去重计数
SELECT COUNT(DISTINCT city) FROM users;
```

#### 2. 增删改（INSERT / UPDATE / DELETE）

```sql
-- 插入单条
INSERT INTO users (name, email, age) VALUES ('张三', 'zhangsan@test.com', 25);

-- 批量插入
INSERT INTO users (name, email) VALUES
('李四', 'lisi@test.com'),
('王五', 'wangwu@test.com');

-- 更新
UPDATE users SET age = 26 WHERE name = '张三';

-- 更新多列
UPDATE users SET age = 26, city = '上海' WHERE id = 1;

-- 删除
DELETE FROM users WHERE id = 1;

-- 清空表（保留表结构）
TRUNCATE TABLE users;
```

#### 3. 表结构操作（DDL）

```sql
-- 建表
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT DEFAULT 18,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 加字段
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- 改字段类型
ALTER TABLE users MODIFY COLUMN age INT NOT NULL;

-- 删字段
ALTER TABLE users DROP COLUMN phone;

-- 建索引
CREATE INDEX idx_users_name ON users(name);

-- 唯一索引
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- 删索引
DROP INDEX idx_users_name ON users;

-- 删表
DROP TABLE users;
```

### 第二篇：进阶语法

#### 4. 多表查询（JOIN）

```sql
-- INNER JOIN：只取两表都有
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN：左表全保留，右表没有用 NULL
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN：右表全保留
SELECT u.name, o.amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- 自连接：查员工和上级
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- 三表连接
SELECT u.name, o.amount, p.name AS product
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id;
```

#### 5. 子查询

```sql
-- WHERE 子查询：查比平均年龄大的人
SELECT * FROM users
WHERE age > (SELECT AVG(age) FROM users);

-- FROM 子查询：查每类商品最贵的
SELECT category, MAX(price) AS max_price
FROM (SELECT * FROM products WHERE status = 'active') AS p
GROUP BY category;

-- EXISTS 子查询：查有订单的用户
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- IN 子查询
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders WHERE amount > 1000);
```

#### 6. 窗口函数（MySQL 8+/PostgreSQL）

```sql
-- ROW_NUMBER：行号（常用于分页/去重）
SELECT name, amount,
       ROW_NUMBER() OVER (ORDER BY amount DESC) AS rank
FROM orders;

-- 分组排名
SELECT name, department,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- LAG/LEAD：取前一行/后一行
SELECT name, amount,
       LAG(amount) OVER (ORDER BY date) AS prev_amount
FROM daily_sales;

-- 累计求和
SELECT name, amount,
       SUM(amount) OVER (ORDER BY date) AS cumulative
FROM daily_sales;

-- 移动平均（近3天）
SELECT date, amount,
       AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS ma3
FROM daily_sales;
```

#### 7. 常用函数

**字符串函数**

```sql
SELECT CONCAT(first_name, last_name) FROM users;       -- 拼接
SELECT LENGTH(name) FROM users;                        -- 长度（字节）
SELECT CHAR_LENGTH(name) FROM users;                   -- 字符数（中文1个）
SELECT SUBSTRING(email, 1, 5) FROM users;              -- 截取
SELECT UPPER(name), LOWER(name) FROM users;            -- 大小写
SELECT REPLACE(phone, '-', '') FROM users;             -- 替换
SELECT TRIM('  abc  ');                                -- 去空格
```

**日期函数（MySQL）**

```sql
SELECT NOW();                              -- 当前时间
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d');     -- 格式化
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);    -- 加7天
SELECT DATEDIFF('2026-01-01', '2025-12-25');  -- 日期差
SELECT YEAR(NOW()), MONTH(NOW()), DAY(NOW());  -- 拆分

-- 查最近7天数据
SELECT * FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);
```

**条件函数**

```sql
-- CASE WHEN
SELECT name, age,
       CASE WHEN age < 18 THEN '未成年'
            WHEN age < 60 THEN '成年'
            ELSE '老年' END AS age_group
FROM users;

-- IF（MySQL）
SELECT name, IF(age >= 18, '成年', '未成年') FROM users;

-- COALESCE：取第一个非空
SELECT name, COALESCE(phone, '未填写') FROM users;

-- NULLIF
SELECT NULLIF(a, b);  -- a=b 时返回 NULL
```

#### 8. 事务与锁（MySQL）

```sql
-- 开启事务
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- 提交
-- ROLLBACK;  -- 回滚

-- 手动锁行
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- 隔离级别
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

#### 9. 高级技巧

```sql
-- UNION 合并（去重）
SELECT name FROM teachers UNION SELECT name FROM students;

-- UNION ALL（不去重，快）
SELECT name FROM teachers UNION ALL SELECT name FROM students;

-- 批量更新（CASE 技巧）
UPDATE products SET price = CASE
    WHEN category = 'A' THEN price * 1.1
    WHEN category = 'B' THEN price * 1.05
    ELSE price END;

-- 分组取每组最大（经典）
SELECT * FROM orders o
WHERE o.amount = (SELECT MAX(amount) FROM orders WHERE user_id = o.user_id);

-- 随机取 N 条（MySQL）
SELECT * FROM users ORDER BY RAND() LIMIT 5;

-- 去重保留一条（窗口函数）
DELETE FROM users WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
    FROM users
  ) t WHERE rn > 1
);
```

### 第三篇：面试考点速答

#### 10. 索引

**10.1 索引为什么用 B+ 树？**

**速答：** 树高矮（3 层千万级），减少磁盘 IO；叶子有序 + 链表，范围查询快；非叶子只存键，一页存更多索引。

**10.2 聚簇索引和非聚簇索引？**

**速答：** 聚簇索引（InnoDB 主键）叶子存整行；非聚簇（二级索引）叶子存主键，查完整数据需回表。覆盖索引（查询列都在索引里）免回表。

**10.3 什么时候索引失效？**

**速答：**
- 左模糊：`LIKE '%xx'`
- 函数处理：`WHERE YEAR(created_at) = 2026`
- 隐式转换：字段是字符串，条件传数字
- 联合索引没走最左前缀
- OR 连接非索引列

#### 11. 事务与隔离级别

**11.1 ACID 是什么？**

**速答：** 原子性（undo log）、一致性、隔离性（锁+MVCC）、持久性（redo log）。

**11.2 隔离级别和问题？**

**速答：**
- 读未提交：脏读
- 读已提交：不可重复读（Oracle 默认）
- 可重复读：幻读（MySQL 默认，MVCC + next-key lock 解决）
- 串行化：性能最差

**11.3 MVCC 是什么？**

**速答：** 多版本并发控制，通过 undo log 保存历史版本 + 隐藏列（事务 ID）实现快照读，读写不互斥。

#### 12. SQL 优化

**12.1 一条慢 SQL 怎么排查？**

**速答：**
1. `EXPLAIN SELECT ...` 看执行计划（type/rows/Extra）
2. 看是否全表扫描（type=ALL）
3. 看是否回表、文件排序（filesort）
4. 加合适索引、改写 SQL

**12.2 常见的优化手段？**

**速答：**
- 加索引、避免 `SELECT *`（用覆盖索引）
- 避免函数/隐式转换导致索引失效
- 分页深翻页优化：`WHERE id > last_id ORDER BY id LIMIT 10`
- 大表分库分表 / 读写分离
- 慢查询日志 + 监控

#### 13. MySQL 和 PostgreSQL 的区别？

**速答：**
- 索引：MySQL B+树为主，PG 支持 B+树/哈希/GiST/BRIN 多种
- JSON：PG 功能更强（JSONB 支持索引）
- 窗口函数：PG 一直支持，MySQL 8+ 才支持
- 复制：MySQL 主从，PG 流复制
- 许可证：MySQL 双协议，PG BSD 宽松

### 附录：多数据库语法差异

| 功能 | MySQL | PostgreSQL | SQLite |
|---|---|---|---|
| 自增 | `AUTO_INCREMENT` | `SERIAL` / `GENERATED` | `AUTOINCREMENT` |
| 分页 | `LIMIT n OFFSET m` | `LIMIT n OFFSET m` | `LIMIT n OFFSET m` |
| 字符串拼接 | `CONCAT()` | `\|\|` | `\|\|` |
| 去重保留一条 | `DELETE ... ROW_NUMBER()` | `DISTINCT ON` | 子查询 |
| 当前时间 | `NOW()` | `NOW()` | `datetime('now')` |
| 随机排序 | `ORDER BY RAND()` | `ORDER BY RANDOM()` | `ORDER BY RANDOM()` |

## 🚀 快速开始

```bash
# 直接浏览
# 打开 README.md 即可查看全部内容

# 或者克隆到本地
git clone https://github.com/zieang88888/sql-cheatsheet-zh.git
```

## 🤝 贡献

- 有常用 SQL 模板？提 PR
- 有想补充的场景？提 Issue
- 觉得有用？点个 ⭐ Star 就是最大的支持！

## 📄 License

MIT

---

**如果这份速查帮到了你，请点个 ⭐ Star，让更多程序员告别"SQL 半天写不对"！** 🙏

**关注我 [@zieang88888](https://github.com/zieang88888)，持续输出中文开源干货！**