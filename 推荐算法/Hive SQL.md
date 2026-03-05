#### Hive SQL

```sql
-- 指定列去重（核心列去重，常用）
SELECT DISTINCT col1, col2 FROM table_name;

-- 基础条件：等于、不等于、大于/小于
SELECT * FROM table_name WHERE col1 = 'value' AND col2 is null；

-- 范围条件：BETWEEN（闭区间）、IN
SELECT * FROM table_name WHERE col3 BETWEEN 10 AND 20;
SELECT * FROM table_name WHERE col4 IN ('a', 'b', 'c');

-- 模糊查询：LIKE（%任意字符，_单个字符）
SELECT * FROM table_name WHERE col5 LIKE 'prefix%';

-- 常用聚合：计数、求和、平均值、最值
SELECT
    col1, -- 分组列
    COUNT(*) AS total, -- 总行数（含非空）
    COUNT(col2) AS col2_count, -- 列非空值计数
    SUM(col3) AS col3_sum, -- 求和
    AVG(col3) AS col3_avg, -- 平均值
    MAX(col4) AS col4_max, -- 最大值
    MIN(col4) AS col4_min -- 最小值
FROM 
	table_name
GROUP BY 
	col1; -- 分组列必须出现在SELECT中
HAVING 
	sum_val > 1000;
	
-- 空值处理
SELECT COALESCE(col1, 0, '默认值') AS new_col FROM table_name;

-- 多条件组合
SELECT
    col1,
    CASE
        WHEN col2 = 'a' AND col3 > 0 THEN '类型1'
        WHEN col2 = 'b' OR col3 < 0 THEN '类型2'
        ELSE '其他'
    END AS type
FROM 
	table_name;
	
-- 内连接（INNER JOIN）：仅保留双方匹配的数据(默认)
-- 左连接（LEFT JOIN）：保留左表所有数据，右表匹配不到则为NULL（最常用）
-- 右连接（RIGHT JOIN）：保留右表所有数据，左表匹配不到则为NULL
-- 全连接（FULL JOIN）：保留双方所有数据，匹配不到则为NULL
SELECT 
	a.*, 
	b.col2
FROM table_a a
XXx JOIN table_b b
ON a.id= b.id;
```

