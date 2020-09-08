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

## 2. SQL分析

1. `SELECT ... FROM` 是一个快照读，通过读取数据库的一个快照，不会加任何锁，除非将隔离级别设置成了 `SERIALIZABLE` 。在` SERIALIZABLE `隔离级别下，如果索引是非唯一索引，那么将在相应的记录上加上一个共享的`next key`锁。如果是唯一索引，只需要在相应记录上加`index record lock`。

2. `SELECT ... FROM ... LOCK IN SHARE MODE` 语句在所有索引扫描范围的索引记录上加上共享的`next key`锁。如果是唯一索引，只需要在相应记录上加`index record lock`。

3. `SELECT ... FROM ... FOR UPDATE` 语句在所有索引扫描范围的索引记录上加上排他的next key锁。如果是唯一索引，只需要在相应记录上加`index record lock`。这将堵塞其他会话利用`SELECT ... FROM ... LOCK IN SHARE MODE` 读取相同的记录，但是快照读将忽略记录上的锁。

4. `UPDATE ... WHERE ...`语句在所有索引扫描范围的索引记录上加上排他的`next key`锁。如果是唯一索引，只需要在相应记录上加`index record lock`。

   当`UPDATE` 操作修改主键记录的时候，将在相应的二级索引上加上隐式的锁。当进行重复键检测的时候，将会在插入新的二级索引记录之前，在其二级索引上加上一把共享锁。

5. `DELETE FROM ... WHERE ... `语句在所有索引扫描范围的索引记录上加上排他的`next key`锁。如果是唯一索引，只需要在相应记录上加`index record lock`。

6. `INSERT` 语句将在插入的记录上加一把排他锁，这个锁是一个`index-record lock`，并不是`next-key` 锁，因此就没有`gap` 锁，他将不会阻止其他会话在该条记录之前的`gap`插入记录。

   在插入记录之前，将会加上一种叫做 `insert intention gap` 的 `gap` 锁。这个 `insert intention gap`表示他有意向在这个`index gap`插入记录，如果其他会话在这个`index gap`中插入的位置不相同，那么将不需要等待。假设存在索引记录4和7，会话A要插入记录5，会话B要插入记录6，每个会话在插入记录之前都需要锁定4和7之间`gap`，但是他们彼此不会互相堵塞，因为插入的位置不相同。

   如果出现了重复键错误，将在重复键上加一个共享锁。如果会话1插入一条记录，没有提交，他会在该记录上加上排他锁，会话2和会话3都尝试插入该重复记录，那么他们都会被堵塞，会话2和会话3将尝试在该记录上加一个共享锁。如果此时会话1回滚，将发生死锁。

   例子如下：

   ```mysql
   表结构：
   CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;
   
   Session 1:
   START TRANSACTION;
   INSERT INTO t1 VALUES(1);
   
   Session 2:
   START TRANSACTION;
   INSERT INTO t1 VALUES(1);
   
   Session 3:
   START TRANSACTION;
   INSERT INTO t1 VALUES(1);
   
   Session 1:
   ROLLBACK;
   ```

   为什么会发生死锁呢？当会话1进行回滚的时候，记录上的排他锁释放了，会话2和会话3都获得了共享锁。然后会话2和会话3都想要获得排他锁，进而发生了死锁。

   还有一个类似的例子：

   ```mysql
   Session 1:
   START TRANSACTION;
   DELETE FROM t1 WHERE i = 1;
   
   Session 2:
   START TRANSACTION;
   INSERT INTO t1 VALUES(1);
   
   Session 3:
   START TRANSACTION;
   INSERT INTO t1 VALUES(1);
   
   Session 1:
   COMMIT;
   ```

   会话1在该记录上拥有一把排他锁，会话2和会话3都碰到了重复记录，因此都在申请共享锁。当会话1提交之后，会话1释放了排他锁，之后的会话2会话3先后获得了共享锁，此时他们发生了死锁，因为会话2和会话3都无法或者排他锁，因为彼此都占用了该记录的共享锁。

7. `INSERT ... ON DUPLICATE KEY UPDATE` 和普通的`INSERT`并不相同。如果碰到重复键值，`INSERT ... ON DUPLICATE KEY UPDATE` 将在记录上加排他的` next-key`锁。

8. `REPLACE `在没有碰到重复键值的时候和普通的`INSERT`是一样的，如果碰到重复键，将在记录上加一个排他的` next-key`锁。

9. `INSERT INTO T SELECT ... FROM S WHERE ...` 语句在插入T表的每条记录上加上` index record lock `。如果隔离级别是 `READ COMMITTED`, 或者启用了 `innodb_locks_unsafe_for_binlog` 且事务隔离级别不是`SERIALIZABLE`，那么innodb将通过快照读取表S(no locks)。否则，innodb将在S的记录上加共享的`next-key`锁。

   `CREATE TABLE ... SELECT ... `和 `INSERT INTO T SELECT ... FROM S WHERE ...` 一样，在S上加共享的`next-key`锁或者进行快照读取（(no locks）。

10. `REPLACE INTO t SELECT ... FROM s WHERE ... `和 `UPDATE t ... WHERE col IN (SELECT ... FROM s ...)` 中的`select `部分将在表s上加共享的`next-key`锁。

11. 当碰到有自增列的表的时候，innodb在自增列的索引最后面加上一个排他锁，叫`AUTO-INC table lock` 。`AUTO-INC table lock`会在语句执行完成后进行释放，而不是事务结束。如果`AUTO-INC table lock`被一个会话占有，那么其他会话将无法在该表中插入数据。innodb可以预先获取sql需要多少自增的大小，而不需要去申请锁，更多设置请参考参数`innodb_autoinc_lock_mode`。

12. 如果一张表的外键约束被启用了，任何在该表上的插入、更新、删除都将需要加共享的` record-level locks`来检查是否满足约束。如果约束检查失败，innodb也会加上共享的` record-level locks`。

13. `lock tables` 是用来加表级锁，但是是MySQL的server层来加这把锁的。当`innodb_table_locks = 1 (the default)` 以及` autocommit = 0`的时候，innodb能够感知表锁，同时server层了解到innodb已经加了`row-level locks`。否则，innodb将无法自动检测到死锁，同时server无法确定是否有行级锁，导致当其他会话占用行级锁的时候还能获得表锁。

## 3. 锁测试注意点

1. 当使用`begin`开启一个事务的时候，之后的查询并不是获取的`begin` 命令的时间点快照，而且`begin`命令之后第一个查询的时间点快照。

   ```mysql
   CREATE TABLE T2 (id INT,name varchar(10) ,PRIMARY KEY (id)) ENGINE = InnoDB;
   INSERT INTO T2 VALUES(1,'zhangsan'),(2,'lisi');
   
   
   SESS1:                      SESS2：
   BEGIN;
                               BEGIN;
                               INSERT INTO T2 values(3,'wangwu');
                               COMMIT;
   SELECT * FROM T2;
   +----+----------+
   | id | name     |
   +----+----------+
   |  1 | zhangsan |
   |  2 | lisi     |
   |  3 | wangwu   |
   +----+----------+
   ```

   在可重复读的情况下，为什么会出现这样的情况呢？

   原因就是SESS1 没有BEGIN之后开启一个查询，导致SESS1的`select * from t2 `查询的快照是执行该SQL的快照，而不是`BEGIN`那个时间点的快照，而此时，SESS2已经提交。

2. 我们来看一个例子：

   ```mysql
   CREATE TABLE T3 (id INT,name varchar(10) ,PRIMARY KEY (id)) ENGINE = InnoDB;
   INSERT INTO T3 VALUES(1,'a'),(2,'b'),(3,'c');
   
   
   SESS1:                      SESS2：
   BEGIN;
   SELECT * FROM T3;
   +----+------+
   | id | name |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   +----+------+
                               BEGIN;
                               INSERT INTO T3 values(4,'a');
                               COMMIT;
   SELECT * FROM T3;
   +----+------+
   | id | name |
   +----+------+
   |  1 | a    |
   |  2 | b    |
   |  3 | c    |
   +----+------+
   UPDATE T3 SET NAME='aa' where name ='a';
   Query OK, 2 rows affected (0.00 sec)
   Rows matched: 2  Changed: 2  Warnings: 0
   SELECT * FROM T3;                       
   +----+------+
   | id | name |
   +----+------+
   |  1 | aa   |
   |  2 | b    |
   |  3 | c    |
   |  4 | aa   |
   +----+------+
                               SELECT * FROM T3;                       
                               +----+------+
                               | id | name |
                               +----+------+
                               |  1 | a    |
                               |  2 | b    |
                               |  3 | c    |
                               |  4 | a    |
                               +----+------+
   ```

   在可重复读的情况下，为什么SESS1先开启事务的情况下，还能更新会话2后来提交的数据呢?然后之后的select还能看到更新后的数据？我们查看下官方解释。

   > A consistent read means that InnoDB uses multi-versioning to present to a query a snapshot of the database at a point in time. The query sees the changes made by transactions that committed before that point of time, and no changes made by later or uncommitted transactions. The exception to this rule is that the query sees the changes made by earlier statements within the same transaction. This exception causes the following anomaly: If you update some rows in a table, a SELECT sees the latest version of the updated rows, but it might also see older versions of any rows. If other sessions simultaneously update the same table, the anomaly means that you might see the table in a state that never existed in the database.

   默认情况下，innodb使用MVCC进行快照读。查询只能查看到之前提交的事务更改的数据，之后提交的数据或者没有提交的事务数据是没办法看到的。但是这里有个例外就是同一个事务的查询能看到同一个事务里面的更改。因此当SESS1 在进行`UPDATE`的时候，会进行当前读，也就是读取所有已经提交的数据，相当于读取的是`select * from t3 where where name ='a' for update;`的结果集，然后进行`UPDATE`。之后的`select * from t3 where where name ='a'`看到的就是当前`UPDATE`之后的结果了。