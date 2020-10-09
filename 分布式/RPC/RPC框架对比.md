> 转载：[6种微服务RPC框架，你知道几个？](https://juejin.im/post/6844903920402333709)

## 1. 前言

开源 RPC 框架有哪些呢？

一类是跟某种特定语言平台绑定的，另一类是与语言无关即跨语言平台的。

跟语言平台绑定的开源 RPC 框架主要有下面几种。

- `Dubbo`：国内最早开源的 RPC 框架，由阿里巴巴公司开发并于 2011 年末对外开源，仅支持 Java 语言。
- `Motan`：微博内部使用的 RPC 框架，于 2016 年对外开源，仅支持 Java 语言。
- `Tars`：腾讯内部使用的 RPC 框架，于 2017 年对外开源，仅支持 C++ 语言。
- `Spring Cloud`：国外 Pivotal 公司 2014 年对外开源的 RPC 框架，仅支持 Java 语言

而跨语言平台的开源 RPC 框架主要有以下几种。

- `gRPC`：Google 于 2015 年对外开源的跨语言 RPC 框架，支持多种语言。

  > gRPC 官方文档中文版：https://doc.oschina.net/grpc?t=58008

- `Thrift`：最初是由 Facebook 开发的内部系统跨语言的 RPC 框架，2007 年贡献给了 Apache 基金，成为 Apache 开源项目之一，支持多种语言。

如果你的业务场景仅仅局限于一种语言的话，可以选择跟语言绑定的 RPC 框架中的一种；

如果涉及多个语言平台之间的相互调用，就应该选择跨语言平台的 RPC 框架。

## 2. RPC 框架，它们具体有何区别？

### 2.1 Dubbo

先来聊聊 Dubbo，Dubbo 可以说是国内开源最早的 RPC 框架了，目前只支持 Java 语言，它的架构可以用下面这张图展示。

![2020-09-22-bJgsKx](https://image.ldbmcs.com/2020-09-22-bJgsKx.jpg)

从图中你能看到，Dubbo 的架构主要包含四个角色，其中 `Consumer` 是服务消费者，`Provider` 是服务提供者，`Registry` 是注册中心，`Monitor` 是监控系统。

具体的交互流程是 Consumer 一端通过注册中心获取到 Provider 节点后，通过 Dubbo 的客户端 SDK 与 Provider 建立连接，并发起调用。Provider 一端通过 Dubbo 的服务端 SDK 接收到 Consumer 的请求，处理后再把结果返回给 Consumer。

### 2.2 Motan

Motan 是国内另外一个比较有名的开源的 RPC 框架，同样也只支持 Java 语言实现，它的架构可以用下面这张图描述。

![2020-09-22-jFgrK0](https://image.ldbmcs.com/2020-09-22-jFgrK0.jpg)

Motan 与 Dubbo 的架构类似，都需要在 Client 端（服务消费者）和 Server 端（服务提供者）引入 SDK，其中 Motan 框架主要包含下面几个功能模块。

- `register`：用来和注册中心交互，包括注册服务、订阅服务、服务变更通知、服务心跳发送等功能。

- `protocol`：用来进行 RPC 服务的描述和 RPC 服务的配置管理，这一层还可以添加不同功能的 filter 用来完成统计、并发限制等功能。

- `serialize`：将 RPC 请求中的参数、结果等对象进行序列化与反序列化。

- `transport`：用来进行远程通信，默认使用 Netty NIO 的 TCP 长链接方式。

- `cluster`：请求时会根据不同的高可用与负载均衡策略选择一个可用的 Server 发起远程调用。

### 2.3 Tars

Tars 是腾讯根据内部多年使用微服务架构的实践，总结而成的开源项目，仅支持 C++ 语言，它的架构图如下。

![2020-09-22-F4CCVr](https://image.ldbmcs.com/2020-09-22-F4CCVr.jpg)

Tars 的架构交互主要包括以下几个流程：

- **服务发布流程**：在 web 系统上传 server 的发布包到 patch，上传成功后，在 web 上提交发布 server 请求，由 registry 服务传达到 node，然后 node 拉取 server 的发布包到本地，拉起 server 服务。
- **管理命令流程**：web 系统上的可以提交管理 server 服务命令请求，由 registry 服务传达到 node 服务，然后由 node 向 server 发送管理命令。
- **心跳上报流程**：server 服务运行后，会定期上报心跳到 node，node 然后把服务心跳信息上报到 registry 服务，由 registry 进行统一管理。
- **信息上报流程**：server 服务运行后，会定期上报统计信息到 stat，打印远程日志到 log，定期上报属性信息到 prop、上报异常信息到 notify、从 config 拉取服务配置信息。
- **client 访问 server 流程**：client 可以通过 server 的对象名 Obj 间接访问 server，client 会从 registry 上拉取 server 的路由信息（如 IP、Port 信息），然后根据具体的业务特性（同步或者异步，TCP 或者 UDP 方式）访问 server（当然 client 也可以通过 IP/Port 直接访问 server）。

### 2.4 Spring Cloud

Spring Cloud 利用 Spring Boot 特性整合了开源行业中优秀的组件，整体对外提供了一套在微服务架构中服务治理的解决方案。

只支持 Java 语言平台，它的架构图可以用下面这张图来描述。

![2020-09-22-ab6kRd](https://image.ldbmcs.com/2020-09-22-ab6kRd.jpg)

由此可见，Spring Cloud 微服务架构是由多个组件一起组成的，各个组件的交互流程如下。

- 请求统一通过 API 网关 Zuul 来访问内部服务，先经过 Token 进行安全认证。
- 通过安全认证后，网关 Zuul 从注册中心 Eureka 获取可用服务节点列表。
- 从可用服务节点中选取一个可用节点，然后把请求分发到这个节点。
- 整个请求过程中，Hystrix 组件负责处理服务超时熔断，Turbine 组件负责监控服务间的调用和熔断相关指标，Sleuth 组件负责调用链监控，ELK 负责日志分析。

### 2.5 gRPC

先来看下 gRPC，它的原理是通过 `IDL（Interface Definition Language）`文件定义服务接口的参数和返回值类型，然后通过代码生成程序生成服务端和客户端的具体实现代码，这样在 gRPC 里，客户端应用可以像调用本地对象一样调用另一台服务器上对应的方法。

![2020-09-22-UzKj8V](https://image.ldbmcs.com/2020-09-22-UzKj8V.jpg)

它的主要特性包括三个方面。

- 通信协议采用了 HTTP/2，因为 HTTP/2 提供了连接复用、双向流、服务器推送、请求优先级、首部压缩等机制。
- IDL 使用了ProtoBuf，ProtoBuf 是由 Google 开发的一种数据序列化协议，它的压缩和传输效率极高，语法也简单。
- 多语言支持，能够基于多种语言自动生成对应语言的客户端和服务端的代码。

### 2.6 Thrift

再来看下 Thrift，Thrift 是一种轻量级的跨语言 RPC 通信方案，支持多达 25 种编程语言。为了支持多种语言，跟 gRPC 一样，Thrift 也有一套自己的接口定义语言 IDL，可以通过代码生成器，生成各种编程语言的 Client 端和 Server 端的 SDK 代码，这样就保证了不同语言之间可以相互通信。它的架构图可以用下图来描述。

![2020-09-22-7Qbgpa](https://image.ldbmcs.com/2020-09-22-7Qbgpa.jpg)

从这张图上可以看出 Thrift RPC 框架的特性。

- 支持多种序列化格式：如 Binary、Compact、JSON、Multiplexed 等。
- 支持多种通信方式：如 Socket、Framed、File、Memory、zlib 等。
- 服务端支持多种处理方式：如 Simple 、Thread Pool、Non-Blocking 等。

## 3. 总结

|| 开发语言 | 服务治理 | 多种序列化 | 多种注册中心   | 管理中心 | 跨语言通讯 | 整体性能 |
| -------- | -------- | ---------- | -------------- | -------- | ---------- | -------- | ---- |
| Dubbo    | Java     | ✔          | ✔              | ✔        | ✔          | ✘        | 3    |
| Maton    | Java     | ✔          | ✔              | ✔        | ✔          | ✘        | 4    |
| Thrift   | 跨语言   | ✘          | 只支持thrift   | ✘        | ✘          | ✔        | 5    |
| Grpc     | 跨语言   | ✘          | 只支持protobuf | ✘        | ✘          | ✔        | 3    |

