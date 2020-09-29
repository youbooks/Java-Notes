## 1. 【简单】[组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

### 1.1 题目描述

表1: Person：

```sql
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
```

`PersonId` 是上表主键。
表2: Address：

```sql
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
```

`AddressId` 是上表主键。

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

```sql
FirstName, LastName, City, State
```

### 1.2 题解

```sql
SELECT
	p.FirstName,
	p.LastName,
	a.City,
	a.State
FROM
	Person p
	LEFT JOIN Address a ON a.PersonId = p.PersonId;
```

## 2. 【简单】[第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

### 2.1 题目描述

编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

```sql
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

```sql
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

### 2.2 题解

```sql
SELECT
	( SELECT DISTINCT
			salary
		FROM
			Employee
		ORDER BY
			salary DESC
		LIMIT 1,
		1) AS SecondHighestSalary;
```

## 3. 【中等】[分数排名](https://leetcode-cn.com/problems/rank-scores/)

### 3.1 题目描述

编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

```sql
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```

例如，根据上述给定的 `Scores` 表，你的查询应该返回（按分数从高到低排列）：

```sql
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

**重要提示：**对于 MySQL 解决方案，如果要转义用作列名的保留字，可以在关键字之前和之后使用撇号。例如 **`Rank`**

### 3.2 题解

```sql
SELECT
	a.Score,
	(
		SELECT
			count(DISTINCT b.Score)
		FROM
			Scores b
		WHERE
			b.Score >= a.Score) AS 'Rank'
	FROM
		Scores a
	ORDER BY
		a.Score DESC;
```

## 4. 【简单】[超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

### 4.1 题目描述

`Employee` 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

```sql
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
```

给定 `Employee` 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

```sql
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

### 4.2 题解

```sql
SELECT
	e.Name AS Employee
FROM
	employee e
WHERE
	salary > (
		SELECT
			salary
		FROM
			employee
		WHERE
			Id = e.ManagerId);
```

## 5. 【简单】[查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

### 5.1 题目描述

编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

**示例：**

```sql
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

根据以上输入，你的查询应返回以下结果：

```sql
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

**说明：**所有电子邮箱都是小写字母。

### 5.2 题解

```sql
-- 解法1
select email from person group by email having count(email)>1

-- 解法2
select email from (select count(1) as t,email from person group by email)r  where r.t>1;

-- 解法3
select distinct(p1.Email) from Person p1  
join Person  p2 on p1.Email = p2.Email AND p1.Id!=p2.Id
```

## 6. 【简单】[从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

### 6.1 题目描述

某网站包含两个表，`Customers` 表和 `Orders` 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

`Customers` 表：

```sql
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```

`Orders` 表：

```sql
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

例如给定上述表格，你的查询应返回：

```sql
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

### 6.2 题解

```sql
select c.Name as Customers from Customers c where c.Id not in (select distinct o.CustomerId from Orders o);
```

## 7. 【中等】[部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

### 7.1 题目描述

`Employee` 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

```sql
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
```

`Department` 表包含公司所有部门的信息。

```sql
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）。

```sql
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```

**解释：**

Max 和 Jim 在 IT 部门的工资都是最高的，Henry 在销售部的工资最高。

### 7.2 题解

```sql
SELECT
	d.Name AS Department,
	e.Name AS Employee,
	e.Salary
FROM
	Employee e
	INNER JOIN Department d ON d.Id = e.DepartmentId
WHERE (e.DepartmentId, e.Salary)
in(
	SELECT
		DepartmentId, max(Salary)
		FROM Employee
	GROUP BY
		DepartmentId)
```

## 8. 【困难】[部门工资前三高的所有员工](https://leetcode-cn.com/problems/department-top-three-salaries/)

### 8.1 题目描述

`Employee` 表包含所有员工信息，每个员工有其对应的工号 `Id`，姓名 `Name`，工资 `Salary` 和部门编号 `DepartmentId` 。

```sql
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 85000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
| 5  | Janet | 69000  | 1            |
| 6  | Randy | 85000  | 1            |
| 7  | Will  | 70000  | 1            |
+----+-------+--------+--------------+
```

`Department` 表包含公司所有部门的信息。

```sql
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

编写一个 SQL 查询，找出每个部门获得前三高工资的所有员工。例如，根据上述给定的表，查询结果应返回：

```sql
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
```

解释：

IT 部门中，Max 获得了最高的工资，Randy 和 Joe 都拿到了第二高的工资，Will 的工资排第三。销售部门（Sales）只有两名员工，Henry 的工资最高，Sam 的工资排第二。

### 8.2 题解

```sql
SELECT
	d.Name AS Department,
	e1.Name AS Employee,
	e1.Salary
FROM
	Employee e1
	INNER JOIN Department d ON e1.DepartmentId = d.Id
WHERE (select count(DISTINCT Salary)
FROM
	Employee e2
WHERE
	e2.Salary >= e1.Salary
	AND e1.DepartmentId = e2.DepartmentId) <= 3
```

## 9. 【简单】[删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

### 9.1 题目描述

编写一个 SQL 查询，来删除 `Person` 表中所有重复的电子邮箱，重复的邮箱里只保留 **Id** *最小* 的那个。

```sql
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id 是这个表的主键。
```

例如，在运行你的查询语句之后，上面的 `Person` 表应返回以下几行:

```sql
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

**提示：**

- 执行 SQL 之后，输出是整个 `Person` 表。
- 使用 `delete` 语句。

### 9.2 题解

```sql
delete from Person where id not in (select id from (select min(id) as id from Person group by Email) t)
```

## 10. 【简单】[上升的温度](https://leetcode-cn.com/problems/rising-temperature/)

### 10.1 题目描述

给定一个 `Weather` 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。

```sql
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```

例如，根据上述给定的 `Weather` 表格，返回如下 Id:

```sql
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

### 10.2 题解

```sql
SELECT
	w1.Id
FROM
	Weather w1
WHERE
	w1.Temperature > (
		SELECT
			w2.Temperature
		FROM
			Weather w2
		WHERE
			DATEDIFF(w1.RecordDate, w2.RecordDate) = 1);
```

## 11. 【简单】[大的国家](https://leetcode-cn.com/problems/big-countries/)

### 11.1 题目描述

这里有张 `World` 表：

```sql
+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
```

如果一个国家的面积超过300万平方公里，或者人口超过2500万，那么这个国家就是大国家。

编写一个SQL查询，输出表中所有大国家的名称、人口和面积。

例如，根据上表，我们应该输出:

```sql
+--------------+-------------+--------------+
| name         | population  | area         |
+--------------+-------------+--------------+
| Afghanistan  | 25500100    | 652230       |
| Algeria      | 37100000    | 2381741      |
+--------------+-------------+--------------+
```

### 11.2 题解

```sql
SELECT
	name,
	population,
	area
FROM
	World
WHERE
	area >= 3000000
	OR population >= 25000000
```

## 12. 【简单】[超过5名学生的课](https://leetcode-cn.com/problems/classes-more-than-5-students/)

### 12.1 题目描述

有一个`courses` 表 ，有: **student (学生)** 和 **class (课程)**。

请列出所有超过或等于5名学生的课。

例如，表：

```sql
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```

应该输出：

```sql
+---------+
| class   |
+---------+
| Math    |
+---------+
```

**提示：**

- 学生在每个课中不应被重复计算。

### 12.2 题解

```sql
SELECT
	class
FROM
	courses
GROUP BY
	class
HAVING
	count(DISTINCT student) >= 5;
```

## 13. 【简单】[有趣的电影](https://leetcode-cn.com/problems/not-boring-movies/)

### 13.1 题目描述

某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

例如，下表 `cinema`：

```sql
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
```

对于上面的例子，则正确的输出是为：

```sql
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

### 13.2 题解

```sql
SELECT
	*
FROM
	cinema
WHERE
	description != 'boring'
	AND id % 2 = 1
ORDER BY
	rating DESC;
```

## 14. 【简单】[交换工资](https://leetcode-cn.com/problems/swap-salary/)

### 14.1 题目描述

给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。

注意，您必只能写一个 Update 语句，请不要编写任何 Select 语句。

例如：

```sql
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
```

运行你所编写的更新语句之后，将会得到以下表：

```sql
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

### 14.2 题解

```sql
UPDATE
	salary
SET
	sex = if(sex = 'm', 'f', 'm')
```

## 15. 【简单】[重新格式化部门表](https://leetcode-cn.com/problems/reformat-department-table/)

### 15.1 题目描述

部门表 `Department`：

```sql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| revenue       | int     |
| month         | varchar |
+---------------+---------+
(id, month) 是表的联合主键。
这个表格有关于每个部门每月收入的信息。
月份（month）可以取下列值 ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]。
```

编写一个 SQL 查询来重新格式化表，使得新的表中有一个部门 id 列和一些对应 **每个月** 的收入（revenue）列。

查询结果格式如下面的示例所示：

```sql
Department 表：
+------+---------+-------+
| id   | revenue | month |
+------+---------+-------+
| 1    | 8000    | Jan   |
| 2    | 9000    | Jan   |
| 3    | 10000   | Feb   |
| 1    | 7000    | Feb   |
| 1    | 6000    | Mar   |
+------+---------+-------+

查询得到的结果表：
+------+-------------+-------------+-------------+-----+-------------+
| id   | Jan_Revenue | Feb_Revenue | Mar_Revenue | ... | Dec_Revenue |
+------+-------------+-------------+-------------+-----+-------------+
| 1    | 8000        | 7000        | 6000        | ... | null        |
| 2    | 9000        | null        | null        | ... | null        |
| 3    | null        | 10000       | null        | ... | null        |
+------+-------------+-------------+-------------+-----+-------------+

注意，结果表有 13 列 (1个部门 id 列 + 12个月份的收入列)。
```

### 15.2 题解

```sql
select id,
    sum(case month when 'Jan' then revenue else null end) as Jan_Revenue,
    sum(case month when 'Feb' then revenue else null end) as Feb_Revenue,
    sum(case month when 'Mar' then revenue else null end) as Mar_Revenue,
    sum(case month when 'Apr' then revenue else null end) as Apr_Revenue,
    sum(case month when 'May' then revenue else null end) as May_Revenue,
    sum(case month when 'Jun' then revenue else null end) as Jun_Revenue,
    sum(case month when 'Jul' then revenue else null end) as Jul_Revenue,
    sum(case month when 'Aug' then revenue else null end) as Aug_Revenue,
    sum(case month when 'Sep' then revenue else null end) as Sep_Revenue,
    sum(case month when 'Oct' then revenue else null end) as Oct_Revenue,
    sum(case month when 'Nov' then revenue else null end) as Nov_Revenue,
    sum(case month when 'Dec' then revenue else null end) as Dec_Revenue
from Department
group by id
```

