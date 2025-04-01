# MySQL练习笔记2

<a id="catalog">目录</a>

[TOC]

## 一、排序

### rank() vs dense_rank()

**rank()有跳跃，而dense_rank()无跳跃**

### 题目一：查询分数的排名

表: `Scores`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| score       | decimal |
+-------------+---------+
id 是该表的主键（有不同值的列）。
该表的每一行都包含了一场比赛的分数。Score 是一个有两位小数点的浮点值。
```

 编写一个解决方案来查询分数的排名。排名按以下规则计算:

- 分数应按从高到低排列。
- 如果两个分数相等，那么两个分数的排名应该相同。
- 在排名相同的分数后，排名数应该是下一个连续的整数。换句话说，排名之间不应该有空缺的数字。

按 `score` 降序返回结果表。

```mysql
select score, ranks as 'rank'
from(
    select score,
            dense_rank() over(
                order by score desc
            ) as ranks
    from Scores
) as rnk;
```

### 题目二：查询最高员工工资

表： `Employee`

```
+--------------+---------+
| 列名          | 类型    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
在 SQL 中，id是此表的主键。
departmentId 是 Department 表中 id 的外键（在 Pandas 中称为 join key）。
此表的每一行都表示员工的 id、姓名和工资。它还包含他们所在部门的 id。
```

 表： `Department`

```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
在 SQL 中，id 是此表的主键列。
此表的每一行都表示一个部门的 id 及其名称。
```

查找出每个部门中薪资最高的员工。
按 **任意顺序** 返回结果表。

```mysql
select
    d.name as department, 
    drnk.name as employee,
    drnk.salary
from(
    select departmentId,
            name,
            salary,
            rank() over(
                partition by departmentId
                order by salary desc
            ) as rnk
    from Employee
) as drnk
left join department d on d.id=drnk.departmentId
where rnk=1;
```

也可以不用窗口函数

```mysql
SELECT
    Department.name AS 'Department',
    Employee.name AS 'Employee',
    Salary
FROM
    Employee
        JOIN
    Department ON Employee.DepartmentId = Department.Id
WHERE
    (Employee.DepartmentId , Salary) IN
    (   SELECT
            DepartmentId, MAX(Salary)
        FROM
            Employee
        GROUP BY DepartmentId
    )
;
```



### 题目三：返回部门工资排名前三的员工

表: `Employee`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| id           | int     |
| name         | varchar |
| salary       | int     |
| departmentId | int     |
+--------------+---------+
id 是该表的主键列(具有唯一值的列)。
departmentId 是 Department 表中 ID 的外键（reference 列）。
该表的每一行都表示员工的ID、姓名和工资。它还包含了他们部门的ID。
```

 表: `Department`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
id 是该表的主键列(具有唯一值的列)。
该表的每一行表示部门ID和部门名。
```

 公司的主管们感兴趣的是公司每个部门中谁赚的钱最多。一个部门的 **高收入者** 是指一个员工的工资在该部门的 **不同** 工资中 **排名前三** 。

```mysql
select
d.name as department,
e_rnk.name as employee,
e_rnk.salary
from(
    select
        name,
        salary,
        departmentId,
        dense_rank() over(
            partition by departmentId
            order by salary desc
        ) as rnk
    from employee
) as e_rnk
left join department d on d.id=e_rnk.departmentId
where e_rnk.rnk<=3;
```

[跳转到目录](#catalog)



## 二、if函数的运用

### if的用法解说

基本语法如下：

```sql
IF(expr, if_true_expr, if_false_expr)
```

- **`expr`**：这是一个表达式，用于判断条件。
- **`if_true_expr`**：如果 `expr` 的结果为 `TRUE`（非零值或非空值），则返回这个表达式的值。
- **`if_false_expr`**：如果 `expr` 的结果为 `FALSE`（零值或空值），则返回这个表达式的值。

例子：

1. 在 `SELECT` 语句中使用

**示例 1：根据性别返回不同的称呼**
假设有一个 `Users` 表，其中包含 `gender` 列（值为 `'M'` 或 `'F'`），你想根据性别返回不同的称呼。

```sql
SELECT 
    name,
    IF(gender = 'M', 'Mr.', 'Ms.') AS title
FROM 
    Users;
```

如果 `gender` 是 `'M'`，则返回 `'Mr.'`；否则返回 `'Ms.'`。

**示例 2：根据年龄返回不同的分组**
假设有一个 `Users` 表，其中包含 `age` 列，你想根据年龄分组。

```sql
SELECT 
    name,
    age,
    IF(age < 18, 'Minor', IF(age >= 18 AND age < 60, 'Adult', 'Senior')) AS age_group
FROM 
    Users;
```

这里使用了嵌套的 `IF` 函数：
- 如果 `age < 18`，返回 `'Minor'`。
- 如果 `age >= 18` 且 `age < 60`，返回 `'Adult'`。
- 否则返回 `'Senior'`。

2. 在 `UPDATE` 语句中使用

**示例：根据订单状态更新折扣**
假设有一个 `Orders` 表，其中包含 `status` 和 `discount` 列，你想根据订单状态更新折扣。

```sql
UPDATE Orders
SET discount = IF(status = 'completed', 10, 0);
```

如果订单状态是 `'completed'`，则将折扣设置为 `10`；否则设置为 `0`。

3. 在 `WHERE` 子句中使用

虽然 `IF` 函数通常不直接用于 `WHERE` 子句，但可以通过 `CASE` 语句或逻辑运算符实现类似的效果。

**示例：根据条件筛选数据**
假设你想根据性别筛选用户，但性别列可能包含空值。

```sql
SELECT 
    name,
    gender
FROM 
    Users
WHERE 
    IFNULL(gender, 'F') = 'F';
```

这里使用了 `IFNULL` 函数来处理空值，确保即使 `gender` 是 `NULL`，也不会影响查询结果。

---

### 题目1：

表：`Trips`

```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| id          | int      |
| client_id   | int      |
| driver_id   | int      |
| city_id     | int      |
| status      | enum     |
| request_at  | varchar  |     
+-------------+----------+
id 是这张表的主键（具有唯一值的列）。
这张表中存所有出租车的行程信息。每段行程有唯一 id ，其中 client_id 和 driver_id 是 Users 表中 users_id 的外键。
status 是一个表示行程状态的枚举类型，枚举成员为(‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’) 。
```

 

表：`Users`

```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| users_id    | int      |
| banned      | enum     |
| role        | enum     |
+-------------+----------+
users_id 是这张表的主键（具有唯一值的列）。
这张表中存所有用户，每个用户都有一个唯一的 users_id ，role 是一个表示用户身份的枚举类型，枚举成员为 (‘client’, ‘driver’, ‘partner’) 。
banned 是一个表示用户是否被禁止的枚举类型，枚举成员为 (‘Yes’, ‘No’) 。
```

 

**取消率** 的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)。

编写解决方案找出 `"2013-10-01"` 至 `"2013-10-03"` 期间有 **至少** 一次行程的非禁止用户（**乘客和司机都必须未被禁止**）的 **取消率**。非禁止用户即 banned 为 No 的用户，禁止用户即 banned 为 Yes 的用户。其中取消率 `Cancellation Rate` 需要四舍五入保留 **两位小数** 。

返回结果表中的数据 **无顺序要求** 。

- 我的解答

```mysql
select 
request_at as 'Day',
ROUND(
        SUM(
            IF(nb.status != 'completed', 1, 0)
        ) / COUNT(nb.id), 
        2
    ) AS 'Cancellation Rate'
from(
select t.*
from Trips t
left join Users u1 on u1.users_id = t.client_id
left join Users u2 on u2.users_id = t.driver_id
where u1.banned='No' and u2.banned='No'
    AND t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
        ) as nb
group by request_at;
```

- kimi的解法

可以发现我的解法和kimi差不多，但是kimi写得更优雅

```mysql
SELECT 
    request_at AS Day,
    ROUND(
        SUM(CASE 
            WHEN status IN ('cancelled_by_driver', 'cancelled_by_client') THEN 1
            ELSE 0
        END) / COUNT(*) * 100, 2) AS Cancellation_Rate
FROM 
    Trips
JOIN 
    Users AS Clients ON Trips.client_id = Clients.users_id
JOIN 
    Users AS Drivers ON Trips.driver_id = Drivers.users_id
WHERE 
    Clients.banned = 'No'
    AND Drivers.banned = 'No'
    AND request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY 
    request_at;
```

[跳转到目录](#catalog)



## 三、跨日期计算

### 题目1：用户两日留存比率

Table: `Activity`

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
（player_id，event_date）是此表的主键（具有唯一值的列的组合）。
这张表显示了某些游戏的玩家的活动情况。
每一行是一个玩家的记录，他在某一天使用某个设备注销之前登录并玩了很多游戏（可能是 0）。
```

 编写解决方案，报告在首次登录的第二天再次登录的玩家的 **比率**，**四舍五入到小数点后两位**。换句话说，你需要计算从首次登录日期开始至少连续两天登录的玩家的数量，然后除以玩家总数。

解决方案，找出每个部门中 **收入高的员工** 。

以 **任意顺序** 返回结果表。

- 我的解法

  ```mysql
  -- 计算出向前n天的日期
  select 
  player_id,
  min(event_date) as min_date,
  adddate(min(event_date),interval 1 day) as next_date
  from Activity
  group by player_id; -- as next_d
  
  
  -- 找出所有符合的player_id
  select
  round(
  	count(next_date)/count(distinct a1.player_id),
      2)
  as fraction
  from Activity a1
  left join (
  	select 
  		player_id,
  		adddate(min(event_date),interval 1 day) as next_date
  		from Activity
  		group by player_id) as next_d
  	on next_d.player_id=a1.player_id and next_d.next_date = a1.event_date
  ;
  ```

- 窗口函数——一次性可以解决n天的相似问题

  ```mysql
  select
  round(
  	sum(if(DATEDIFF(coalesce(sp.dp1,sp.event_date),sp.event_date)=1,1,0))
      / 
  	sum(if(sp.rn=1,1,0))
      ,
  	2)
  as fraction
  from (
  	select
  		a.player_id,
  		a.device_id,
  		a.event_date,
  		lead(a.event_date,1) over(
  			partition by a.player_id 
              order by a.event_date) as dp1, -- 向前错开一行.当前行的下一行的 event_date 值
  		row_number() over(
  			partition by a.player_id 
              order by a.event_date ) as rn -- 标记最小日期，用于计算distinct player_id
  		from Activity a
  		)sp
  where sp.rn=1; 
  ```

  1. FROM 子查询 (`sp`)
     - 使用 `Activity` 表生成临时表 `sp`，包含 `player_id`, `device_id`, `event_date`, `dp1`, 和 `rn`。
  2. WHERE 子句过滤
     - 从 `sp` 中筛选出 `rn = 1` 的记录。
  3. SELECT 子句计算
     - 计算分子（满足条件的玩家数量）。
     - 计算分母（总的玩家数量）。
     - 计算比值并四舍五入。
  4. 返回结果
     - 输出名为 `fraction` 的最终结果。

[跳转到目录](#catalog)



## 四、IN和with的使用

### 题目1：

`Insurance` 表：满足条件的投保人的投保金额之和

```
+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| pid         | int   |
| tiv_2015    | float |
| tiv_2016    | float |
| lat         | float |
| lon         | float |
+-------------+-------+
pid 是这张表的主键(具有唯一值的列)。
表中的每一行都包含一条保险信息，其中：
pid 是投保人的投保编号。
tiv_2015 是该投保人在 2015 年的总投保金额，tiv_2016 是该投保人在 2016 年的总投保金额。
lat 是投保人所在城市的纬度。题目数据确保 lat 不为空。
lon 是投保人所在城市的经度。题目数据确保 lon 不为空。
```

 编写解决方案报告 2016 年 (`tiv_2016`) 所有满足下述条件的投保人的投保金额之和：

- 他在 2015 年的投保额 (`tiv_2015`) 至少跟一个其他投保人在 2015 年的投保额相同。
- 他所在的城市必须与其他投保人都不同（也就是说 (`lat, lon`) 不能跟其他任何一个投保人完全相同）。

`tiv_2016` 四舍五入的 **两位小数** 。



- 我的解答

  ```mysql
  with ll as (
      select
          lat,
          lon,
          count(pid) as ll_count
      from Insurance i2
      group by lat, lon
  ),
  
  tiv as (
      select
          tiv_2015,
          count(pid) as tiv_count
      from Insurance i3
      group by tiv_2015
  )
  
  select
  round(sum(tiv_2016),2) as tiv_2016
  from Insurance i1,ll,tiv
  
  where ll.ll_count=1 and ll.lat=i1.lat and ll.lon=i1.lon
       and tiv.tiv_count>1 and i1.tiv_2015=tiv.tiv_2015
  ;
  ```

- 官方的解答

  ```mysql
  SELECT
      SUM(insurance.TIV_2016) AS TIV_2016
  FROM
      insurance
  WHERE
      insurance.TIV_2015 IN
      (
        SELECT
          TIV_2015
        FROM
          insurance
        GROUP BY TIV_2015
        HAVING COUNT(*) > 1
      )
      AND CONCAT(LAT, LON) IN
      (
        SELECT
          CONCAT(LAT, LON)
        FROM
          insurance
        GROUP BY LAT , LON
        HAVING COUNT(*) = 1
      )
  ;
  ```

  **性能对比**

  | 特性              | 官方方法                      | 我的方法                          |
  | ----------------- | ----------------------------- | --------------------------------- |
  | **查询逻辑**      | 简单直观                      | 分步处理，逻辑更清晰              |
  | **索引利用率**    | 较低（`CONCAT` 阻止索引使用） | 较高（可利用索引）                |
  | **CPU 开销**      | 较高（`CONCAT` 计算）         | 较低（避免动态计算）              |
  | **I/O 开销**      | 较高（多次扫描表）            | 较低（中间结果集复用）            |
  | **内存/磁盘占用** | 较低（无中间结果集存储）      | 较高（`WITH` 子句生成中间结果集） |
  | **扩展性**        | 较差（难以适应复杂逻辑）      | 较好（易于扩展和优化）            |

[跳转到目录](#catalog)



## 五、递归操作

### 递归 CTE 的基本结构

递归 CTE 由两部分组成：**初始查询**和**递归查询**。这两部分由 `UNION ALL` 连接。

```sql
WITH RECURSIVE cte_name AS (
    -- 初始查询
    SELECT ...
    FROM ...
    WHERE ...
    
    UNION ALL
    
    -- 递归查询
    SELECT ...
    FROM cte_name
    JOIN ...
    WHERE ...
)
SELECT * FROM cte_name;
```

#### 示例1：计算层级关系

假设你有一个员工表，其中包含员工的 ID 和其经理的 ID，你想要查询出每个员工的层级关系。

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT
);

INSERT INTO employees (id, name, manager_id) VALUES
(1, 'CEO', NULL),
(2, 'Alice', 1),
(3, 'Bob', 1),
(4, 'Charlie', 2),
(5, 'David', 2);
```

使用递归 CTE 来查询每个员工的层级关系：

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- 初始查询：从顶级（经理 ID 为 NULL 的员工）开始
    SELECT
        id,
        name,
        manager_id,
        1 AS level
    FROM
        employees
    WHERE
        manager_id IS NULL
    
    UNION ALL
    
    -- 递归查询：连接到自身以获取下一层级的员工
    SELECT
        e.id,
        e.name,
        e.manager_id,
        eh.level + 1 AS level
    FROM
        employees e
    INNER JOIN
        employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy;
```

#### 示例2：计算阶乘

递归 CTE 也可以用于数学计算，比如计算一个数的阶乘：

```sql
WITH RECURSIVE factorial AS (
    -- 初始查询：定义基例
    SELECT
        1 AS n,
        1 AS result
    
    UNION ALL
    
    -- 递归查询：计算下一个数的阶乘
    SELECT
        n + 1,
        result * (n + 1)
    FROM
        factorial
    WHERE
        n < 5  -- 计算到 5 的阶乘
)
SELECT * FROM factorial;
```

#### 注意事项

1. **终止条件**：递归查询必须有一个明确的终止条件，否则会导致无限递归。
2. **性能**：递归查询在处理大数据集时可能会比较慢，因此需要谨慎使用。
3. **最大递归深度**：MySQL 默认的递归深度是 1000，如果需要调整，可以设置 `cte_max_recursion_depth` 系统变量。

### 题目1： 计算员工层级



