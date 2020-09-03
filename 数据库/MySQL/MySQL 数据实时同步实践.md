## 1. 背景

MySQL 由于自身简单、高效、可靠的特点，成为小米内部使用最广泛的数据库，但是当数据量达到千万 / 亿级别的时候，MySQL 的相关操作会变的非常迟缓；如果这时还有实时 BI 展示的需求，对于 mysql 来说是一种灾难。

为了解决 sql 查询慢，查不了的业务痛点，我们探索出一套完整的实时同步，即席查询的解决方案，本文主要从实时同步的角度介绍相关工作。

早期业务借助 Sqoop 将 Mysql 中的数据同步到 Hive 来进行数据分析，使用过程中也带来了一些问题：

- 虽然 Sqoop 支持增量同步但还属于粗粒度的离线同步，无法满足实时性的需求。
- 每次同步 Sqoop 以 sql 的方式向 Mysql 发出数据请求也在一定程度上对 Mysql 带来一定的压力。
- 同时 Hive 对数据更新的支持也相对较弱。

为了更有效地连接前端业务数据系统（MySQL）和后端统计分析系统（查询分析引擎），我们需要一套实时同步 MySQL 数据的解决方案。

## 2. 小米内部实践

如何能够做到数据的实时同步呢？我们想到了 MySQL 主从复制时使用的 binlog 日志，它记录了所有的 DDL 和 DML 语句（除了数据查询语句 select、show 等），以事件形式记录，还包含语句所执行的消耗时间。

下面来看一下 MySQL 主从复制的原理，主要有以下几个步骤：

1. master（主库）在每次准备提交事务完成数据更新前，将改变记录到二进制日志 (binary log) 中。
2. slave（从库）发起连接，连接到 master，请求获取指定位置的 binlog 文件。
3. master 创建 dump 线程，推送 binlog 的 slave。
4. slave 启动一个 I/O 线程来读取主库上 binary log 中的事件，并记录到 slave 自己的中继日志 (relay log) 中。
5. slave 还会起动一个 SQL 线程，该线程从 relay log 中读取事件并在备库执行，完成数据同步。
6. slave 记录自己的 binlog。

![2020-07-06-F81z0F](https://image.ldbmcs.com/2020-07-06-F81z0F.jpg)

binlog 记录了 Mysql 数据的实时变化，是数据同步的基础，服务需要做的就是遵守 Mysql 的协议，**将自己伪装成 Mysql 的 slave 来监听业务从库，完成数据实时同步。**

结合小米内部系统特点，构建了 Mysql 数据同步服务–**LCSBinlog**，作为一种独立的数据接入方式整合在 Talos Platform 中，Talos Platform 作为大数据集成的基础解决方案，以自研消息队列 Talos 为数据总线，连接各种系统为主要目标，提供丰富的数据 Source 输入和数据 Sink 输出，并且 Talos 天然支持流式计算，因此业务可以充分利用 Talos Platform 互联互通的特性，并结合自身的业务需求实现更加高阶的业务场景。

![2020-07-06-VXigWl](https://image.ldbmcs.com/2020-07-06-VXigWl.jpg)

上图是 Talos Platform 中的整体流程架构，其中标红部分是目前 LCSBinlog 在小米内部使用最广泛的一条链路：Mysql —> Talos —> Kudu —> BI，数据同步到 kudu 后借助 Sparksql 查询引擎为上层 BI 系统提供即席查询服务，Kudu 和 Sparksql 的整合细节可以参见往期内容：[告别”纷纷扰扰”—小米 OLAP 服务架构演进](http://mp.weixin.qq.com/s?__biz=MzUxMDQxMDMyNg==&mid=2247485952&idx=1&sn=0b89096865f1033207efc90d936f49ed&chksm=f9022139ce75a82f5aa505db71312fb3642c0111c3818abf3b726d7a1ad79971c4c7611daab6&scene=21#wechat_redirect)

## 3. LCSBinlog 服务的主体架构

服务一共有两种角色

- Master ：主要负责作业的调度，

- Worker： 主要完成具体的数据同步任务

在 Worker 上运行两种作业：

1. BinlogSyncJob：每一个 mysql 库都会对应这样一个 Job，将 binlog 日志完整地写入到服务创建的 Talos topic 中。
2. MysqlSyncJob：同步历史数据，消费 binlog 数据，过滤特定库表数据实时同步至用户配置的 topic 中。

服务整体依赖于 Zookeeper 来同步服务状态，记录作业调度信息和标记作业运行状态；在 kudu 表中记录作业同步进度。

![2020-07-06-Wt7sic](https://image.ldbmcs.com/2020-07-06-Wt7sic.jpg)

控制流程如下：

1. Worker 节点通过在 Zookeeper 上注册告知自己可以被调度。
2. 通过在 Zookeeper 上抢占 EPHEMERAL 临时节点实现 Master 的 HA。
3. 用户在融合云（Web）上注册 BinlogSource 同步任务。
4. Master 周期性从配置服务读取 Binlog 同步作业配置。
5. Master 更新 Zookeeper 中的调度信息。
6. Worker 节点 根据 Zookeeper 上的调度信息启动新分配任务，停止配置失效任务；作业启动后完成数据实时同步并周期性将同步进度记录在 kudu 中。
7. 服务上报监控信息到 Falcon 平台，作业异常退出发送报警邮件。

## 4. 如何保障数据正确性

### 4.1 顺序性

用户配置的每一个 BinlogSource 都会绑定一个 Talos 的 topic，在进行消费的时候需要保证同一条 mysql 记录操作的顺序性，消息队列 Talos 是无法保证全局消息有序的，只能保证 partition 内部有序。

对于配置分库分表或者多库同步任务的 BinlogSource，服务会根据库表信息进行 hash，将数据写入相应的 partiton，保证同一张表的数据在一个 partition 中，使得下游消费数据的顺序性；对于单表同步的作业目前使用一个 partition 保证其数据有序。

### 4.2 一致性

如何保证在作业异常退出后，作业重新启动能够完整地将 mysql 中的数据同步到下游系统，主要依赖于以下三点：

1. 服务会记录作业同步的 offset，重启后从上次 commit 的 offset 继续消费
2. Binlog 数据的顺序性保证了即便数据被重复消费（未 commit 的数据），也能对同一条记录的操作以相同的顺序执行
3. 下游存储系统 kudu，Es ，Redis 基于主键的操作能够保证 binlog 重复回放后数据的最终一致性

## 5. 应用场景

有了这份数据我们可以做些什么事情呢，本节例举了几种常见的应用场景：

### 5.1 实时更新缓存

业务查询类服务往往会在 mysql 之上架设一个缓存，减少对底层数据库的访问；当 mysql 库数据变化时，如果缓存还没有过期那么就会拿到过期的数据，业务期望能够实时更新缓存；

利用 binlog 服务，根据策略实时将数据同步到 redis 中，这样就能够保证了缓存中数据有效性，减少了对数据库的调用，从而提高整体性能。

![2020-07-06-zDPU1P](https://image.ldbmcs.com/2020-07-06-zDPU1P.jpg)

### 5.2 异步处理，系统解耦

随着业务的发展，同一份数据可能有不同的分析用途，数据成功写入到 mysql 的同时也需要被同步到其他系统；如果用同步的方式处理，一方面拉长了一次事务整个流程，另一方面系统间也会相互影响

数据在 mysql 中操作成功后才会记录在 binlog 中，保证下游处理到时的一致性；使用 binlog 服务完成数据的下发，有助于系统的解耦

关于异步处理，系统解耦在[消息队列价值思考](http://mp.weixin.qq.com/s?__biz=MzUxMDQxMDMyNg==&mid=2247485986&idx=1&sn=795b2121c71df3780dfe15620351d388&chksm=f902211bce75a80dac5993b2393b6063f6bba6649fa3f58c02ad7773b6e68b7a8144b8a2637a&scene=21#wechat_redirect)一文中有更深入的解读。

### 5.3 即席查询的 BI 系统

就如文章开篇提到的，mysql 在一定场景下的性能瓶颈，mysql 数据同步到 kudu 后可以借助 sparksql 完成性能的提升。

因为同样是 sql 接口，对使用者的切换成本也是较低的，数据同步到更适合的存储中进行查询，也能够避免因大查询而对原 mysql 库其他查询的影响。

目前小米内部稳定运行 3000+ 的同步作业，使用 binlog 服务同步数据到 kudu 中；小米内部 BI 明星产品 XDATA 借助整套同步流程很好地支持了运营、sql 分析同学日常统计分析的需求。

## 6. 如何使用 Binlog 数据

用户接入数据的时候要求 mysql 库开启 binlog 日志格式必须为 Row 模式：记录的是每一行记录的每个字段变化前后的值，虽然会造成 binlog 数据量的增多，但是能够确保每一条记录准确性，避免数据同步不一致情况的出现。

最终通过监听 binlog 日志，LCSBinlog 服务将数据转换成如下的数据结构，写入用户注册的 Topic 中， 目前 Sink 服务使用 SparkStreaming 实时转储数据到 kudu 中，后续也将逐步迁移到 Flink 上以提升资源利用、降低延迟。

![2020-07-06-rUrfpO](https://image.ldbmcs.com/2020-07-06-rUrfpO.jpg)

业务用户也可以根据我们提供的数据格式，实时消费 Talos 数据以实现更复杂的业务逻辑，下表为每一种数据操作，是否保存修改前后的列表。

![2020-07-06-Vvl2ZQ](https://image.ldbmcs.com/2020-07-06-Vvl2ZQ.jpg)

疑难杂症下面分享 2 个上线后遇到的有趣问题：

### 6.1 数据不一致问题，业务使用唯一索引

业务接入一段时间后, 发现部分表会偶尔存在 kudu 表的数据条目数多于同步的 mysql 表的数据条目数，我们将多出来的数据与 mysql 产生的 binlog 日志经过一一对比，发现用户在 mysql 表中设置了唯一索引，通过唯一索引修改了主键，而 kudu 中的数据是通过主键标识或更新一条记录的，于是 update 操作变成了 insert 操作，这就造成了原来的 1 条记录变成了 2 条。

解决办法：对于这种类型的表，LCSBinlog 服务会把一次 Update 操作转换成一条 Delete 数据和一条 Insert 数据。

### 6.2 Full Dump 同步历史数据时，客户端超时

服务刚上线的时候，通过 jdbc 执行 sql 的方式完成全量历史数据的同步，在同步的过程中会发现 dump 任务会卡顿很长时间才会返回结果，当数据量很大会出现超时同步失败的情况，会造成数据的延迟。调研后发现使用 mysql 官方 jdbc 在客户端查询数据的时候，默认为从服务器一次取出所有数据放在客户端内存中，fetch size 参数不起作用，当一条 SQL 返回数据量较大时可能会出现 OOM

解决办法：当 statement 设置以下属性时，采用的是流数据接收方式，每次只从服务器接收部份数据，直到所有数据处理完毕。优化后历史数据同步稳定运行，对 mysql 端的压力也很小。

![2020-07-06-hoHfTX](https://image.ldbmcs.com/2020-07-06-hoHfTX.jpg)

## 7. 总结

MySQL 以 Binlog 日志的方式记录数据变化，基于流式数据的 Change Data Caputre (CDC) 机制实现了 LCSBinlog 服务，

本文主要对 LCSBinlog 的服务架构、应用场景以及在小米内部的实践经验进行了介绍，也和大家分享了我们实际中遇到的问题和解决方案，希望能够帮助到大家理解服务的原理，带来启发，也欢迎大家和我们一起交流。

## 8. 参考

1. [MySQL 数据实时同步实践](https://mp.weixin.qq.com/s/n9gvt_dSqqnDavnA9D6ImA)

