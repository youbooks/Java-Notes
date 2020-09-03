# mysql 联合索引匹配原则

原文链接：[mysql 联合索引匹配原则](https://skydh.github.io/2018/01/24/mysql索引/)

## 1. 读mysql文档有感

看了mysql关于索引的文档，网上有一些错误的博客文档，这里我自己记一下。

## 2. 几个重要的概念

１．对于mysql来说，一条sql中，一个表无论其蕴含的索引有多少，但是有且只用一条。 ２．对于多列索引来说（a,b,c）其相当于3个索引（a），（a,b），（a,b,c）3个索引，又由于mysql的索引优化器，其where条件后的语句是可以乱序的，比如（b,c,a）也是可以用到索引。如果条件中a,c出现的多，为了更好的利用索引故最好将其修改为（a.c,b）。

## 3. ICP概念

看了一篇大神的博客，上面说了通用索引匹配原则，这里也顺便说下。 １．Index range 先确认索引的起止范围。 ２．Index Filter 索引过滤。 ３．Table Filter 表过滤。 传说中mysql5.6后提出的icp就是多了第二步，以前Index filter是放在数据上操作的，现在5.6后多了第二步，因此效率提高了很多。

## 4. 案例分析

### 4.1 表结构

```sql
CREATE TABLE `left_test` (
          `id` int(11) NOT NULL,
          `a` int(11) DEFAULT NULL,
          `b` int(11) DEFAULT NULL,
          `c` int(11) DEFAULT NULL,
          `d` int(11) DEFAULT NULL,
         `e` int(11) DEFAULT NULL,
           PRIMARY KEY (`id`),
          KEY `m_index` (`a`,`b`,`c`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

且插入了100万条数据。

### 4.2 sql的分析

```sql
select * from left_table where id=1。
select * from left_table where id>1 and id<3
```

使用了聚集索引，id为主键，那么这个表里面id则是聚集索引列，这条sql默认使用了聚集索引来搜索。

```sql
select * from left_table where a=1
select * from left_table where a=1 and b=1
select * from left_table where a=1 and b=1 and c=1
```

使用联合索引（a,b,c）。其中这些条件可以可以乱序，因为mysql的sql优化器会优化这些代码。

```sql
select * from left_table where a<1
select * from left_table where a<1 and b<1
select * from left_table where a<1 and b<1 and c<1
```

对于现在mysql5.7中，小于可以乱序（mysql优化器优化了），但是按照最左匹配原则。比如条件\(b\)，\(c\),\(b,c\)组合就不行。

```sql
select * from left_table where b<1
select * from left_table where b<1 and c<1
select * from left_table where c<1
```

这个组合就用不到索引，因为不符合最左匹配原则。

```sql
select * from left_table where a=1 and id=2
```

这里面id是聚簇索引列，而a是个二级索引列，那么这个是用聚集索引列，不用（a,b,c）这个索引，因为对于mysql 5.7 innodb 这个版本一条sql里面索引只能用一条。至于用那个，则是mysql自身的算法选择了。经过大量测试实验，规则如下，如果索引列数据数据一模一样，那么是谁先创建就选谁，如不一样，那么谁占用的列越多，或者列的数据越复杂则选它。当然最好手动指定。

