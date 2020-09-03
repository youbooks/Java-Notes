# Dubbo的架构原理

原文链接: [https://juejin.im/post/5ab09943f265da238f125ee8](https://juejin.im/post/5ab09943f265da238f125ee8)

无论是Dubbo还是Dubbox，包括在之前[《聊聊Dubbo（一）：为何选择》](https://juejin.im/post/5a5ee63d518825732914748c)中介绍的其他框架，**其本质都是远程调用框架**，而对于远程调用如果没有分布式的需求，其实是不需要用这么重的框架，只有在分布式的时候，才有Dubbo这样的分布式服务框架的需求，说白了就是个远程服务调用的分布式框架，**其重点在于分布式的治理**。那Dubbox这样的框架在分布式治理方面带来了哪些核心功能呢？

## 1. Dubbo核心功能

1. **Remoting:远程通讯**，提供对多种NIO框架抽象封装，包括“同步转异步”和“请求-响应”模式的信息交换方式。
2. **Cluster: 服务框架**，提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
3. **Registry: 服务注册中心**，基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

**对于以上的3个核心功能，Dubbo有涉及到哪些组件角色，来协作完成分布式治理的呢？**

## 2. Dubbo组件角色

![](https://image.ldbmcs.com/2019-07-17-030635.jpg)

| 组件角色 | 说明 |
| :--- | :--- |
| Provider | 暴露服务的服务提供方 |
| Consumer | 调用远程服务的服务消费方 |
| Registry | 服务注册与发现的注册中心 |
| Monitor | 统计服务的调用次调和调用时间的监控中心 |
| Container | 服务运行容器 |

**调用关系说明：**

1. 服务容器Container负责启动，加载，运行服务提供者。
2. 服务提供者Provider在启动时，向注册中心注册自己提供的服务。
3. 服务消费者Consumer在启动时，向注册中心订阅自己所需的服务。
4. 注册中心Registry返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者Consumer，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者Consumer和提供者Provider，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。 

![](https://image.ldbmcs.com/2019-07-17-030921.jpg)

## 3. Dubbo总体架构

上面介绍给出的都是**抽象层面的组件关系，可以说是纵向的以服务模型的组件分析**，其实Dubbo最大的特点是**按照分层的方式来架构**，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。所以，我们**横向以分层的方式来看下Dubbo的架构**，如图所示：

![](https://image.ldbmcs.com/2019-07-17-030943.jpg)

Dubbo框架设计一共划分了10个层，而最上面的Service层是留给实际想要使用Dubbo开发分布式服务的开发者实现业务逻辑的接口层。**图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口， 位于中轴线上的为双方都用到的接口。**

下面，结合Dubbo官方文档，我们分别理解一下框架分层架构中，各个层次的设计要点：

1. **服务接口层（Service）**：与实际业务逻辑相关的，根据服务提供方和服务消费方的**业务设计对应的接口和实现**。
2. **配置层（Config）**：对外配置接口，以ServiceConfig和ReferenceConfig为中心，可以直接new配置类，也可以通过Spring解析配置生成配置类。
3. **服务代理层（Proxy）**：服务接口透明代理，**生成服务的客户端Stub和服务器端Skeleton**，以ServiceProxy为中心，扩展接口为ProxyFactory。
4. **服务注册层（Registry）**：封装服务地址的注册与发现，以服务URL为中心，扩展接口为RegistryFactory、Registry和RegistryService。可能没有服务注册中心，此时服务提供方直接暴露服务。
5. **集群层（Cluster）**：封装多个提供者的路由及负载均衡，并桥接注册中心，以Invoker为中心，扩展接口为Cluster、Directory、Router和LoadBalance。将多个服务提供方组合为一个服务提供方，实现对服务消费方来透明，只需要与一个服务提供方进行交互。
6. **监控层（Monitor）**：RPC调用次数和调用时间监控，以Statistics为中心，扩展接口为MonitorFactory、Monitor和MonitorService。
7. **远程调用层（Protocol）**：封将RPC调用，以Invocation和Result为中心，扩展接口为Protocol、Invoker和Exporter。Protocol是服务域，它是Invoker暴露和引用的主功能入口，它负责Invoker的生命周期管理。Invoker是实体域，它是Dubbo的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起invoke调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。
8. **信息交换层（Exchange）**：封装请求响应模式，同步转异步，以Request和Response为中心，扩展接口为Exchanger、ExchangeChannel、ExchangeClient和ExchangeServer。
9. **网络传输层（Transport）**：抽象mina和netty为统一接口，以Message为中心，扩展接口为Channel、Transporter、Client、Server和Codec。
10. **数据序列化层（Serialize）**：可复用的一些工具，扩展接口为Serialization、 ObjectInput、ObjectOutput和ThreadPool。

从上图可以看出，**Dubbo对于服务提供方和服务消费方，从框架的10层中分别提供了各自需要关心和扩展的接口，构建整个服务生态系统（服务提供方和服务消费方本身就是一个以服务为中心的）**。

**根据官方提供的，对于上述各层之间关系的描述，如下所示：**

1. 在RPC中，Protocol是核心层，也就是只要有Protocol + Invoker + Exporter就可以完成非透明的RPC调用，然后在Invoker的主过程上Filter拦截点。
2. 图中的Consumer和Provider是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用Client和Server的原因是Dubbo在很多场景下都使用Provider、Consumer、Registry、Monitor划分逻辑拓普节点，保持统一概念。
3. 而Cluster是外围概念，所以Cluster的目的是将多个Invoker伪装成一个Invoker，这样其它人只要关注Protocol层Invoker即可，加上Cluster或者去掉Cluster对其它层都不会造成影响，因为只有一个提供者时，是不需要Cluster的。
4. Proxy层封装了所有接口的透明化代理，而在其它层都以Invoker为中心，只有到了暴露给用户使用时，才用Proxy将Invoker转成接口，或将接口实现转成Invoker，也就是去掉Proxy层RPC是可以Run的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。
5. 而Remoting实现是Dubbo协议的实现，如果你选择RMI协议，整个Remoting都不会用上，Remoting内部再划为Transport传输层和Exchange信息交换层，Transport层只负责单向消息传输，是对Mina、Netty、Grizzly的抽象，它也可以扩展UDP传输，而Exchange层是在传输层之上封装了Request-Response语义。
6. Registry和Monitor实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。

## 4. 服务调用流程

![](https://image.ldbmcs.com/2019-07-17-031036.jpg)

## 5. 注册/注销服务

服务的注册与注销，是对服务提供方角色而言，那么注册服务与注销服务的时序图，如图所示：

![](https://image.ldbmcs.com/2019-07-17-031055.jpg)

## 6. 订阅/取消服务

为了满足应用系统的需求，服务消费方的可能需要从服务注册中心订阅指定的有服务提供方发布的服务，在得到通知可以使用服务时，就可以直接调用服务。反过来，如果不需要某一个服务了，可以取消该服务。下面看一下对应的时序图，如图所示：

![](https://image.ldbmcs.com/2019-07-17-031116.jpg)

## 7. 参考资料与推荐阅读

1. [dubbo.io](https://link.juejin.im/?target=http%3A%2F%2Fdubbo.io)
2. [Dubbo架构设计详解](https://link.juejin.im/?target=http%3A%2F%2Fshiyanjun.cn%2Farchives%2F325.html)

