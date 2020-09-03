# Sentinel: 分布式系统的流量防卫兵

> 转载：[Sentinel: 分布式系统的流量防卫兵](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

## 1. Sentinel 是什么？

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。**Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。**

Sentinel 具有以下特征:

* **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
* **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 的主要特性：

![2020-08-11-1dfpl9](https://image.ldbmcs.com/2020-08-11-1dfpl9.jpg)

Sentinel 的开源生态：

![2020-08-11-UYOBho](https://image.ldbmcs.com/2020-08-11-UYOBho.jpg)

Sentinel 分为两个部分:

* 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
* 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

## 2. Quick Start

### 1.1 公网 Demo

如果希望最快的了解 Sentinel 在做什么，我们可以通过 [Sentinel 新手指南](https://github.com/alibaba/Sentinel/wiki/新手指南#公网-demo) 来运行一个例子，并且能在云上控制台上看到最直观的监控和流控效果等。

### 1.2 手动接入 Sentinel 以及控制台

下面的例子将展示应用如何三步接入 Sentinel。同时，Sentinel 也提供所见即所得的控制台，可以实时监控资源以及管理规则。

#### 1.2.1 STEP 1. 在应用中引入Sentinel Jar包

如果应用使用 pom 工程，则在 `pom.xml` 文件中加入以下代码即可：

```markup
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.7.2</version>
</dependency>
```

> 注意: 从 Sentinel 1.5.0 开始仅支持 JDK 1.7 或者以上版本。Sentinel 1.5.0 之前的版本最低支持 JDK 1.6。

如果您未使用依赖管理工具，请到 [Maven Center Repository](https://mvnrepository.com/artifact/com.alibaba.csp/sentinel-core) 直接下载 JAR 包。

#### 1.2.2 STEP 2. 定义资源

接下来，我们把需要控制流量的代码用 Sentinel API `SphU.entry("HelloWorld")` 和 `entry.exit()` 包围起来即可。在下面的例子中，我们将 `System.out.println("hello world");` 这端代码作为资源，用 API 包围起来（埋点）。参考代码如下:

```java
public static void main(String[] args) {
    initFlowRules();
    while (true) {
        Entry entry = null;
        try {
          entry = SphU.entry("HelloWorld");
          /*您的业务逻辑 - 开始*/
          System.out.println("hello world");
          /*您的业务逻辑 - 结束*/
        } catch (BlockException e1) {
            /*流控逻辑处理 - 开始*/
            System.out.println("block!");
            /*流控逻辑处理 - 结束*/
        } finally {
           if (entry != null) {
               entry.exit();
           }
        }
    }
}
```

完成以上两步后，代码端的改造就完成了。当然，我们也提供了 [注解支持模块](https://github.com/alibaba/Sentinel/wiki/注解支持)，可以以低侵入性的方式定义资源。

#### 1.2.3 STEP 3. 定义规则

接下来，通过规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源 `HelloWorld` 每秒最多只能通过 20 个请求。

```java
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

完成上面 3 步，Sentinel 就能够正常工作了。更多的信息可以参考 [使用文档](https://github.com/alibaba/Sentinel/wiki/如何使用)。

#### 1.2.4 STEP 4. 检查效果

Demo 运行之后，我们可以在日志 `~/logs/csp/${appName}-metrics.log.xxx` 里看到下面的输出:

```text
|--timestamp-|------date time----|-resource-|p |block|s |e|rt
1529998904000|2018-06-26 15:41:44|HelloWorld|20|0    |20|0|0
1529998905000|2018-06-26 15:41:45|HelloWorld|20|5579 |20|0|728
1529998906000|2018-06-26 15:41:46|HelloWorld|20|15698|20|0|0
1529998907000|2018-06-26 15:41:47|HelloWorld|20|19262|20|0|0
1529998908000|2018-06-26 15:41:48|HelloWorld|20|19502|20|0|0
1529998909000|2018-06-26 15:41:49|HelloWorld|20|18386|20|0|0
```

其中 `p` 代表通过的请求, `block` 代表被阻止的请求, `s` 代表成功执行完成的请求个数, `e` 代表用户自定义的异常, `rt` 代表平均响应时长。

可以看到，这个程序每秒稳定输出 "hello world" 20 次，和规则中预先设定的阈值是一样的。

更详细的说明可以参考: [如何使用](https://github.com/alibaba/Sentinel/wiki/如何使用)

更多的例子可以参考: [Sentinel Examples](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo)

#### 1.2.5 STEP 5. 启动 Sentinel 控制台

您可以参考 [Sentinel 控制台文档](https://github.com/alibaba/Sentinel/wiki/控制台) 启动控制台，可以实时监控各个资源的运行情况，并且可以实时地修改限流规则。

![2020-08-11-1CfpcZ](https://image.ldbmcs.com/2020-08-11-1CfpcZ.jpg)

## 3. 详细文档

请移步 [Wiki](https://github.com/alibaba/Sentinel/wiki/主页)，查阅详细的文档、示例以及[使用说明](https://github.com/alibaba/Sentinel/wiki/如何使用)。若您希望从其它熔断降级组件（如 Hystrix）迁移或进行功能对比，可以参考 [迁移指南](https://github.com/alibaba/Sentinel/wiki/Guideline:-从-Hystrix-迁移到-Sentinel)。

Please refer to [README](https://github.com/alibaba/Sentinel) for README in English。

与 Sentinel 相关的生态（包括社区用户实现的扩展、整合、示例以及文章）可以参见 [Awesome Sentinel](https://github.com/alibaba/Sentinel/blob/master/doc/awesome-sentinel.md)，欢迎补充！

如果您正在使用 Sentinel，欢迎在 [Wanted: Who is using Sentinel](https://github.com/alibaba/Sentinel/issues/18) 留言告诉我们您的使用场景，以便我们更好地去改进。

