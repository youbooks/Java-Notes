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







