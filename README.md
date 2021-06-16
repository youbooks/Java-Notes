写在最前面：大多数文章都是转载的，我只是做了整理工作，如果对大家有帮助，希望star一波，谢谢大家。

GitHub地址：https://github.com/ldbmcs/Java-Notes

推荐使用[GitBook](https://ldbmcs.gitbook.io/java/)阅读。

```
├─ Java
│  ├─ Java IO&NIO&AIO
│  │  ├─ Java AIO - 异步IO详解.md
│  │  ├─ Java IO - BIO 详解.md
│  │  ├─ Java IO - Unix IO模型.md
│  │  ├─ Java IO - 分类.md
│  │  ├─ Java IO - 常见类使用.md
│  │  ├─ Java IO - 概述.md
│  │  ├─ Java IO - 设计模式.md
│  │  ├─ Java N(A)IO - Netty.md
│  │  ├─ Java NIO - IO多路复用详解.md
│  │  ├─ Java NIO - 基础详解.md
│  │  └─ Java NIO - 零拷贝实现.md
│  ├─ Java JVM
│  │  ├─ JVM 优化经验总结.md
│  │  ├─ JVM 内存结构.md
│  │  ├─ JVM参数设置.md
│  │  ├─ Java 内存模型.md
│  │  ├─ 从实际案例聊聊Java应用的GC优化.md
│  │  ├─ 垃圾回收器G1详解.md
│  │  ├─ 垃圾回收器Shenandoah GC详解.md
│  │  ├─ 垃圾回收器ZGC详解.md
│  │  ├─ 垃圾回收基础.md
│  │  ├─ 如何优化Java GC.md
│  │  ├─ 类加载机制.md
│  │  └─ 类字节码详解.md
│  ├─ Java Library
│  │  └─ 使用ModelMapper的一次踩坑经历.md
│  ├─ Java 基础
│  │  ├─ Java hashCode() 和 equals().md
│  │  ├─ Java native方法以及JNI实践.md
│  │  ├─ Java serialVersionUID 有什么作用？.md
│  │  ├─ Java 泛型的类型擦除.md
│  │  ├─ Java中final关键字详解.md
│  │  ├─ Java中static关键字详解.md
│  │  ├─ Java多态的面试题.md
│  │  ├─ SPI机制详解.md
│  │  ├─ Unsafe类解析.md
│  │  ├─ 为什么 String hashCode 方法选择数字31作为乘子.md
│  │  ├─ 为什么要有抽象类？.md
│  │  ├─ 为什么说Java中只有值传递？.md
│  │  ├─ 即时编译器原理解析及实践.md
│  │  ├─ 反射机制详解.md
│  │  ├─ 异常机制详解.md
│  │  ├─ 接口的本质.md
│  │  ├─ 泛型机制详解.md
│  │  └─ 注解机制详解.md
│  ├─ Java 并发
│  │  ├─ Java 并发 - 14个Java并发容器.md
│  │  ├─ Java 并发 - AQS.md
│  │  ├─ Java 并发 - BlockingQueue.md
│  │  ├─ Java 并发 - CAS.md
│  │  ├─ Java 并发 - Condition接口.md
│  │  ├─ Java 并发 - CopyOnWriteArrayList.md
│  │  ├─ Java 并发 - CountDownLatch、CyclicBarrier和Phaser对比.md
│  │  ├─ Java 并发 - Fork&Join框架.md
│  │  ├─ Java 并发 - Java CompletableFuture 详解.md
│  │  ├─ Java 并发 - Java 线程池.md
│  │  ├─ Java 并发 - Lock接口.md
│  │  ├─ Java 并发 - ReentrantLock.md
│  │  ├─ Java 并发 - ReentrantReadWriteLock.md
│  │  ├─ Java 并发 - Synchronized.md
│  │  ├─ Java 并发 - ThreadLocal 内存泄漏问题.md
│  │  ├─ Java 并发 - ThreadLocal.md
│  │  ├─ Java 并发 - Volatile.md
│  │  ├─ Java 并发 - 从ReentrantLock的实现看AQS的原理及应用.md
│  │  ├─ Java 并发 - 公平锁和非公平锁.md
│  │  ├─ Java 并发 - 内存模型.md
│  │  ├─ Java 并发 - 原子类.md
│  │  ├─ Java 并发 - 如何确保三个线程顺序执行？.md
│  │  └─ Java 并发 - 锁.md
│  ├─ Java 框架
│  │  ├─ Logback
│  │  │  └─ 自定义 logback 日志过滤器.md
│  │  ├─ Mybatis
│  │  │  ├─ 从源码的角度解析Mybatis的会话机制.md
│  │  │  ├─ 浅析pagehelper分页原理.md
│  │  │  ├─ 深入理解Mybatis技术与原理.md
│  │  │  └─ 聊聊MyBatis缓存机制.md
│  │  ├─ Netty
│  │  │  ├─ Netty 可靠性分析.md
│  │  │  ├─ Netty 线程模型.md
│  │  │  ├─ Netty堆外内存泄露排查盛宴.md
│  │  │  └─ Netty高性能之道.md
│  │  ├─ Shiro
│  │  │  ├─ Shiro + JWT + Spring Boot Restful 简易教程.md
│  │  │  └─ 非常详尽的 Shiro 架构解析！.md
│  │  ├─ Spring
│  │  │  ├─ Spring AOP 使用介绍，从前世到今生.md
│  │  │  ├─ Spring AOP 源码解析.md
│  │  │  ├─ Spring Event 实现原理.md
│  │  │  ├─ Spring Events.md
│  │  │  ├─ Spring IOC容器源码分析.md
│  │  │  ├─ Spring Integration简介.md
│  │  │  ├─ Spring MVC 框架中拦截器 Interceptor 的使用方法.md
│  │  │  ├─ Spring bean 解析、注册、实例化流程源码剖析.md
│  │  │  ├─ Spring validation中@NotNull、@NotEmpty、@NotBlank的区别.md
│  │  │  ├─ Spring 如何解决循环依赖？.md
│  │  │  ├─ Spring 异步实现原理与实战分享.md
│  │  │  ├─ Spring中的“for update”问题.md
│  │  │  ├─ Spring中的设计模式.md
│  │  │  ├─ Spring事务失效的 8 大原因.md
│  │  │  ├─ Spring事务管理详解.md
│  │  │  ├─ Spring计时器StopWatch使用.md
│  │  │  ├─ 详述 Spring MVC 框架中拦截器 Interceptor 的使用方法.md
│  │  │  └─ 透彻的掌握 Spring 中@transactional 的使用.md
│  │  ├─ Spring Boot
│  │  │  ├─ Spring Boot 使用ApplicationListener监听器.md
│  │  │  ├─ Spring Boot引起的“堆外内存泄漏”排查及经验总结.md
│  │  │  ├─ Spring Boot的启动流程.md
│  │  │  ├─ Spring Boot自动化配置源码分析.md
│  │  │  └─ 如何自定义Spring Boot Starter？.md
│  │  └─ Spring Security
│  │     ├─ Spring Boot 2 + Spring Security 5 + JWT 的单页应用 Restful 解决方案.md
│  │     ├─ Spring Security Oauth.md
│  │     ├─ Spring Security.md
│  │     └─ 理解Oauth2.0.md
│  ├─ Java 调试排错
│  │  ├─ 调式排错 - Java Debug Interface(JDI)详解.md
│  │  ├─ 调试排错 - CPU 100% 排查优化实践.md
│  │  ├─ 调试排错 - Java Heap Dump分析.md
│  │  ├─ 调试排错 - Java Thread Dump分析.md
│  │  ├─ 调试排错 - Java动态调试技术原理.md
│  │  ├─ 调试排错 - Java应用在线调试Arthas.md
│  │  ├─ 调试排错 - Java问题排查：工具单.md
│  │  ├─ 调试排错 - 内存溢出与内存泄漏.md
│  │  ├─ 调试排错 - 在线分析GC日志的网站GCeasy.md
│  │  └─ 调试排错 - 常见的GC问题分析与解决.md
│  ├─ Java 集合
│  │  ├─ Java 集合 - ArrayList.md
│  │  ├─ Java 集合 - HashMap 和 ConcurrentHashMap.md
│  │  ├─ Java 集合 - HashMap的死循环问题.md
│  │  ├─ Java 集合 - LinkedHashSet&Map.md
│  │  ├─ Java 集合 - LinkedList.md
│  │  ├─ Java 集合 - PriorityQueue.md
│  │  ├─ Java 集合 - Stack & Queue.md
│  │  ├─ Java 集合 - TreeSet & TreeMap.md
│  │  ├─ Java 集合 - WeakHashMap.md
│  │  ├─ Java 集合 - 为什么HashMap的容量是2的幂次方.md
│  │  ├─ Java 集合 - 概览.md
│  │  └─ Java 集合 - 高性能队列Disruptor详解.md
│  ├─ Java8 以上特性
│  │  ├─ Java 10 新特性概述.md
│  │  ├─ Java 11 新特性概述.md
│  │  ├─ Java 12 新特性概述.md
│  │  ├─ Java 13 新特性概述.md
│  │  ├─ Java 14 新特性概述.md
│  │  ├─ Java 15 新特性概述.md
│  │  └─ Java 9 新特性概述.md
│  ├─ Java8 特性
│  │  ├─ Java 8 - Guide To Java 8 Optional.md
│  │  ├─ Java 8 - JRE精简.md
│  │  ├─ Java 8 - JavaFx 2.0.md
│  │  ├─ Java 8 - Lambda 表达式.md
│  │  ├─ Java 8 - LocalDate&LocalDateTime.md
│  │  ├─ Java 8 - StampedLock.md
│  │  ├─ Java 8 - 其它更新.md
│  │  ├─ Java 8 - 函数式接口Function、Consumer、Predicate、Supplier.md
│  │  ├─ Java 8 - 移除Permgen.md
│  │  ├─ Java 8 - 类型推断优化.md
│  │  ├─ Java 8 - 类型注解.md
│  │  ├─ Java 8 - 重复注解.md
│  │  └─ Java 8 - 默认方法.md
│  └─ 设计模式
│     ├─ 设计模式 - JDK中的设计模式.md
│     ├─ 设计模式 - Java三种代理模式.md
│     ├─ 设计模式 - 六大设计原则.md
│     ├─ 设计模式 - 单例模式.md
│     ├─ 设计模式 - 命名模式.md
│     ├─ 设计模式 - 备忘录模式.md
│     ├─ 设计模式 - 概览.md
│     └─ 设计模式 - 没用的设计模式.md
├─ Java 8中日期
├─ Java进阶.md
├─ README.md
├─ 云原生
│  └─ 什么是无服务器(what is serverless)？.md
├─ 分享
│  └─ Agile Coach
│     ├─ # Branching.md
│     ├─ 6.15.md
│     ├─ Agile is a competitive advantage.md
│     ├─ Going agila.md
│     └─ branching.md
├─ 分布式
│  ├─ RPC
│  │  ├─ RPC - Dubbo&hsf&Spring cloud的区别.md
│  │  ├─ RPC - Dubbo的架构原理.md
│  │  ├─ RPC - HSF的原理分析.md
│  │  ├─ RPC - 你应该知道的RPC原理.md
│  │  ├─ RPC - 动态代理.md
│  │  ├─ RPC - 协议.md
│  │  ├─ RPC - 序列化和反序列化.md
│  │  ├─ RPC - 服务注册与发现.md
│  │  ├─ RPC - 核心原理.md
│  │  ├─ RPC - 网络通信.md
│  │  └─ RPC框架对比.md
│  ├─ 分布式事务
│  │  ├─ 分布式事务 Seata TCC 模式深度解析.md
│  │  ├─ 分布式事务的实现原理.md
│  │  ├─ 常用的分布式事务解决方案.md
│  │  └─ 手写实现基于消息队列的分布式事务框架.md
│  ├─ 分布式算法
│  │  ├─ CAP 定理的含义.md
│  │  ├─ Paxos和Raft比较.md
│  │  └─ 分布式一致性与共识算法.md
│  ├─ 分布式锁
│  │  └─ 分布式锁的原理及实现方式.md
│  ├─ 协调服务
│  │  ├─ Zookeeper
│  │  │  ├─ Zookeeper - 客户端之 Curator.md
│  │  │  └─ Zookeeper -分布式协调服务 ZooKeeper详解.md
│  │  └─ etcd
│  │     └─ 高可用分布式存储 etcd 的实现原理.md
│  ├─ 搜索引擎
│  │  ├─ ElasticSearch与SpringBoot的集成与JPA方法的使用.md
│  │  ├─ 全文搜索引擎 Elasticsearch 入门教程.md
│  │  ├─ 十分钟学会使用 Elasticsearch 优雅搭建自己的搜索系统.md
│  │  └─ 腾讯万亿级 Elasticsearch 技术解密.md
│  ├─ 日志系统
│  │  ├─ Grafana Loki 简明教程.md
│  │  ├─ 分布式系统中如何优雅地追踪日志.md
│  │  ├─ 日志收集组件—Flume、Logstash、Filebeat对比.md
│  │  └─ 集中式日志系统 ELK 协议栈详解.md
│  ├─ 消息队列
│  │  ├─ 消息队列 - Kafka.md
│  │  ├─ 消息队列 - Kafka、RabbitMQ、RocketMQ等消息中间件的对比.md
│  │  ├─ 消息队列 - RabbitMQ.md
│  │  ├─ 消息队列 - 使用docker-compose构建kafka集群.md
│  │  ├─ 消息队列 - 分布式系统与消息的投递.md
│  │  ├─ 消息队列 - 如何保证消息的可靠性传输.md
│  │  ├─ 消息队列 - 如何保证消息的顺序性.md
│  │  ├─ 消息队列 - 如何保证消息队列的高可用.md
│  │  └─ 消息队列 - 消息队列设计精要.md
│  └─ 监控系统
│     └─ 深度剖析开源分布式监控CAT.md
├─ 大数据
│  └─ Flink
│     └─ Flink架构与核心组件.md
├─ 如何快速的学习一门新的技术
├─ 学习&成长
│  └─ 如何成为技术大牛.md
├─ 开发工具
│  ├─ Git Commit Message Guidelines.md
│  ├─ Git命令大全.md
│  ├─ Gradle vs Maven Comparison.md
│  ├─ Swagger2常用注解及其说明.md
│  └─ 简明 VIM 练级攻略.md
├─ 微服务
│  ├─ DDD
│  │  ├─ Domain Primitive.md
│  │  ├─ Repository模式.md
│  │  ├─ 从三明治到六边形.md
│  │  ├─ 应用架构.md
│  │  ├─ 聊聊如何避免写流水账代码.md
│  │  ├─ 阿里盒马领域驱动设计实践.md
│  │  ├─ 领域层设计规范.md
│  │  ├─ 领域驱动设计(DDD)编码实践.md
│  │  └─ 领域驱动设计在互联网业务开发中的实践.md
│  ├─ Dubbo
│  │  ├─ 基于dubbo的分布式应用中的统一异常处理.md
│  │  └─ 服务调用过程.md
│  ├─ Service Mesh
│  │  ├─ Istio 是什么？.md
│  │  ├─ OCTO 2.0：美团基于Service Mesh的服务治理系统详解.md
│  │  ├─ Service Mesh是什么？.md
│  │  ├─ Spring Cloud向Service Mesh迁移.md
│  │  ├─ conduit.md
│  │  └─ linkerd.md
│  └─ Spring Cloud
│     ├─ Spring Cloud Alibaba
│     │  ├─ Nacos
│     │  │  ├─ Nacos 服务注册的原理.md
│     │  │  └─ Nacos 配置中心原理分析.md
│     │  ├─ Seata
│     │  │  ├─ SEATA Saga 模式.md
│     │  │  ├─ Seata AT 模式.md
│     │  │  ├─ Seata TCC 模式.md
│     │  │  └─ Seata XA 模式.md
│     │  └─ Sentinel
│     │     ├─ Sentinel 与 Hystrix 的对比.md
│     │     └─ Sentinel.md
│     ├─ Spring Cloud Bus.md
│     ├─ Spring Cloud Config.md
│     ├─ Spring Cloud Consul.md
│     ├─ Spring Cloud Gateway.md
│     ├─ Spring Cloud Netflix
│     │  └─ Hystrix
│     │     ├─ How Hystrix Works.md
│     │     ├─ Hystrix Overview.md
│     │     ├─ Hystrix原理与实战.md
│     │     └─ Spring Cloud Hystrix基本原理.md
│     ├─ Spring Cloud OpenFeign.md
│     └─ Spring Cloud Stream.md
├─ 数据库
│  ├─ Database Version Control
│  │  ├─ Liquibase
│  │  │  ├─ Introduction to Liquibase Rollback.md
│  │  │  ├─ LiquiBase中文学习指南.md
│  │  │  └─ Use Liquibase to Safely Evolve Your Database Schema.md
│  │  ├─ Liquibase vs. Flyway.md
│  │  ├─ Six reasons to version control your database.md
│  │  └─ flyway
│  │     ├─ Database Migrations with Flyway.md
│  │     ├─ How Flyway works.md
│  │     ├─ Rolling Back Migrations with Flyway.md
│  │     └─ The meaning of the concept of checksums.md
│  ├─ MySQL
│  │  ├─ How Sharding Works.md
│  │  ├─ MySQL InnoDB中各种SQL语句加锁分析.md
│  │  ├─ MySQL 事务隔离级别和锁.md
│  │  ├─ MySQL 索引性能分析概要.md
│  │  ├─ MySQL 索引设计概要.md
│  │  ├─ MySQL出现Waiting for table metadata lock的原因以及解决方法.md
│  │  ├─ MySQL实战.xmind
│  │  ├─ MySQL的Limit性能问题.md
│  │  ├─ MySQL索引优化explain.md
│  │  ├─ MySQL索引背后的数据结构及算法原理.md
│  │  ├─ MySQL行转列、列转行问题.md
│  │  ├─ 一条SQL更新语句是如何执行的？.md
│  │  ├─ 一条SQL查询语句是如何执行的？.md
│  │  ├─ 为什么 MySQL 使用 B+ 树.md
│  │  ├─ 为什么 MySQL 的自增主键不单调也不连续.md
│  │  ├─ 为什么我的MySQL会“抖”一下？.md
│  │  ├─ 为什么数据库不应该使用外键.md
│  │  ├─ 为什么数据库会丢失数据.md
│  │  ├─ 事务的可重复读的能力是怎么实现的？.md
│  │  ├─ 大众点评订单系统分库分表实践.md
│  │  ├─ 如何保证缓存与数据库双写时的数据一致性？.md
│  │  ├─ 浅谈数据库并发控制 - 锁和 MVCC.md
│  │  ├─ 深入浅出MySQL 中事务的实现.md
│  │  └─ 深入浅出MySQL 和 InnoDB.md
│  ├─ PostgreSQL
│  │  └─ PostgreSQL upsert功能(insert on conflict do)的用法.md
│  ├─ Redis
│  │  ├─ Redis GEO & 实现原理深度分析.md
│  │  ├─ Redis 和 IO 多路复用.md
│  │  ├─ Redis分布式锁.md
│  │  ├─ Redis实现分布式锁中的“坑”.md
│  │  ├─ Redis总结.md
│  │  ├─ Redis高可用技术解决方案.md
│  │  ├─ Redlock：Redis分布式锁最牛逼的实现.md
│  │  └─ 为什么 Redis 选择单线程模型.md
│  ├─ TiDB
│  │  └─ 新一代数据库TiDB在美团的实践.md
│  └─ 数据库原理
│     └─ 为什么 OLAP 需要列式存储.md
├─ 系统设计
│  ├─ 可扩展架构
│  ├─ 基础架构
│  │  └─ 容错，高可用和灾备.md
│  ├─ 数据聚合
│  │  ├─ GraphQL及元数据驱动架构在后端BFF中的实践.md
│  │  └─ 高效研发-闲鱼在数据聚合上的探索与实践.md
│  ├─ 架构案例
│  │  └─ 微信 Android 客户端架构演进之路.md
│  ├─ 流量控制
│  │  ├─ RateLimiter
│  │  │  └─ Guava Rate Limiter实现分析.md
│  │  ├─ Sentinel
│  │  │  ├─ Sentinel 与 Hystrix 的对比.md
│  │  │  └─ Sentinel工作主流程.md
│  │  └─ 算法
│  │     └─ 分布式服务限流实战.md
│  ├─ 解决方案
│  │  ├─ 推荐系统
│  │  ├─ 秒杀系统
│  │  │  └─ 如何设计一个秒杀系统.md
│  │  └─ 红包系统
│  │     └─ 微信高并发资金交易系统设计方案--百亿红包背后的技术支撑.md
│  ├─ 高可用架构
│  │  └─ 业务高可用的保障：异地多活架构.md
│  └─ 高性能架构
├─ 计算机基础
│  ├─ 字符编码
│  │  └─ 字符编码笔记：ASCII，Unicode 和 UTF-8.md
│  ├─ 常见协议
│  │  └─ MQTT - The Standard for IoT Messaging.md
│  ├─ 操作系统
│  │  ├─ 为什么 CPU 访问硬盘很慢.md
│  │  ├─ 为什么 HTTPS 需要 7 次握手以及 9 倍时延.md
│  │  ├─ 为什么 Linux 默认页大小是 4KB.md
│  │  ├─ 为什么 TCP 协议有性能问题.md
│  │  ├─ 为什么 TCP 协议有粘包问题.md
│  │  ├─ 为什么 TCP 建立连接需要三次握手.md
│  │  ├─ 为什么 TCPIP 协议会拆分数据.md
│  │  ├─ 磁盘IO那些事.md
│  │  └─ 虚拟机的3种网络模式.md
│  ├─ 数据结构与算法
│  │  ├─ 其他相关
│  │  │  ├─ 什么是预排序遍历树算法（MPTT）.md
│  │  │  ├─ 加密算法.md
│  │  │  ├─ 推荐系统算法.md
│  │  │  ├─ 数据挖掘算法.md
│  │  │  ├─ 查找算法.md
│  │  │  ├─ 缓存淘汰算法中的LRU和LFU.md
│  │  │  └─ 负载均衡算法.md
│  │  ├─ 分布式算法
│  │  │  ├─ 分布式算法 - Paxos算法.md
│  │  │  ├─ 分布式算法 - Raft算法.md
│  │  │  ├─ 分布式算法 - Snowflake算法.md
│  │  │  ├─ 分布式算法 - ZAB算法.md
│  │  │  └─ 分布式算法 - 一致性Hash算法.md
│  │  ├─ 大数据处理
│  │  │  ├─ 大数据处理 - Bitmap & Bloom Filter.md
│  │  │  ├─ 大数据处理 - Map & Reduce.md
│  │  │  ├─ 大数据处理 - Trie树数据库倒排索引.md
│  │  │  ├─ 大数据处理 - 分治hash排序.md
│  │  │  ├─ 大数据处理 - 双层桶划分.md
│  │  │  ├─ 大数据处理 - 外（磁盘文件）排序.md
│  │  │  ├─ 大数据处理 - 布隆过滤器.md
│  │  │  └─ 大数据处理算法.md
│  │  ├─ 字符串匹配算法
│  │  │  ├─ 字符串匹配 - 文本预处理：后缀树（Suffix Tree）.md
│  │  │  ├─ 字符串匹配 - 模式预处理：BM 算法 (Boyer-Moore).md
│  │  │  ├─ 字符串匹配 - 模式预处理：KMP 算法（Knuth-Morris-Pratt）.md
│  │  │  ├─ 字符串匹配 - 模式预处理：朴素算法（Naive)(暴力破解).md
│  │  │  └─ 字符串匹配.md
│  │  ├─ 常用算法
│  │  │  ├─ 分支限界算法.md
│  │  │  ├─ 分治算法.md
│  │  │  ├─ 动态规划算法.md
│  │  │  ├─ 回溯算法.md
│  │  │  └─ 贪心算法.md
│  │  ├─ 排序算法
│  │  │  ├─ 十大排序算法.md
│  │  │  ├─ 图解排序算法(一)之3种简单排序(选择，冒泡，直接插入).md
│  │  │  ├─ 图解排序算法(三)之堆排序.md
│  │  │  ├─ 图解排序算法(二)之希尔排序.md
│  │  │  └─ 图解排序算法(四)之归并排序.md
│  │  └─ 数据结构
│  │     ├─ 树的高度和深度.md
│  │     ├─ 红黑树深入剖析及Java实现.md
│  │     ├─ 线性结构 - Hash.md
│  │     ├─ 线性结构 - 数组、链表、栈、队列.md
│  │     └─ 逻辑结构 - 树.md
│  ├─ 服务器
│  │  ├─ Mac终端bash、zsh、oh-my-zsh最实用教程.md
│  │  ├─ Nginx强制跳转Https.md
│  │  └─ curl 的用法指南.md
│  ├─ 网络安全
│  │  ├─ 如何设计一个安全的对外接口？.md
│  │  └─ 浅谈常见的七种加密算法及实现.md
│  └─ 网络编程
│     ├─ JSON Web Token 入门教程.md
│     ├─ 两万字长文 50+ 张趣图带你领悟网络编程的内功心法.md
│     ├─ 使用 OAuth 2 和 JWT 为微服务提供安全保障.md
│     ├─ 四种常见的 POST 提交数据方式.md
│     ├─ 理解OAuth 2.0.md
│     ├─ 看完这篇HTTP，跟面试官扯皮就没问题了.md
│     └─ 详细解析 HTTP 与 HTTPS 的区别.md
├─ 质量&效率
│  ├─ Homebrew 替换国内镜像源.md
│  ├─ 工作中如何做好技术积累.md
│  ├─ 快捷键
│  │  ├─ Idea快捷键（Mac版）.md
│  │  ├─ Shell快捷键.md
│  │  └─ Vim快捷键.md
│  └─ 敏捷开发
│     ├─ Scrum的3种角色.md
│     ├─ Scrum的4种会议.md
│     ├─ ThoughtWorks的敏捷开发.md
│     └─ 敏捷开发入门教程.md
├─ 运维&测试
│  ├─ Docker
│  │  ├─ Docker (容器) 的原理.md
│  │  ├─ Docker Compose：链接外部容器的几种方式.md
│  │  ├─ Docker 入门教程.md
│  │  ├─ Docker 核心技术与实现原理.md
│  │  ├─ Dockerfile 最佳实践.md
│  │  ├─ Docker开启Remote API 访问 2375端口.md
│  │  └─ Watchtower - 自动更新 Docker 镜像与容器.md
│  ├─ Kubernetes
│  │  ├─ Kubernetes 介绍.md
│  │  ├─ Kubernetes 在有赞的实践.md
│  │  ├─ Kubernetes 学习路径.md
│  │  ├─ Kubernetes如何改变美团的云基础设施？.md
│  │  ├─ NodePort、LoadBalancer 和 Ingress.md
│  │  └─ 谈 Kubernetes 的架构设计与实现原理.md
│  ├─ 压测
│  │  └─ 全链路压测平台（Quake）在美团中的实践.md
│  └─ 测试
│     ├─ Cpress - JavaScript End to End Testing Framework.md
│     ├─ Cypress
│     ├─ Spock
│     │  ├─ Groovy 简明教程.md
│     │  └─ Spock 官方文档.md
│     ├─ TDD
│     │  ├─ TDD 实践 - FizzFuzzWhizz（一）.md
│     │  ├─ TDD 实践 - FizzFuzzWhizz（三）.md
│     │  ├─ TDD 实践 - FizzFuzzWhizz（二）.md
│     │  └─ 测试驱动开发（TDD）- 原理篇.md
│     ├─ 代码覆盖率-JaCoCo.md
│     ├─ 浅谈代码覆盖率.md
│     └─ 测试中 Fakes、Mocks 以及 Stubs 概念明晰.md
└─ 面试.md

```