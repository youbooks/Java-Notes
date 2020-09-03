# MySQL索引优化explain

原文：https://mp.weixin.qq.com/s/rem7Ds_QSnyhlrtNPByQcg

[TOC]

## 1. MySQL逻辑架构介绍

日常在CURD的过程中，都避免不了跟数据库打交道，大多数业务都离不开数据库表的设计和SQL的编写，那如何让你编写的SQL语句性能更优呢？

先来整体看下MySQL逻辑架构图：

<img src="https://image.ldbmcs.com/2020-03-05-070113.jpg" style="zoom:50%;" />

MySQL整体逻辑架构图可以分为Server和存储引擎层。

### 1.1 Server层

Server层涵盖了MySQL的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），以及存储过程、触发器、视图等跨存储引擎的实现也在这一层来实现。

- **连接器**：负责跟客户端建立连接、获取权限、维持和管理连接。
- **分析器**：SQL词法分析，SQL语法分析
- **优化器**：索引选择，选择一个执行效率高的，生成执行计划
- **执行器**：操作引擎，返回执行结果
- ...
- **查询缓存**：执行SQL语句之前，先查缓存，缓存结果可能是以key-value对方式存储的，key 是查询的语句，value 是查询的结果。

### 1.2 存储引擎层

负责数据的存储和提取，是一种插件式的架构方式。支持 InnoDB、MyISAM、Memory 等多个存储引擎。MySQL 5.5.5版本开始默认存储引擎是 InnoDB，也是目前常用的存储引擎。

今天我们来看下详细看下优化器里的执行计划如何分析，要分析一个 SQL 的执行效率，就要会看执行计划，根据执行计划优化 SQL，使其能达到高效查询的目的。

一条查询语句需要经过 MySQL 查询优化器的各种基于成本和规则，优化后会生成一个所谓的`执行计划`。

那么这个执行计划主要展示具体执行查询的方式，比如多表连接的顺序是多少，表里包含多个索引，每个表采用什么访问方法来具体执行查询等。

而设计 MySQL 的大佬是非常贴心的，知道开发的朋友们都是亲自写 SQL 的，但是写出 SQL 容易，想写出性能高的 SQL 可不简单。

所以，大佬提供了 `Explain` 语句来帮我们查询某个查询语句的具体执行计划。

## 2. SQL 执行计划解析

本文带大家看懂 `EXPLAIN` 语句，必须要熟悉各项输出是做什么的，从而有针对性的提升SQL 查询语句的性能。

| **`列名`**      | **`用途`**                    |
| ------------- | --------------------------- |
| id            | 每一个SELECT关键字查询语句都对应一个唯一id   |
| select_type   | SELECT关键字对应的查询类型            |
| table         | 表名                          |
| partitions    | 匹配的分区信息                     |
| type          | 单表的访问方法                     |
| possible_keys | 可能用到的索引                     |
| key           | 实际使用到的索引                    |
| key_len       | 实际使用到的索引长度                  |
| ref           | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息 |
| rows          | 预估需要读取的记录条数                 |
| filtered      | 某个表经过条件过滤后剩余的记录条数百分比        |
| Extra         | 额外的一些信息                     |

为了方便解释上面的执行计划各项输出的含义，下面创建三张数据库表。

### 2.1 数据库创建三张表

```sql
DROP TABLE IF EXISTS user;
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `name` varchar(45) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO user (`id`, `name`, `update_time`)
  VALUES (1,'a','2017-12-22 15:27:18'), (2,'b','2017-12-22 15:27:18'), (3,'c','2017-12-22 15:27:18');

DROP TABLE IF EXISTS `group`;
CREATE TABLE `group` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `group` (`id`, `name`) VALUES (1,'group1'),(2,'group2'),(3,'group3');

DROP TABLE IF EXISTS user_group;
CREATE TABLE `user_group` (
  `id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  `group_id` int(11) NOT NULL,
  `remark` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_group_id` (`group_id`),
  KEY `idx_user_group_id` (`user_id`,`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO user_group (`id`, `user_id`, `group_id`, `remark`)
  VALUES (1,1,1,'bak1'), (2,2,2,'bak2'), (3,3,3,'bak3');
```

### 2.2 EXPLAIN 输出计划参数详解

下载了最新的 MySQL8.0+ 版本，直接执行 `EXPLAIN` ，对比了 MySQL 5.0+ 版本执行的 `EXPLAIN EXTENDED` 命令同样都提供了一些查询优化的信息。除了执行计划各项输出参数外，额外还有 `filtered` 列，是一个百分比的值，`rows * filtered/100` 可以估算出将要和 `EXPLAIN` 中前一个表进行连接的行数 。

如下所示：

![](https://image.ldbmcs.com/2020-03-05-070818.jpg)

`EXPLAIN` 中的列 接下来我们将详细说明下 `EXPLAIN` 执行结果每一列的信息。

#### 2.2.1 id 列

设计表时通常会设计 id，一般会作为主键，执行计划的结果也不例外，也有 id 列，`id` 列编号是 `SELECT` 的序列号，并且 id 的顺序是按 `SELECT` 出现的顺序增长的。id列越大执行优先级越高，id 相同则从上往下执行，id 为 NULL 最后执行。

MySQL将 `SELECT` 查询分为简单查询 `SIMPLE` 和复杂查询 `PRIMARY`。

复杂查询包括：简单子查询、派生表（ `FROM` 语句中的子查询）、`UNION` 和 `UNION ALL` 查询。

**简单查询：**

![](https://image.ldbmcs.com/2020-03-05-071047.jpg)

**复杂查询：**

1）简单子查询

EXPLAIN SELECT (SELECT 1 from user LIMIT 1) from `user`;

![](https://image.ldbmcs.com/2020-03-05-071125.jpg)

2）`FROM` 子句中的子查询

EXPLAIN SELECT *FROM (SELECT id, count(*) as c from `group` GROUP BY name) as derived；

![](https://image.ldbmcs.com/2020-03-05-071152.jpg)

这个查询执行时有个临时表别名为 `derived`，外部 `SELECT` 查询引用了这个临时表

3）`UNION` 和 `UNION ALL` 查询

EXPLAIN SELECT *FROM user UNION SELECT* FROM user;

![](https://image.ldbmcs.com/2020-03-05-071222.jpg)

`UNION` 结果总是放在一个匿名临时表中，临时表不在 SQL 中出现，临时表名为 ``，因此它的 `id` 是 `NULL`，表明这个临时表是为了合并两个查询结果集而创建的。

跟 `UNION` 对比，`UNION ALL` 无需为最终结果而去重，仅是单纯的将多个查询结果集中的记录合并成一个并返回给用户，所以不会使用到临时表，故没有 `id` 为 `NULL` 记录。如下所示：

EXPLAIN SELECT *FROM user UNION ALL SELECT* FROM user;

![](https://image.ldbmcs.com/2020-03-05-071334.jpg)

注意点：**子查询优化为连接查询**

`查询优化器可能对子查询进行重写，进而转换为连接查询`，查询计划中的两个id值是相同的，如下所示：

EXPLAIN SELECT * FROM user WHERE id IN (SELECT user_id FROM `user_group`);

![](https://image.ldbmcs.com/2020-03-05-071403.jpg)

#### 2.2.2 select_type 列

**MySQL中优化器中的概念：**

`物化`:

子查询语句中的子查询结果集中的记录保存到临时表的过程称之为 `物化`（英文名：`Materialize`），简单理解为存储子查询结果集的临时表称之为 `物化表`。

也正因为物化表的记录都建立了索引（基于内存的物化表有哈希索引，基于磁盘的有B+树索引），因此通过 `IN` 语句判断某个操作数在不在子查询的结果集中变得很快，从而提升语句的性能。

`半连接 semi-join`：

也是跟 `IN` 语句子查询有关。

通用语句：

```sql
SELECT ... FROM outer_tables
    WHERE expr IN (SELECT ... FROM inner_tables ...) AND ...
```

`outer_tables` 表对 `inner_tables` 半连接的意思：

`对于`outer_tables`的某条记录来说，我们仅关心在`inner_tables`表中是否存在匹配的记录，而不用关心具体有多少条记录与之匹配，最终结果只保留 outer_tables 表的记录`。

每一个 `SELECT` 关键字的查询都定义了一个 `select_type` 属性，知道这个查询属性就能知道在整个查询语句中所扮演的角色。

1）`SIMPLE`：简单查询。查询不包含子查询 和 `UNION`。

2）`PRIMARY`：复杂查询中最外层的`SELECT`，可参照上面的 `UNION` 查询语句。

3）`SUBQUERY`：包含的子查询语句无法转换为 `semi-join`，并且为不相关子查询，查询优化器采用物化方案执行该子查询，该子查询的第一个 `SELECT` 就会 `SUBQUERY`。该查询由于被物化，`只需要执行一次`。

4）`DERIVED`：对于采用物化形式执行的包含派生表的查询，该派生表的对应的子查询为 `DERIVED`。

查询语句如下所示：

```sql
EXPLAIN SELECT * FROM (SELECT id, count(*) as c FROM user GROUP BY id) AS derived_u where c>1;
```

![](https://image.ldbmcs.com/2020-03-05-071651.jpg)

5）`UNION`：在 `UNION` 查询语句中的第二个和紧随其后的 `SELECT`。

6）`UNION RESULT`：MySQL选择使用临时表完成 `UNION` 查询的去重工作。

当 `select_type` 为这个值时，经常可以看到table的值是 ``，这说明匹配的 id 行 是这个集合的一部分。请看上面 `UNION` 查询示例。

7）`MATERIALIZED`：当查询优化器执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询类型为 `MATERIALIZED`。

8）`DEPENDENT SUBQUERY`：包含的子查询语句无法转换为 `semi-join`，并且为相关子查询，则该子查询的第一个 `SELECT` 就会 `DEPENDENT SUBQUERY`。该查询`可能会被执行多次`。

8）`DEPENDENT UNION`：包含的子查询语句中包含了 `UNION` 或者 `UNION ALL` 的大查询，这些查询都依赖外层查询，这些子查询语句类型为 `DEPENDENT UNION`。

```sql
EXPLAIN SELECT * FROM user WHERE id IN (SELECT user_id FROM user_group WHERE name = 'a' UNION SELECT id FROM user WHERE name = 'b');
```

![](https://image.ldbmcs.com/2020-03-05-071725.jpg)

上面这个子查询语句中的 `SELECT user_id FROM user_group WHERE name = 'a'` 这个小查询是第一个子查询，所以它的 `select_type` 为 `DEPENDENT SUBQUERY`，而 `SELECT id FROM user WHERE name = 'b'` 这个查询在 `UNION` 后面，所以它的 `select_type` 为 `DEPENDENT UNION`。

最常见的值包括：`SIMPLE` 、`PRIMARY`、`DERIVED`、`UNION`。

#### 2.2.3 table 列

`table` 列表示 `EXPLAIN` 的单独行的唯一标识符。这个值可能是表名、表的别名或者一个未查询产生临时表的标识符，如派生表、子查询或集合。

当 `FROM` 子句中有子查询时，如果优化器采用的物化方式，table 列是 `` 格式，表示当前查询依赖 `id=N` 的查询，于是先执行 `id=N` 的查询。

当使用 `UNION` 查询时，`UNION RESULT` 的 table 列的值为 ``，1和2表示参与 `UNION` 的 SELECT 的行 id。

#### 2.2.4 type 列

这一列表示关联类型或访问类型，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。依次从最优到最差分别为：**system > const > eq_ref > ref > range > index > ALL** 一般来说，得保证查询达到range级别，最好达到ref NULL：mysql能够在优化阶段分解查询语句，在执行阶段用不着再访问表或索引。例如：在索引列中选取最小值，可以单独查找索引来完成，不需要在执行时访问表 mysql> explain select min(id) from user;

1）`system,const`：MySQL 能对查询的某部分进行优化并将其转化成一个常量。用于主键或唯一二级索引列与常数比较时，所以表最多有一个匹配行，`读取1次，速度比较快`。`system`是 `const` 的特例，表里只有一条记录匹配时为 `system`。

EXPLAIN SELECT *FROM (SELECT* FROM user where id = 1) tmp;

2）`eq_ref`：在**连接查询**时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行访问的，则对该被驱动表的访问方法就是 `eq_ref`。这可能是在 const 之外最好的联接类型了。

EXPLAIN SELECT * FROM user_group INNER JOIN user ON user_group.user_id = user.id;

![](https://image.ldbmcs.com/2020-03-05-071854.jpg)

3）`ref`：相比 eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。

a. 简单 `SELECT` 查询，name 是普通索引（非唯一索引)。

EXPLAIN SELECT * FROM user where user.name = 'a';

![](https://image.ldbmcs.com/2020-03-05-071911.jpg)

b. `关联表`查询，`idx_user_group_id (user_id,group_id)` 为联合索引，这里使用到了user_group联合索引最左边前缀 user_id。

EXPLAIN SELECT user_id FROM user LEFT JOIN user_group ON user.id = user_group.user_id;

![](https://image.ldbmcs.com/2020-03-05-071948.jpg)

4）`ref_or_null`：对普通二级索引进行等值查询，该索引列也可以为NULL值时。

EXPLAIN SELECT * FROM user where user.name = 'a' OR name IS NULL;

![](https://image.ldbmcs.com/2020-03-05-072008.jpg)

5）`index_merge`：MySQL使用索引合并的方式执行的。

EXPLAIN SELECT * FROM user WHERE user.name = 'a' OR user.id = 1;

![](https://image.ldbmcs.com/2020-03-05-072026.jpg)

6）`range`：使用索引获取`范围区间`的记录，通常出现在 `in, between ,> ,<, >=` 等操作中。

EXPLAIN SELECT * FROM user WHERE user.id > 1;

![](https://image.ldbmcs.com/2020-03-05-072049.jpg)

7）`index`：扫描全表索引，这通常比ALL快一些。（`index`是从索引中读取的，而 `ALL` 是从硬盘中读取）

`group` 表里的两个字段都有索引。

EXPLAIN SELECT * FROM `group`;

![](https://image.ldbmcs.com/2020-03-05-072109.jpg)

8）`ALL`：即全表扫描，MySQL 需要从头到尾去查找表中所需要的行。通常情况下这需要增加索引来进行优化了。

EXPLAIN SELECT * FROM `user`;

![](https://image.ldbmcs.com/2020-03-05-072125.jpg)

#### 2.2.5 possible_keys 列

`possible_keys` 列表示查询可能使用哪些索引来查找。

`EXPLAIN` 执行计划结果可能出现 `possible_keys` 列，而 `key` 显示 `NULL` 的情况，这种情况是因为表中数据不多，MySQL 会认为索引对此查询帮助不大，选择了全表查询。

如果 `possible_keys` 列为 `NULL`，则没有相关的索引。在这种情况下，可以通过检查 `WHERE` 子句去分析下，看看是否可以创造一个适当的索引来提高查询性能，然后用 `EXPLAIN` 查看效果。

另外**注意**：不是这一列的值越多越好，使用索引过多，查询优化器计算时查询成本高，所以如果可能的话，尽量删除那些不用的索引。

#### 2.2.6 key 列

`key` 列表示实际采用哪个索引来优化对该表的访问。

如果没有使用索引，则该列是 NULL。如果想强制 MySQL使用或忽视 `possible_keys` 列中的索引，在查询中使用 `force index`、`ignore index`。

#### 2.2.7 key_len 列

`key_len` 列表示当查询优化器决定使用某一个索引查询时，该索引记录的最大长度。

**`key_len` 列计算规则如下：**

- **字符串**

char(n)：n字节长度

varchar(n)：2字节存储字符串长度，如果是utf-8，则长度 3n + 2

**注意：**该索引列可以存储`NULL`值，则`key_len`比不可以存储`NULL`值时多1个字节。

比如：varchar(50)，则实际占用的`key_len`长度是 3 * 50 + 2 = 152，如果该列允许存储`NULL`，则`key_len`长度是153。

- **数值类型**

tinyint：1字节 smallint：2字节 int：4字节 bigint：8字节　

- **时间类型**

date：3字节 timestamp：4字节 datetime：8字节

索引最大长度是768字节，当字符串过长时，MySQL 会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引。

**举例1：**

`user_group`表中的联合索引 `idx_user_group_id` 由 `user_id` 和 `group_id` 两个int 列组成，并且每个 int 是 4 字节。

EXPLAIN SELECT * FROM user_group WHERE user_id = 2;

![](https://image.ldbmcs.com/2020-03-05-072308.jpg)

通过结果中的 key_len=4可推断出查询使用了第一个列：`user_id` 列来执行索引查找。

**举例2：**

再看 `user` 表 name 字段是 varchar(45) 变长字符串类型，`key_len`为138 等于 `45 * 3 + 2 (变长字节) + 1字节（允许存储NULL值）`

EXPLAIN SELECT * FROM user WHERE name = 'a';

![](https://image.ldbmcs.com/2020-03-05-072323.jpg)

所以，以后再看到 `key_len` 字段的值，不要在懵逼咯，固定套路~

#### 2.2.8 ref 列

`ref` 列显示了在 `key` 列记录的索引中，表查找值所用到的列或常量，常见的有：`const`（常量），`字段名`（例：`user.id`）。

#### 2.2.9 rows 列

`rows` 列是查询优化器估计要读取并检测的行数，注意这个不是结果集里的行数。

如果查询优化器使用全表扫描查询，`rows` 列代表预计的需要扫码的行数；如果查询优化器使用索引执行查询，`rows` 列代表预计扫描的索引记录行数。

#### 2.2.10 filtered 列

对于单表来说意义不大，主要用于连接查询中。

前文中也已提到 `filtered` 列，是一个百分比的值，对于连接查询来说，主要看`驱动表`的 `filtered`列的值 ，通过 `rows * filtered/100` 计算可以估算出`被驱动表`还需要执行的查询次数。

EXPLAIN SELECT * FROM user INNER JOIN user_group ON user.id = user_group.user_id WHERE user.update_time = '2019-01-01';

![](https://image.ldbmcs.com/2020-03-05-072448.jpg)

可以看到驱动表`user`执行的rows列为3行，filtered列为 33.33，计算驱动表的`扇出值`为 3 * 33.33% 约等于1，说明还需要对被驱动表执行大约1次查询。

#### 2.2.11 Extra 列

`Extra` 列提供了一些额外信息。这一列在 MySQL中提供的信息有几十个，这里仅列举一些常见的重要值如下：

1）`Using index`：查询的列被索引覆盖，并且 `WHERE` 筛选条件是索引的前导列，使用了索引性能高。一般是使用了覆盖索引(查询列都是索引列字段)。对于 INNODB 存储引擎来说，如果是辅助索引性能会有不少提高，并且也不需要回表查询。

2）`Using where Using index`：查询的列被索引覆盖，并且 `WHERE` 筛选条件是索引列之一，但并不是索引的前导列，意味着无法直接通过索引查找来查询到符合条件的数据。

3）`NULL`：查询的列未被索引覆盖，并且 `WHERE` 筛选条件是索引的前导列，意味着用到了索引，但是部分字段未被索引覆盖，必须通过 `回表` 来查询，不是纯粹地用到了索引，也不是完全没用到索引。

4）`Using index condition`：与`Using where`类似，查询的列不完全被索引覆盖，`WHERE` 条件中是一个前导列的范围。

5）`Using temporary`：MySQL 中需要创建一张内部临时表来处理查询，一般出现这种情况就需要考虑进行优化了，首先是想到用索引来优化。

通常在许多执行包括DISTINCT、GROUP BY、ORDER BY等子句查询过程中，如果不能有效利用索引来完成查询，MySQL很有可能会寻求建立内部临时表来执行查询。

所以，执行计划中出现了 `Using temporary` 并不是个好兆头，因为建立与维护临时表要付出很大的成本的，要考虑使用`索引`来优化改进。

6）`Using filesort`：MySQL 会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。此时 MySQL 会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。这种情况下一般也是要考虑使用索引来优化的。

查询中需要使用 `filesort` 的方式进行排序的记录非常多，那么这个过长是很耗时的，想办法将使用 `文件排序` 的执行方式改进为使用`索引`进行排序。

7）`Index merges`：通常显示为`Using sort_union(...)` 说明准备用 `Sort-Union` 索引合并方式来查询；显示为 `Using union(...)`，说明准备用`Union`索引合并方式查询；显示为`Using intersect(...)`，说明准备使用`Intersect`索引合并方式查询。

8）`LooseScan`：在 IN 子查询转为 `semi-join` 时，如果采用的是 `LooseScan` 执行策略，则会在`Extra`中提示。

9）`FirstMatch(tbl_name)`：在 IN 子查询转为 `semi-join` 时，如果采用的是 `FirstMatch` 执行策略，则会在`Extra`中提示。

10）`Using join buffer`：强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。出现该值，应该注意，根据查询的具体情况可能需要添加索引来改进性能。

我们所提到的`回表`操作 ，其实是一种随机IO，比较耗时，所以尽量避免上面提到的回表操作，当发现`Extra`提示为 `Using filesort`、`Using temporary` 时就需要格外注意了，考虑索引优化。

## 3. 最佳姿势索引实践

### 3.1 新建 staff 表表演使用

```sql
# 重建 `staff` 表
DROP TABLE `staff`;
CREATE TABLE `staff` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
  `s_name` VARCHAR(24) NOT NULL DEFAULT '' COMMENT  '花名',
  `s_no` INT(4) NOT NULL DEFAULT 0 COMMENT  '工号',
  `work_age` int(11) NOT NULL DEFAULT '0' COMMENT '工龄',
  `position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
  `arrival_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  `remark` VARCHAR(500) DEFAULT NULL COMMENT '备注', # 允许 NULL
  PRIMARY KEY (`id`), # 主键
  UNIQUE KEY idx_s_name (s_name), # 唯一索引
  KEY idx_s_no (s_no), # 普通索引
  KEY `idx_name_age_position` (`name`,`work_age`,`position`) USING BTREE # 联合索引
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='员工记录表';
# 初始化 `staff` 表数据
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('zhangsan','zs',10,2,'manager',NOW());
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('lisi','ls',11,3,'dev',NOW());
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('wangwu','ww',12,8,'dev',NOW());
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('zhangliu','zl',110,5,'dev',NOW());
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('xiaosun','xs',111,5,'dev',NOW());
INSERT INTO staff(name,s_name,s_no,work_age,position,arrival_time) VALUES('donggua','dg',200,3,'dev',NOW());
```

### 3.2 数据库索引最佳实践

#### 3.2.1 全值匹配

EXPLAIN SELECT * FROM staff WHERE name= 'zhangsan';

![](https://image.ldbmcs.com/2020-03-05-072905.jpg)

EXPLAIN SELECT * FROM staff WHERE name= 'zhangsan' AND work_age = 2;

![](https://image.ldbmcs.com/2020-03-05-072923.jpg)

EXPLAIN SELECT * FROM staff where name = 'zhangsan' AND work_age = 2 AND position = 'dev';

![](https://image.ldbmcs.com/2020-03-05-072949.jpg)

EXPLAIN SELECT * FROM staff where position = 'dev' AND name = 'zhangsan' AND work_age = 2;

![](https://image.ldbmcs.com/2020-03-05-073004.jpg)

最后一条，我们将 `position` 放到了 `WHERE` 条件后面，尽管没有按照联合索引的顺序编写条件，MySQL 优化器会自动优化，将 name 排到最前面去，所以还是会正确使用联合索引的。

联合索引创建后，你必须严格按照最左前缀的原理进行使用，否则会无法使用到索引。尽量按照这个顺序去写，这样避免 MySQL 优化器再次优化了。

#### 3.2.2 最佳左前缀法则

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。

以下 SQL 符合最左前缀匹配法则：

EXPLAIN SELECT * FROM staff WHERE name = 'zhangsan' AND work_age = 3 AND position = 'manager';

![](https://image.ldbmcs.com/2020-03-05-073043.jpg)

EXPLAIN SELECT * FROM staff WHERE name = 'zhangsan' AND position = 'manager';

![](https://image.ldbmcs.com/2020-03-05-073106.jpg)

以下执行都是全表扫描，`type` 为 `ALL`，都不符合最左前缀法则：

EXPLAIN SELECT * FROM staff WHERE work_age = 2 AND position ='dev';

![](https://image.ldbmcs.com/2020-03-05-073122.jpg)

EXPLAIN SELECT * FROM staff WHERE position = 'dev';

![](https://image.ldbmcs.com/2020-03-05-073134.jpg)

#### 3.2.3 索引列上避免做计算操作

索引上尽量避免做函数计算等操作，会导致索引失效而转向全表扫描。

`WHERE` 条件后面索引列使用函数:

EXPLAIN SELECT *FROM staff WHERE LEFT(name, 5) = 'zhang'; EXPLAIN SELECT* FROM staff WHERE LOWER(name) = 'zhangsan';

![](https://image.ldbmcs.com/2020-03-05-073204.jpg)

EXPLAIN SELECT *FROM staff WHERE staff.s_no* 2 > 3;

![](https://image.ldbmcs.com/2020-03-05-073220.jpg)

查询的结果 type 列为 `ALL`，key 是空的，索引失效，全表扫描。

计算逻辑尽量放到业务层去处理，最大限度的命中索引，同时还能节省数据库资源开销。

#### 3.2.4 范围条件右边的列无法使用索引

EXPLAIN SELECT * FROM staff WHERE name= 'zhangsan' AND work_age > 2 AND position ='dev';

![](https://image.ldbmcs.com/2020-03-05-073300.jpg)

我们看到了执行结果中 type 为 `range` 级别，使用了范围查找，而 position 字段并没有用到索引（没有使用到BTree的索引去查询），只是从 `name = 'zhangsan' AND work_age > 2` 条件返回的结果集中，再过滤符合 position 字段条件的数据。

#### 3.2.5 尽量使用覆盖索引

覆盖索引：简单理解，只访问建了索引的列。减少使用 `SELECT *` 语句查询列。

使用了覆盖索引：

EXPLAIN SELECT name,work_age FROM staff WHERE name= 'zhangsan' AND work_age = 3;

![](https://image.ldbmcs.com/2020-03-05-073338.jpg)

使用了 `SELECT *` 查询：

EXPLAIN SELECT * FROM staff WHERE name= 'zhangsan' AND work_age = 3;

![](https://image.ldbmcs.com/2020-03-05-073359.jpg)

我们重点看下使用了 `覆盖索引` 方式查询，会在结果中 `Extra` 列显示 `Using index` ，这说明在查询列包含了索引列，不需要再次回表查询了。而如果使用 `SELECT *` 方式查询，查询列包含非索引的列，`Extra` 显示为 `NULL`，所以还会进行回表查询。

附一个曾经线上SQL的优化记录：

![](https://image.ldbmcs.com/2020-03-05-073414.jpg)

artist 表有几十万条的数据量，第一条执行的SQL没有索引直接查询，查询耗时 `0.557` 毫秒；`第一次优化`新建 founded 字段作为普通索引，查询耗时 `0.0224` 毫秒；`第二次优化`再次重建联合索引 founded_name，优化后查询耗时：`0.0051` 毫秒。因为使用了覆盖索引查询方式，基于此优化，SQL查询效率提升非常明显。

#### 3.2.6 范围条件查找能够命中索引

范围条件主要包括 `<、<=、>、>=、between` 等。

若条件中范围列有普通索引和主键索引同时存在， 优先使用主键索引:

EXPLAIN SELECT * FROM staff WHERE staff.s_no > 10 AND staff.id > 2;

![](https://image.ldbmcs.com/2020-03-05-073511.jpg)

范围列可以用到索引，注意联合索引必须符合最左前缀法则，如果查询条件中有两个范围列则无法全用到索引，优化器会去选择：

EXPLAIN SELECT * FROM staff WHERE staff.name != 'zl' AND staff.s_no > 1;

![](https://image.ldbmcs.com/2020-03-05-073533.jpg)

若条件中范围查询和等值查询同时存在，优先匹配等值查询列的索引：

EXPLAIN SELECT * FROM staff WHERE staff.s_no > 10 AND staff.s_name = 'zl';

![](https://image.ldbmcs.com/2020-03-05-073553.jpg)

#### 3.2.7 IS NOT NULL 无法使用索引

索引列建议都使用 `NOT NULL 约束` 及默认值，单列索引不存 NULL 值，联合索引不存全部为 NULL 的值，如果列允许为 NULL，查询结果可能不符合预期。

staff 表中为 `remark` 字段新建普通索引：

```sql
ALTER TABLE staff ADD INDEX idx_remark (remark);
```

`IS NULL` 查询命中索引：

EXPLAIN SELECT * FROM staff WHERE staff.remark IS NULL;

![](https://image.ldbmcs.com/2020-03-05-073711.jpg)

`IS NOT NULL` 查询不会命中索引：

EXPLAIN SELECT * FROM staff WHERE staff.name IS NOT NULL;

![](https://image.ldbmcs.com/2020-03-05-073727.jpg)

#### 3.2.8 模糊条件查询以通配符开头索引失效

`like '%xx'` 或 `like '%xx%'` 前导模糊查询不能命中索引：

EXPLAIN SELECT * from staff where name like '%zhang%';

![](https://image.ldbmcs.com/2020-03-05-073801.jpg)

**如何使用模拟查询才能命中索引？**

**a）`like 'xx%'` 非前导模糊查询可以命中索引：**

EXPLAIN SELECT * FROM staff WHERE name LIKE 'zhang%';

![](https://image.ldbmcs.com/2020-03-05-073823.jpg)

**b）使用覆盖索引，查询字段必须要建立覆盖索引字段**

EXPLAIN SELECT name,work_age FROM staff WHERE name LIKE '%zhang%';

联合索引是 `idx_name_work_age_position`

![](https://image.ldbmcs.com/2020-03-05-073845.jpg)

#### 3.2.9 字符串类型不加单引号索引失效

字符串的数据类型一定要将常量值使用单引号，这个在日常开发中要特别注意的，数据类型出现隐式转换的时候不会命中索引。

**不加单引号索引失效**

EXPLAIN SELECT * FROM staff WHERE name = 1;

![](https://image.ldbmcs.com/2020-03-05-073916.jpg)

name=1 类似于在该字段上做了一个函数运算，因此不会走索引的。

**加单引号会命中索引：**

EXPLAIN SELECT * FROM staff WHERE name = 'zhangsan';

![](https://image.ldbmcs.com/2020-03-05-073932.jpg)

#### 3.2.10 OR使用多数情况下索引会失效

EXPLAIN SELECT * FROM staff WHERE name='zhangsan' OR work_age = 2;

![](https://image.ldbmcs.com/2020-03-05-074005.jpg)

尽管 name 和 work_age 是联合索引，但是 work_age 列上并没有建索引，所以使用了 `OR` 不会走索引。

如果 `OR` 前后都是联合索引带头大哥 name 字段，那么就会用到索引，如下所示：

![](https://image.ldbmcs.com/2020-03-05-074021.jpg)

因 `OR` 后面的条件列中没有索引，会走全表扫描。存在全表扫描的情况下，就没有必要多一次索引扫描增加IO访问。

**可使用覆盖索引查询：**

EXPLAIN SELECT name,work_age FROM staff WHERE name='zhangsan' OR work_age = 2;

![](https://image.ldbmcs.com/2020-03-05-074041.jpg)

**OR 后面也使用索引列：**

EXPLAIN SELECT * FROM staff WHERE name='zhangsan' OR s_name='wangwu';

![](https://image.ldbmcs.com/2020-03-05-074105.jpg)

s_name 是唯一索引，name是联合索引第一个字段，两者使用 `OR` 查询结果 `Extra` 显示 `Using sort_union(idx_name_age_position,idx_s_name); Using where` 解释一下。

如果执行计划 `Extra` 列出现了 `Using sort_union(...)` 的提示，说明准备使用 `Sort-Union` 索引合并的方式执行查询。如果出现了 `Using intersect(...)` 的提示，说明准备使用 `Intersect` 索引合并方式执行查询，如果出现了 `Using union(...)` 的提示 ，说明准备使用 `Union` 索引合并方式执行查询。括号中 `...` 表示需要进行索引合并的索引名称。

**使用UNION优化改进：**

EXPLAIN SELECT *FROM staff WHERE name='zhangsan' UNION SELECT* FROM staff WHERE s_name = 'zs';

![](https://image.ldbmcs.com/2020-03-05-074122.jpg)

使用 `UNION` 执行计划中出现了第三条记录，`Extra` 中出现 `Using temporary`，说明 MySQL因为不能有效利用索引，建立了内部临时表来执行查询。当你在使用 `DISTINCT 、GROUP BY、UNION` 等子句中的查询过程中，都有可能会出现该扩展信息。

**使用UNION ALL进一步优化：**

EXPLAIN SELECT *FROM staff WHERE name='zhangsan' UNION ALL SELECT* FROM staff WHERE s_name = 'zs';

![](https://image.ldbmcs.com/2020-03-05-074136.jpg)

执行结果中不再出现内部临时表，具体用的时候结合实际需求来定是否使用。

#### 3.2.11 负向查询条件不能使用索引

负向查询条件包括：`!=、<>、NOT IN、NOT EXISTS、NOT LIKE` 等。

**不会命中索引：**

EXPLAIN SELECT *FROM staff WHERE s_no !=1 AND s_no != 2; EXPLAIN SELECT* FROM staff WHERE s_no NOT IN (1,2);

![](https://image.ldbmcs.com/2020-03-05-074222.jpg)

**使用IN优化，命中索引：**

EXPLAIN SELECT * FROM staff WHERE s_no IN (11,12);

![](https://image.ldbmcs.com/2020-03-05-074238.jpg)

但是使用 `IN` 命中索引有个前提，是查询条件字段数据区分度要高，通常如：状态、类型、性别之类的字段。

#### 3.2.12 排序对索引的影响

`ORDER BY`是经常用的语句，排序也遵循最左前缀列的原则。

查询所有列未命中索引：

EXPLAIN SELECT * FROM staff ORDER BY name,work_age;

![](https://image.ldbmcs.com/2020-03-05-074325.jpg)

覆盖索引查询可命中索引：

![](https://image.ldbmcs.com/2020-03-05-074338.jpg)

覆盖索引能够利用联合索引查询，但是 `ORDER BY` 后的条件查询不符合最左前缀原则，执行结果 `Extra` 中出现了 `Using filesort` 的提示，一般看到这个就要想办法优化了。

调整排序的两个字段顺序之后，`Extra` 会提示为 `Using index`，使用了索引，避免了排序的资源开销：

EXPLAIN SELECT name,work_age FROM staff ORDER BY name,work_age;

![](https://image.ldbmcs.com/2020-03-05-074352.jpg)

#### 3.2.13 局部索引的使用

局部索引，区别于最左列索引（顺序取索引中靠左的列的查询），它只取某列的一部分作为索引。

INNODB存储引擎下，一般是字符串类型，很长，全部作为索引大大增加存储空间，索引也需要维护，对于长字符串，又想作为索引列，可取的办法就是取前一部分（局部），代表一整列作为索引串。

如何确保这个前缀能代表或大致代表这一列？MySQL中有个概念是 `索引选择性`，是指索引中不重复的值的数目（也称基数X）与整个表该列记录总数（T）的比值。基数可以通过`SHOW INDEX FROM 表名` 查看。

比如一个列表 [1,2,2,3,5,6]，总数是 6，不重复值数目为 5，选择性为 5/6，因此选择性范围是[X/T, 1]，这个值越大，表示列中不重复值越多，越适合作为局部索引，而唯一索引（UNIQUE KEY）的选择性是1。

`SELECT COUNT(DISTINCT(CONCAT(LEFT(remark, N))/COUNT(*) FROM t; 测试出接近 1 的索引选择性，其中N是索引的长度，穷举法去找出N的值，然后再建索引。

**创建 `局部索引` ，使用 remark 字段举个例子**

EXPLAIN SELECT * FROM staff where remark LIKE 'xxx%';

![](https://image.ldbmcs.com/2020-03-05-074421.jpg)

**对 remark 字段重建局部索引：**

```sql
 ALTER TABLE staff DROP INDEX idx_remark_part, ADD INDEX idx_remark_part(remark(5));
```

再次执行查询：

EXPLAIN SELECT * FROM staff where remark LIKE 'xxx%';

![](https://image.ldbmcs.com/2020-03-05-074458.jpg)

## 4. 索引优化总结

上面列了大部分场景索引最佳实战，除此之外，不宜建索引的几点小总结：

1）更新非常频繁字段不宜建索引

因为字段更新台频繁，会导致B+树的频繁的变更，重建索引。所以这个过程是十分消耗数据库性能的。

2）区分度不大的字段不宜建索引

比如类似性别这类的字段，区分度不大，建立索引的意义不大。因为不能有效过滤数据，性能和全表扫描相当。另外注意一点，返回数据的比例在 `30%` 之外的，优化器不会选择使用索引。

3）业务中有唯一特性的字段，建议建成`唯一索引`

业务中如果有唯一特性的字段，即使是多个字段的组合，也尽量都建成唯一索引。尽管唯一索引会影响插入效率，但是对于查询的速度提升是非常明显的。此外，还能够提供校验机制，如果没有唯一索引，高并发场景下，可能还会产生脏数据。

4）多表关联时，要确保关联字段上必须有索引

5）创建索引时避免建立错误的认识

> 索引越多越好，认为一个查询就需要建一个索引。
> 
> 宁缺勿滥，认为索引会消耗空间、严重拖慢更新和新增速度。
> 
> 抵制唯一索引，认为业务的唯一性一律需要在应用层通过“先查后插”方式解决。
> 
> 过早优化，在不了解系统的情况下就开始优化。

6）最佳索引实践口诀

如果你觉得上面哪些太啰嗦，有朋友已总结为一套优化口诀，优化SQL时也能提个醒吧。

> 全值匹配我最爱，最左前缀要遵守；
> 
> 带头大哥不能死，中间兄弟不能断；
> 
> 索引列上少计算，范围之后全失效；
> 
> Like百分写最右，覆盖索引不写星；
> 
> 不等空值还有or，索引失效要少用；
> 
> VAR引号不可丢，SQL高级也不难！

7）`EXPLAIN` 执行计划实践总结

如果还是觉得 `EXPLAIN` 执行计划列太多了，也记不住呀，那么请重点关注以下几列：

`第1列`：ID越大，执行的优先级越高；ID相等，从上往下优先顺序执行。

`第2列`：select_type 查询语句的类型，SIMPLE简单查询，PRIMARY复杂查询，DERIVED衍生查询(from子查询的临时表)，派生表。

`第4列`：请重点掌握，type类型，查询效率优先级：system->const->eq_ref->ref->range->index->ALL

`ALL` 是`最差`的，`system` 是`最好`的，性能最佳，阿里巴巴开发规约中要求最差也得到 `range` 级别，而不能有 `index、ALL`。

最后，对于后端工程师而言，尽力都能掌握 `EXPLAIN` 的使用，写完SQL请习惯性的用它帮助你分析一下，做一个对SQL性能有追求的程序员，因为SQL也是程序员必备技能，将慢查询问题拍死在项目上线前夕。