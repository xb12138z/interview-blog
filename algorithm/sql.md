# 数据库编程基础

## 基础语法

1.SQL语句书写顺序
```sql
SELECT [DISTINCT] 字段1 [AS 别名1], 字段2 [AS 别名2], 聚合函数(字段) [AS 别名]
FROM 表名1 
[JOIN 表名2 ON 连接条件]  -- 多表关联时用
[WHERE 行筛选条件]  -- 分组前筛选行
[GROUP BY 分组字段1, 分组字段2]  -- 按字段分组
[HAVING 分组筛选条件]  -- 分组后筛选分组
[ORDER BY 排序字段 [ASC/DESC]]  -- 结果排序
[LIMIT 行数限制]  -- 限制返回行数（MySQL特有）
```
SELECT：指定要查询的字段 / 聚合结果（比如 name、COUNT(*)），DISTINCT 用于去重；
FROM：指定数据来源的表（必须有）；
JOIN：多表关联（比如订单表关联用户表）；分为innerjoin、outerjoin。inner join：2表值都存在。outer join：附表中值可能存在null的情况。
WHERE：分组前过滤行（只保留满足条件的行，不能用聚合函数）；
GROUP BY：按字段分组（搭配聚合函数使用）；
HAVING：分组后过滤分组（只能用在 GROUP BY 后，可使用聚合函数）；
ORDER BY：对结果排序（ASC 升序，DESC 降序，默认 ASC）；
LIMIT：限制返回的行数（比如只取前 10 条）。

2.聚合函数 —— 常与group by连用，DISTINCT可以用于去重

COUNT()语句: 计算组内行数，内填*的话为计算全部组的函数，内填“字段”的话计算该“字段”组中的非空行数、DISTINCT+字段为统计组内非空字段的行数。

SUM()语句:“字段”计算分组内该字段的和，DISTINCT排除重复值（算出每一个小组的和）

AVG()语句:同上，计算平均数

MAX()\MIN()语句:同上计算最大最小值

3.字符串函数

TRIM(字段)：去除字符串首尾的空格

CONCAT(字段1, 字段2)：拼接多个字符串

SUBSTRING(字段, 起始位, 长度)：截取字符串（起始位从 1 开始）

LENGTH(字段)：计算字符串长度（字符数，MySQL 中中文占 1 个字符，其他数据库可能不同）

UPPER/LOWER(字段)：转大写 / 转小写（对中文无效，用于英文）

REPLACE(字段, 旧值, 新值)：替换字符串中的指定内容

4.日期 / 时间函数（处理时间类数据）

DATE_FORMAT(日期, 格式)：日期格式化（核心！）DATE_FORMAT(trans_date, '%Y-%m') → 年月；%Y-%m-%d → 年月日；%H:%i:%s → 时分秒	

DATE(字段)：提取日期部分（去掉时分秒）

YEAR/MONTH/DAY(字段)提取年 / 月 / 日

DATEDIFF(日期1, 日期2)：计算两个日期的天数差（日期 1 - 日期 2）

DATE_ADD(日期, INTERVAL 数值 单位)：日期加减（单位：DAY/ MONTH/ YEAR 等

NOW() / CURDATE()：获取当前时间（含时分秒）/ 当前日期（仅年月日）

5.判断函数

IF(条件, 满足条件返回值, 不满足返回值)

CASE
  WHEN 条件1 THEN 返回值1
  WHEN 条件2 THEN 返回值2
  ELSE 默认返回值
END

6.数值函数

ROUND(数值, 小数位)四舍五入到规定的小数位置

FLOOR/CEIL(数值)向下、上取整

ABS(数值)绝对值

MOD(数值1, 数值2)取余

## 例题

### 1.175组合两个表

题目：
表一存储personid、firstname、lastname。表二存储personid、city、state。表二中可能没有表一中的人。要求输出表一中所有人的对应的city、state。没有记录则输出null
题解：

```sql
select FirstName, LastName, City, State
from Person left join Address --表二中可能没有用left join
on Person.PersonId = Address.PersonId
;
```

### 2.181超过经理的员工

题目：
表存储  id、name、salary、managerid 四个数据。
题解：

```sql
select A.Name AS employee
from Employee AS A, --一个表解码成两个表
     Employee AS B
WHERE A.managerId = B.id AND A.salary > B.salary
```

