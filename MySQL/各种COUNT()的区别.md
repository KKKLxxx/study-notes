# 各种COUNT()的区别

COUNT 是 SQL 中最常用的聚合函数之一，但不同用法在性能、结果和原理上有重要区别。

## 一、基本语法和结果对比

### 1. COUNT(\*)

```sql
-- 统计结果集的总行数，包括NULL行
SELECT COUNT(*) FROM table_name;
```

**特点**：统计所有行，不考虑列值是否为NULL

### 2. COUNT(1)

```sql
-- 统计结果集的总行数，包括NULL行
SELECT COUNT(1) FROM table_name;
```

**特点**：统计所有行，1是常量表达式

### 3. COUNT(主键)

```sql
-- 统计主键列非NULL的行数
SELECT COUNT(id) FROM table_name;
```

**特点**：统计主键不为NULL的行数（主键不可能为NULL，所以通常等于总行数）

### 4. COUNT(普通列)

```sql
-- 统计该列非NULL的行数
SELECT COUNT(name) FROM table_name;
```

**特点**：只统计该列不为NULL的行数

### 5. COUNT(DISTINCT 列)

```sql
-- 统计该列去重后非NULL的行数
SELECT COUNT(DISTINCT name) FROM table_name;
```

**特点**：去重后统计非NULL值

## 二、执行原理和性能分析

### 1. COUNT(*) 的工作原理

**MySQL InnoDB 优化**：

- MySQL 5.7.18 之前：
  - InnoDB会扫描主键索引（聚簇索引）来统计行数
- MySQL 5.7.18 及之后：
  - InnoDB会选择一个**非NULL的最小二级索引**来统计（加载的数据量更小）
  - 如果没有二级索引，则扫描主键索引

### 2. COUNT(1) 与 COUNT(*) 的区别

**理论区别**：

- `COUNT(*)`：SQL标准写法，统计行数
- `COUNT(1)`：统计常量表达式1非NULL的行数

**实际执行**：

```sql
-- 在大多数数据库中，优化器会将它们视为相同
-- EXPLAIN 查看执行计划通常相同
EXPLAIN SELECT COUNT(*) FROM users;
EXPLAIN SELECT COUNT(1) FROM users;
```

## 三、性能对比测试

| 查询方式          | 时间复杂度      | 使用索引     | 推荐度 |
| :---------------- | :-------------- | :----------- | :----- |
| `COUNT(*)`        | O(最小索引大小) | 最小可用索引 | ★★★★★  |
| `COUNT(1)`        | O(最小索引大小) | 最小可用索引 | ★★★★★  |
| `COUNT(主键)`     | O(主键索引大小) | 主键索引     | ★★★★☆  |
| `COUNT(索引列)`   | O(该索引大小)   | 该列索引     | ★★★☆☆  |
| `COUNT(非索引列)` | O(表大小)       | 无，全表扫描 | ★☆☆☆☆  |

## 四、不同场景下的最佳实践

### 场景1：统计总行数

```sql
-- 最佳选择：COUNT(*) 或 COUNT(1)
SELECT COUNT(*) FROM table_name;

-- 理由：
-- 1. 语义明确，统计行数
-- 2. 数据库有专门优化
-- 3. SQL标准写法
```

### 场景2：统计某列非NULL值的数量

```sql
-- 使用 COUNT(column_name)
SELECT COUNT(email) FROM users;  -- 统计有邮箱的用户数

-- 注意：这不等同于总行数，因为可能有NULL
```

## 五、优化方案

### 1. MyISAM引擎的特殊优化

```sql
-- MyISAM引擎会缓存表的总行数
-- 没有WHERE条件的COUNT(*)是O(1)复杂度
SELECT COUNT(*) FROM myisam_table;  -- 极快
```

### 2. InnoDB优化方案

由于InnoDB需要支持事务，在MVCC的控制下，如果像MyISAM一样缓存表的总行数，会破坏事务的隔离性

**近似值方案**：当业务上并不需要返回准确行数时，可通过EXPLAIN指令快速得到近似行数，因为EXPLAIN并不会真正执行指令，而其返回的rows字段表示执行查询时必须检查的行数，可以作为近似值来返回

还有就是用另一张表去记录某张表的行数，但感觉不太可靠