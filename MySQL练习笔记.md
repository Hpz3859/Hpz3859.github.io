

# MySQL练习笔记1

<a id="catalog">目录</a>

[TOC]



## 一、合并两表

编写解决方案，报告 `Person` 表中每个人的姓、名、城市和州。如果 `personId` 的地址不在 `Address` 表中，则报告为 `null` 。

```mysql
-- 如果不存在，创建表
create table if not exists Person(
	PersonId int primary key,
    FirstName varchar(50),
    LastName varchar(50)
);

create table if not exists Address(
	AddressId int primary key,
    PersonId int,
    City varchar(50),
    State varchar(50),
    foreign key (PersonId) references Person(PersonId)
);

-- 根据PersonId合并两表
select
    p.FirstName,
    p.LastName,
    a.City, -- 如无，直接返回Null
    a.State
from Person p
left join
	Address a on p.PersonId = a.PersonId;
```

:exclamation:乍一看没什么问题，但是在leetcode上提交时会报<font style=background:red>“时间超出限额”</font>，也就是说这个合并不高效，主要原因是<font style=background:red>缺乏索引</font>。

- **没有索引时：**`Person`表的`PersonId`是主键自带索引，但是`Address`的`PersonId`是一个外键，如果没有显示创建索引，用<font style=background:LightCoral>LEFT JOIN</font>时，需要对`Address`表的`PersonId`进行查找，通过全表扫描`Address`来匹配`Person`表中的`PersonId`，时间复杂度为<font style=background:LightCoral>O(N*M)</font>
- **有索引时**：MySQL 可以通过索引直接定位到 `Address` 表中匹配的 `PersonId`，时间复杂度接近 <font style=background:LightCoral>O(N)</font>，效率大幅提升

```mysql
-- 添加索引（如果尚未创建）
CREATE INDEX idx_personid ON Address(PersonId);

-- 再执行查询
SELECT
    p.FirstName,
    p.LastName,
    a.City,
    a.State
FROM Person p
LEFT JOIN Address a ON p.PersonId = a.PersonId;
```

[跳转到目录](#catalog)



## 二、表的自连接

表：`Employee` 

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| salary      | int     |
| managerId   | int     |
+-------------+---------+
id 是该表的主键（具有唯一值的列）。
该表的每一行都表示雇员的ID、姓名、工资和经理的ID。
```

编写解决方案，找出收入比经理高的员工。

```mysql
-- 创建表
CREATE TABLE Employee (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary INT,
    managerId INT
);
-- 插入数据（包含员工和经理）
INSERT INTO Employee (id, name, salary, managerId) VALUES
-- 顶层经理（无经理）
(1, 'Alice', 120000, NULL),
(2, 'Bob', 130000, NULL),
-- 第一层员工
(3, 'Charlie', 80000, 1),
(4, 'David', 85000, 1),
(5, 'Eva', 90000, 2),
(6, 'Frank', 95000, 2),
-- 第二层员工
(7, 'Grace', 70000, 3),
(8, 'Henry', 75000, 3),
(9, 'Ivy', 78000, 4),
(10, 'Jack', 82000, 5),
-- 工资比经理高的员工（验证用例）
(11, 'Joe', 75000, 3),    -- 经理 Charlie 工资 80000（Joe 工资更低，不应出现在结果）
(12, 'Kate', 82000, 3),   -- 经理 Charlie 工资 80000（Kate 工资更高，应出现在结果）
(13, 'Leo', 90000, 5),    -- 经理 Eva 工资 90000（Leo 工资相同，不应出现）
(14, 'Mia', 95000, 6),    -- 经理 Frank 工资 95000（Mia 工资相同，不应出现）
-- 其他随机数据
(15, 'Noah', 60000, 7),
(16, 'Olivia', 65000, 8),
(17, 'Peter', 72000, 9),
(18, 'Quinn', 68000, 10),
(19, 'Rachel', 83000, 5),
(20, 'Sam', 88000, 6);

select * from Employee;
-- 查询并返回工资比经理高的员工
-- 将表自连接
select
e.id,
e.name,
e.salary,
m.salary as managersalary
from Employee e
left join Employee m on m.id = e.managerId;

-- 返回满足条件的名字
select
e.name
from Employee e
left join Employee m on m.id=e.managerId
where e.salary > m.salary; 
```

- :exclamation:在SQL中，`NULL`值在比较操作中被视为“未知”或“不确定”，因此任何与`NULL`的比较都会返回`FALSE`。具体到该问题中，如果`managerSalary`（即`m.salary`）为`NULL`，条件`e.salary > m.salary`会返回`FALSE`，因此这些记录不会被`WHERE`子句过滤后的结果集包含

[跳转到目录](#catalog)



## 三、SQL语句的执行顺序

1. **FROM**：首先确定数据来源的表。
2. **JOIN**：执行表连接操作（如`INNER JOIN`、`LEFT JOIN`等），将多个表连接成一个虚拟的中间结果集。
3. **WHERE**：对中间结果集进行过滤，筛选出符合条件的记录。
4. **GROUP BY**：对`WHERE`子句过滤后的结果集进行分组。
5. **HAVING**：对分组后的结果进行过滤。
6. **SELECT**：从中间结果集中选择需要的列。
7. **ORDER BY**：对最终结果进行排序。
8. **LIMIT/OFFSET**：限制返回的记录数量或跳过某些记录。

[跳转到目录](#catalog)



## 四、SQL的几种join的区别

- `INNER JOIN`取两表交集

- `LEFT JOIN`返回左表（`table1`）的所有记录，以及右表（`table2`）中匹配的记录。

  ```mysql
  SELECT *
  FROM table1
  LEFT JOIN table2
  ON table1.column = table2.column;
  
  - 示例
  SELECT e.name, d.departmentName
  FROM Employee e
  LEFT JOIN Department d
  ON e.departmentId = d.id;
  ```

- `RIGHT JOIN`返回右表（`table2`）的所有记录，以及左表（`table1`）中匹配的记录。如果左表中没有匹配的记录，则结果集中左表的列会显示为`NULL`。

  ```sql
  SELECT *
  FROM table1
  RIGHT JOIN table2
  ON table1.column = table2.column;
  
  - 示例
  SELECT e.name, d.departmentName
  FROM Employee e
  RIGHT JOIN Department d
  ON e.departmentId = d.id;
  ```

- `FULL JOIN`——`UNION`返回两表并集

  **注意**：MySQL原生并不支持`FULL JOIN`，但可以通过`UNION`操作模拟：
  
  即左连接UNION右连接
  
  ```sql
  SELECT *
  FROM table1
  LEFT JOIN table2
  ON table1.column = table2.column
  UNION
  SELECT *
  FROM table1
  RIGHT JOIN table2
  ON table1.column = table2.column;
  ```
  
- `CROSS JOIN`用于返回两个表的笛卡尔积，即左表的每一行与右表的每一行进行组合。

  结果集的行数是左表行数与右表行数的乘积。

  ```sql
  SELECT *
  FROM table1
  CROSS JOIN table2;
  ```

  **示例**：
  假设`table1`有3行，`table2`有2行，`CROSS JOIN`的结果将有6行。

  ```sql
  SELECT e.name, d.departmentName
  FROM Employee e
  CROSS JOIN Department d;
  ```

[跳转到目录](#catalog)



## 五、查询

`Customers` 表：

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
在 SQL 中，id 是该表的主键。
该表的每一行都表示客户的 ID 和名称。
```

`Orders` 表：

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| customerId  | int  |
+-------------+------+
在 SQL 中，id 是该表的主键。
customerId 是 Customers 表中 ID 的外键( Pandas 中的连接键)。
该表的每一行都表示订单的 ID 和订购该订单的客户的 ID。
```

找出所有从不点任何东西的顾客。

```mysql
-- 创建 Customers 表
CREATE TABLE Customers (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

-- 创建 Orders 表
CREATE TABLE Orders (
    id INT PRIMARY KEY,
    customerId INT,
    FOREIGN KEY (customerId) REFERENCES Customers(id)
);

-- 插入 Customers 数据
INSERT INTO Customers (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie'),
(4, 'David');

-- 插入 Orders 数据
INSERT INTO Orders (id, customerId) VALUES
(101, 1),
(102, 3),
(103, 3);
```

- 方法一，表连接（时间和空间复杂率较高）

  ```mysql
  select c.name
  from Customers c
  left join Orders o on o.customerId=c.id
  where o.customerId is null;
  ```

- 方法二，从零时表中比对查询

  ```mysql
  select c.name
  from Customers c
  where c.id not in (
      select customerId from Orders);
  ```

[跳转到目录](#catalog)



## 六、日期处理

表： `Weather`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| recordDate    | date    |
| temperature   | int     |
+---------------+---------+
id 是该表具有唯一值的列。
没有具有相同 recordDate 的不同行。
该表包含特定日期的温度信息
```

 编写解决方案，找出与之前（昨天的）日期相比温度更高的所有日期的 `id` 。

```mysql
CREATE TABLE Weather (
    id INT PRIMARY KEY,
    recordDate DATE,
    temperature INT
);

INSERT INTO Weather (id, recordDate, temperature) VALUES
(1, '2023-01-01', 10),
(2, '2023-01-02', 15),
(3, '2023-01-03', 12),
(4, '2023-01-04', 20),
(5, '2023-01-05', 25),
(6, '2023-01-06', 26);
select * from Weather;

-- 日期自减
select w1.id
from Weather w1
left join Weather w2 on date_sub(w2.recordDate, INTERVAL -1 DAY) = w1.recordDate
where w1.temperature > w2.temperature;
```

[跳转到目录](#catalog)



## 七、日期计算专题

1. **`DATE_ADD()`和`DATE_SUB()`函数**

```sql
DATE_ADD(date, INTERVAL expr type);
DATE_SUB(date, INTERVAL expr type);
```

- `date`：要操作的日期或时间值。
- `INTERVAL`：关键字，表示后面的表达式是一个时间间隔。
- `expr`：时间间隔的值（如`1`、`-2`等）。
- `type`：时间间隔的类型，可以是`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`、`SECOND`等。

2. **使用`ADDDATE()`和`SUBDATE()`函数**

`ADDDATE()`和`SUBDATE()`是`DATE_ADD()`和`DATE_SUB()`的别名，功能完全相同。

3. **计算两个日期之间的差值**

如果想计算两个日期之间的差值，可以使用`TIMESTAMPDIFF()`函数。

**语法**

```sql
TIMESTAMPDIFF(unit, start_date, end_date);
```

- `unit`：时间单位（如`YEAR`、`MONTH`、`DAY`、`HOUR`等）。
- `start_date`和`end_date`：两个日期或时间值。

**示例**

1. **计算两个日期之间的天数差**
   
   ```sql
   SELECT TIMESTAMPDIFF(DAY, '2025-01-01', '2025-03-18');
   ```
   结果：`76`
   
2. **计算两个日期之间的月数差**
   
   ```sql
   SELECT TIMESTAMPDIFF(MONTH, '2024-01-01', '2025-03-18');
   ```
   结果：`14`

[跳转到目录](#catalog)



## 八、子查询（嵌套查询）、分组查询

活动表 `Activity`：

```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
在 SQL 中，表的主键是 (player_id, event_date)。
这张表展示了一些游戏玩家在游戏平台上的行为活动。
每行数据记录了一名玩家在退出平台之前，当天使用同一台设备登录平台后打开的游戏的数目（可能是 0 个）。
```

查询每位玩家 **第一次登录平台的日期**。

```mysql
-- 查询每位玩家 第一次登录平台的日期。
CREATE TABLE Activity (
    player_id INT,
    device_id INT,
    event_date DATE,
    games_played INT,
    PRIMARY KEY (player_id, event_date)
);
INSERT INTO Activity (player_id, device_id, event_date, games_played) VALUES
(1, 2, '2023-01-01', 5),
(1, 2, '2023-01-02', 6),
(1, 3, '2023-01-03', 0),
(2, 4, '2023-01-05', 7),
(2, 4, '2023-01-06', 1),
(3, 5, '2023-01-07', 3),
(3, 5, '2023-01-08', 2);


-- 分组子查询
select * from Activity;
select a1.player_id, a1.device_id, a1.event_date, games_played
from Activity a1
left join
	(select player_id, min(event_date) as first_login
    from Activity a2 
    group by a2.player_id) as a3
    on a3.player_id = a1.player_id
where a3.first_login = a1.event_date;

```

[跳转到目录](#catalog)



## 九、窗口函数

上一题还可以用窗口函数来做

```mysql
SELECT player_id, device_id, event_date, games_played
FROM (
    SELECT 
        player_id, 
        device_id, 
        event_date,
        games_played,
        RANK() OVER (
            PARTITION BY player_id
            ORDER BY event_date
        ) AS rkn	--窗口函数
    FROM Activity
) AS subquery
WHERE rkn = 1;
```

- :exclamation:错误的做法：

  ```mysql
  select 
  	player_id, 
  	device_id, 
      event_date,
      games_played,
      rank() over(
  		partition by player_id
          order by event_date) as rkn
  from Activity a
  where rkn=1;
  ```

  `WHERE` 子句无法直接引用由窗口函数（如 `RANK()`）生成的别名列（如 `rkn`）。这是因为窗口函数的计算是在 `SELECT` 阶段完成的，而 `WHERE` 子句在逻辑上是在 `SELECT` 阶段之前执行的。因此，`WHERE` 子句无法“看到”由窗口函数生成的别名列。

  :exclamation:<font color=red>**窗口函数（如`RANK()`）的结果是在`SELECT`阶段计算的**</font>

  :star:为了使用窗口函数的结果进行过滤，需要将窗口函数的结果封装到一个子查询中，然后在外层查询中引用它。这样可以确保窗口函数的结果在`WHERE`子句中可用

- `PARTITION BY` 

  在SQL中，`PARTITION BY` 是窗口函数（Window Functions）的一部分，用于将数据划分为多个分区（或组）。每个分区内的数据可以独立进行计算，而不会影响其他分区的数据。`PARTITION BY` 的作用类似于 `GROUP BY`，但它不会像 `GROUP BY` 那样对数据进行聚合，而是保留了每个分区内的所有行

  ```mysql
  SELECT column1, column2, ...,
         window_function() OVER (
             PARTITION BY column_name
             ORDER BY column_name
         ) AS alias
  FROM table_name;
  ```

  - **`PARTITION BY column_name`**：按指定列对数据进行分区。
  - **`ORDER BY column_name`**：在每个分区内对数据进行排序。
  - **`window_function()`**：在每个分区内应用的窗口函数，如 `RANK()`、`ROW_NUMBER()`、`SUM()` 等。

[跳转到目录](#catalog)



## 十、NULL的处理

在MySQL中，`NULL`是一个特殊的值，表示“未知”或“无值”。`NULL`与任何值（包括它自己）的比较都会返回`FALSE`，因为`NULL`不能与其他值进行直接比较。这就是为什么在你的查询中，`c1.referee_id`为`NULL`的记录不会被返回的原因。

1. **比较操作**：
   
   - 任何与`NULL`的比较（如`=`、`!=`、`<>`、`>`、`<`等）都会返回`FALSE`。
   - 例如：
     ```sql
     NULL = 2  -- 返回 FALSE
     NULL != 2 -- 返回 FALSE
     NULL <> 2 -- 返回 FALSE
     ```
   
2. **检查`NULL`**：
   
   - 要检查一个值是否为`NULL`，必须使用`IS NULL`或`IS NOT NULL`。
   - 例如：
     ```sql
     c1.referee_id IS NULL  -- 检查 c1.referee_id 是否为 NULL
     c1.referee_id IS NOT NULL  -- 检查 c1.referee_id 是否不为 NULL
     ```

[跳转到目录](#catalog)



## 十一、排序

表: `Orders`

```
+-----------------+----------+
| Column Name     | Type     |
+-----------------+----------+
| order_number    | int      |
| customer_number | int      |
+-----------------+----------+
在 SQL 中，Order_number是该表的主键。
此表包含关于订单ID和客户ID的信息。
```

 查找下了 **最多订单** 的客户的 `customer_number` 。

测试用例生成后， **恰好有一个客户** 比任何其他客户下了更多的订单。

**进阶：** 如果有多位顾客订单数并列最多，你能找到他们所有的 `customer_number` 吗？

- 如果只返回一个

  我自己写的，感觉有点笨重：

  ```mysql
  select c.customer_number
  from(
      select
          o.customer_number,
          count(o.order_number) as con
      from Orders o
      group by o.customer_number
  ) as c
  order by c.con desc
  limit 1;
  ```

  - 默认情况下，如果没有指定排序方式，MySQL会按升序排列。使用`ORDER BY column_name DESC`可以按指定列的值进行倒序排列

  - 如果需要按多个列排序，可以在`ORDER BY`子句中指定多个列，并为每个列指定排序方式（`ASC`或`DESC`）

    ```mysql
    -- 例如
    SELECT *
    FROM Customer
    ORDER BY con DESC, name ASC;
    ```

  :star:官方答案

  ```mysql
  select
  o.customer_number
  from Orders o
  group by o.customer_number
  order by count(*) desc
  limit 1;
  ```

  - 更简洁一些
