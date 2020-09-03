> 转载：[MySQL innodb中各种SQL语句加锁分析](https://www.fordba.com/locks-set-by-different-sql-statements-in-innodb.html)

## 1. 概要

Locking read（ `SELECT ... FOR UPDATE` or `SELECT ... LOCK IN SHARE MODE`），`UPDATE`以及`DELETE`语句通常会在他扫描的索引所有范围上加锁，忽略没有用到索引的那部分where语句。
举个例子：

```sql
CREATE TABLE `test` (
  `id` int(11) NOT NULL DEFAULT '0',
  `name` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

select * from test where id > 3 and name <'A' for update;
```

这条SQL语句的会将所有`id>3`的记录进行加锁，而不是`id>3 and name <'A' `进行加锁，因为name上面没有索引。

如果一个SQL通过二级索引进行扫描，并且在二级索引上设置了一个锁，那么innodb将会在对应的聚簇索引记录上也加上一把锁。

如果一个SQL语句无法通过索引进行`Locking read`，`UPDATE`，`DELETE`，那么MySQL将扫描整个表，表中的每一行都将被锁定（在RC级别，通过`semi-consistent read`，能够提前释放不符合条件的记录，在RR级别，需要设置`innodb_locks_unsafe_for_binlog`为1，才能打开`semi-consistent read`）。在某些场景下，锁也不会立即被释放。例如一个`union`查询，生成 了一张临时表，导致临时表的行记录和原始表的行记录丢失了联系，只能等待查询执行结束才能释放。

