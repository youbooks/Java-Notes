# Spring Integration简介

> 转载：[Spring Integration简介](https://www.tony-bro.com/posts/1578338213/index.html)

## 1. 前言

此番介绍一个不是特别热门的Spring子项目：Spring Integration，总的来说它是一种消息控制框架，核心思想源自[Enterprise Integration Pattern](https://www.enterpriseintegrationpatterns.com/)一书。构建此框架的核心目标是为企业复杂系统集成提供一种便捷的解决方案，可以说它与ESB（Enterprise Service Bus）有很多相似之处，意在通过消息将各个子系统解耦分离。

由于核心概念就是从Enterprise Integration Pattern一书中的理论出发，那么了解这本书必然会对熟悉使用此框架有很大的帮助，推荐阅读[这篇文章](https://www.cnblogs.com/loveis715/p/5185332.html)来帮助理解，本文也会直接引用一些该文章的内容。由于Spring Integration相关的第三方资料、文章比较少，因此在学习和使用过程中很有必要参考[官方文档](https://docs.spring.io/spring-integration/docs/5.1.6.RELEASE/reference/html/)，本文的结构其实也与官方文档比较类似。

## 2. 代码样例

在我对此框架进行初探和了解之时发现零星的相关文章总是描述相关的概念而没有任何示例表明这个框架到底做了什么、可以干什么，与此同时很多相关案例均是以XML配置的形式来表达，而时至今日XML配置正逐步被淘汰，因此学习的过程极其不顺。没有什么比一段简洁的可执行代码更能说明问题，因此首先贴上一段示例代码，以SpringBoot为基础、纯java配置：

```java
@Configuration
@EnableIntegration
@MessageEndpoint
public class flow {

    @Bean
    public MessageChannel beep(){
        return new QueueChannel();
    }
    
    @InboundChannelAdapter(value = "beep", poller = @Poller(fixedRate = "2000"))
    public int input(){
        Random random = new Random();
        System.out.println("Generate !!!");
        return random.nextInt();
    }
    
    @ServiceActivator(inputChannel = "beep", poller = @Poller(fixedRate = "500") )
    public void output(Message<?> message){
        System.out.println("Msg:"+ message);
    }

}
```

依赖项：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-integration</artifactId>
	</dependency>
		
	<!-- something more ... -->
</dependencies>
```

直接将上述内容置入一个SpringBoot项目，启动后即可观察到打印输出的内容。相信即使未有了解过任何Spring Integration概念也能明白上述代码做了什么，基本上就是个典型的生产者消费者模型，而其中传输的就是消息。可以看出Spring Integration做的最重要的一件事就是配置管道、传递消息。

## 3. 核心概念

从源头Enterprise Integration Pattern来说，它认为企业内部各个子服务基于消息集成，在这种方式下各个组件之间的交互将不再使用远程调用等同步方式，而是通过向目标子系统发送一个消息来令该子系统执行某个功能，在消息成功发送之后，调用方即可以开始对其它任务进行处理，而不再是同步调用过程中的等待。在使用这种处理方式时，一个系统的吞吐量可以大大增加。这种应用场景被抽象为Pipes-and-Filters模型：

![2021-04-15-1zwDLG](https://image.ldbmcs.com/2021-04-15-1zwDLG.jpg)

该模型主要由两部分组成：用来传递消息的通道（Pipe）以及用来对消息进行处理的过滤器（Filter）。这些消息通道将过滤器串联起来，而消息自身则会沿着这些通道流动。

在Spring Integration中，pipe和filter加上消息本身构成了三大基本组件：

1. Message：即消息本身，它由Payload和Header两部分组成，Payload是对任意Java对象的包装而Header则包含了消息的元数据信息，同时header也常用于Http、Mail等其他消息头部的转换。其基接口为`Message<T>`，需要注意的是通用的消息实现是不可变的。

2. Message Channel：即Pipes-and-Filters模型中的pipe，它是消息传输的载体，通常可以分为point-to-point（点对点）和publish-subscribe（发布订阅）两种行为模式。此外从通道是否保存消息的角度来说，通道还分为Pollable Channel和Subscribable Channel两种。

   - Pollable Channel：保存消息，消费者需要主动拉取消息，核心接口为`PollableChannel`。
   - Subscribable Channel：可订阅型通道，不存储消息，消费者被动通知消息，核心接口为`SubscribableChannel`。

   这种划分方式也是API接口的划分方式，不同的通道类型对消息流程的处理会有不同的表现形式。

3. Message Endpoint：即Pipes-and-Filters模型中的Filter，它是消息的消费端，通常与外部系统对接。Spring Integration提供了多种不同的EndPoint满足不同的需求。

## 4. 组件实例与API

在Spring Integration中几个基本概念都有其基本接口，下面介绍一些具体实现类以及辅助工具类。

### 4.1 Message

Message的通用实现类为`org.springframework.messaging.support.GenericMessage`，在框架执行过程中Message的头部默认会包括timestamp（时间戳）和id（UUID类型），特别注意它是不可变的。在接收消息时，参数可以直接使用`Message`接口，也可以直接使用载荷的具体类型辅以`@Payload`，在获取头部信息时，可以使用`@Header("head name")`来接受头部的具体某项值。

### 4.2 Message Channel

- Pollable Channel
  - QueueChannel：基本的拉取型通道，底层封装了一个队列。它是一个point-to-point型的通道，这也意味着即使该通道有多个消费者，每个消息只会被一个消费者获取。队列可以指定容量大小，消息生产者和消费者在发送消息、接收消息时的行为与操作阻塞队列类似。
  - PriorityChannel：与QueueChannel类似，但是底层是一个优先队列，与QueueChannel先入先出的形式不同，它可以为通道内的消息进行优先级操作。
- Subscribable Channel
  - PublishSubscribeChannel：基本的通知型通道，从名称上就可以看出他是一个publish-subscribe型通道，该通道所有的消费者都会接收到通道的每一个消息，常用于事件消息的处理。此通道继承了AbstractExecutorChannel，如果构造时传入`Executor`则在执行消费者时会使用到线程池，消费者与生产者不在同一个线程中执行，否则将在同一个线程中依次调用各个消费者。
  - DirectChannel：point-to-point型的通道但是本身不存储消息，只能有一个消费者，此通道的消费者和生产者将在同一个线程中执行任务，相当于函数调用。注意这是框架默认的通道类型。
  - ExecutorChannel：同样是一个point-to-point型的通道，与DirectChannel不同的是它继承自AbstractExecutorChannel，消费者与生产者不在同一个线程中执行。

### 4.3 Message Endpoint

以java config配置Endpoint时，通常会创建一个标有`@MessageEndpoint`注解的配置类，然后编写相关方法并配以相关的注解。注意配置类上可以辅以`@Configuration`以便同时配置通道Bean，但是Endpoint方法通常不会与`@Bean`注解同时出现，配以`@Bean`后Endpoint表现出的行为可能会不符合预期，例如上面的那一段示例代码，如果在消费者上增加`@Bean`那么每次产生的消息都将是同一个int值。

相关的注解均在org.springframework.integration.annotation包下，这些注解通常都配有inputChannel和outChannel属性，用于指定输入输出通道，如果在容器中没有找到该名称的通道Bean则会自动创建（DirectChannel类型，必须有`@EnableIntegration`）。以下是常用的几种Endpoint实现：

- Message Transformer：消息转换器，用于转换消息内容和格式。
- Message Filter：消息过滤器，没什么可多说的，返回类型为boolean即可。
- Message Router：消息路由器，用于判断将消息发往哪个通道，相关注解可以配置映射通道的名称列表，函数返回判断后决定发送的通道名称。
- Splitter：消息分割器，用于将单个消息分割为多个消息，通常在分割组合内容的载荷时使用。
- Aggregator：基本上是Splitter的反面，Aggregator将多个消息聚合为一个消息。为了聚合消息，这种Endpoint需要消息存储器、判断是否属于同一个聚合的方法等额外的配置，而且通常会接收多个通道的输出，因此在实际使用过程中通常无法直接用单个注解来完成定义，往往采用Bean的方式详细定制。
- Service Activator：通常用于连接具体的服务，作为消息最终的消费方，因此称为服务激活器。
- Channel Adapter：用于连接外部系统，可以是输入也可以是输出，也可以作为两个通道的连接适配器（例如协调Pollable Channel和Subscribable Channel）。框架提供了`@InboundChannelAdapter`这一注解来快速配置一个消息发生器，可以以指定的时间间隔产生、发送消息。

### 4.4 辅助类与工具

- @EnableIntegration：工程配置辅助注解，该注解会在Spring容器中注入一些消息系统的基本组件，主要如有如下：
  - 内建的Bean，例如errorChannel、taskScheduler等
  - 一些BeanFactoryPostProcessor，增强集成环境
  - 一些BeanPostProcessor，对一些集成的Bean进行转化、包装
  - 解析相关注解的Annotaion Processor，根据这些注解注册相关的Bean
- MessagingTemplate：通常情况下发送和接受消息需要直接调用channel的方法，为了实现非侵入性接入消息系统，提供的消息发送、接受工具，同时也能更加符合收发消息的语义。
- ChannelInterceptor：通道拦截器接口，AOP相关的概念，包含了6个接口方法，结合通道的send和receive方法，执行顺序如下：
  - channel.send 调用
  - interceptor.preSend
  - channel.doSend （存入通道）
  - interceptor.postSend
  - interceptor.afterSendCompletion
  - channel.send 返回
  - channel.receive 调用
  - interceptor.preReceive （若此方法返回false则调用receive将收到null，但仍会执行afterReceiveCompletion，此方法仅应用于PollableChannel）
  - channel.doReceive （取出消息）
  - interceptor.postReceive（此方法仅应用于PollableChannel且收到消息后才会执行）
  - interceptor.afterReceiveCompletion
  - channel.receive 返回
- Advice：Endpoint的切面工具类，提供对单个Endpoint执行切入（直接使用AOP可能切得过宽，有些通道是直接方法调用），以期对Endpoint方法进行重试、过滤等等。框架默认提供了一些Advice类，我们也可以进行自定义，通常自定义Advice可以继承`AbstractRequestHandlerAdvice`来减少额外的编码。

## 5. DSL简述

DSL即Domain Specific Language，是为了解决某一类任务而专门设计的语言，Spring Integration目前提供了一套由Java流式API和Java Config实现的配置Spring Integration的DSL。通过java dsl可以快速定义出整个消息流程，看一下官方示例代码就可以快速理解其精髓。

```java
@Configuration
@EnableIntegration
public class MyConfiguration {

    @Bean
    public AtomicInteger integerSource() {
        return new AtomicInteger();
    }

    @Bean
    public IntegrationFlow myFlow() {
        return IntegrationFlows.from(integerSource::getAndIncrement,
                                         c -> c.poller(Pollers.fixedRate(100)))
                    .channel("inputChannel")
                    .filter((Integer p) -> p > 0)
                    .transform(Object::toString)
                    .channel(MessageChannels.queue())
                    .get();
    }
}
```

如果流程没有分支、聚合且没有特殊的定制要求，那么DSL的优势非常明显。目前Spring Integration所有的组件均可以在DSL中找到对应的调用方法。所有流程都从`IntegrationFlows`开始，具体如何使用参见[官方文档](https://docs.spring.io/spring-integration/docs/5.1.6.RELEASE/reference/html/#java-dsl)，有一整个章节来描述使用的方法。

## 6. 监控与异常处理

在系统管理方面Spring Integration也提供了相关的工具来监控消息组件的使用状况以及对于Endpoint执行过程中出现异常的处理。

### 6.1 监控

开启监控后可以掌握Message Channel和Message Endpoint以及Message Source的使用状态信息。启用监控非常方便，在Java Config应用情景下，使用`@EnableIntegrationManagement`注解即可：

```java
@Configuration
@EnableIntegration
@EnableIntegrationManagement(
    defaultLoggingEnabled = "true", 
    defaultCountsEnabled = "false", 
    defaultStatsEnabled = "false", 
    countsEnabled = { "foo", "${count.patterns}" }, 
    statsEnabled = { "qux", "!*" }, 
    MetricsFactory = "myMetricsFactory") 
public static class ContextConfiguration {
...
}
```

可以看到它支持配置只对特定的组件进行监控，监控的各项指标包括总计、错误总计、执行时间等，具体可以查阅官方文档。

开启监控后容器中会注入`IntegrationManagementConfigurer`类型的Bean，在应用内可以通过它来获取到各个组件的监控对象，此外启用JMX时也可以通过MBean来获取相关信息。

此外SI还提供了一个注解`@EnableIntegrationGraphController`来将所有SI的组件的运行时监控信息通过一个Controller返回Json格式的数据，基本上包含了所有的监控内容，非常方便。

### 6.2 异常处理

Spring Integration对于错误处理的解决主要通过errorChannel来实现，它将组件中发生的错误包装为一个错误消息发送至error channel，这个通道默认类型为subscribable channel，没有指定时将提供默认的实现。这样即使发送者与接收者即使不在同一个线程（同一个线程的情况下，错误会直接抛给发送者）的情况下依然能够对错误进行处理。

框架默认会注册一个errorChannel，也可以进行自定义配置。如果需要对不同的错误分类处理，Spring Integration提供了几种方法：

1. Message头部可以指定错误通道的名称（头部名为常量`MessageHeaders.ERROR_CHANNEL`）
2. 配置一个`ErrorMessageExceptionTypeRouter` 来订阅error channel以实现对不同错误类型的分发

## 7. 扩展

为了适配不同的输入、出处，第三方系统集成以及对消息的持久化等，Spring Integration提供了多种适配包，如Http、Ftp、Mail、JDBC等等，基本涵盖了大部分常用的工具、系统。具体详见[官方文档](https://docs.spring.io/spring-integration/docs/5.1.6.RELEASE/reference/html/#spring-integration-endpoints)。

## 8. 经典样例

Spring Integration提供了一些样例代码以供学些参考，地址为https://github.com/spring-projects/spring-integration-samples。这里将最经典的一个样例拿出来解释一下，此样例非常能说明问题，同样对理解大有裨益。该样例名称为咖啡店，对咖啡店的下单流程使用Spring Integration进行了模拟，整个流程描述如图：

![2021-04-15-viBXbG](https://image.ldbmcs.com/2021-04-15-viBXbG.jpg)

简单描述一下：一个订单包含多个商品，商品分为冷饮和热饮，两种商品需要分开制作准备，全部制作完成后由服务员统一传递。以Java Config版本为示例，这其中包含了

- 信息拆分（Splitter）：将订单消息拆分为多个饮品消息

  ```java
  @MessageEndpoint
  public class OrderSplitter {
  
  	@Splitter(inputChannel="orders", outputChannel="drinks")
  	public List<OrderItem> split(Order order) {
  		return order.getItems();
  	}
  
  }
  ```

- 信息路由（Router）：将冷饮项和热饮项传递到不同的通道

  ```java
  @MessageEndpoint
  public class DrinkRouter {
  
  	@Router(inputChannel="drinks")
  	public String resolveOrderItemChannel(OrderItem orderItem) {
  		return (orderItem.isIced()) ? "coldDrinks" : "hotDrinks";
  	}
  
  }
  ```

- 信息消费（Service Activator）：制作饮品

  ```java
  @Component
  public class Barista {
  
  	private static Log logger = LogFactory.getLog(Barista.class);
  
  	private long hotDrinkDelay = 5000;
  
  	private long coldDrinkDelay = 1000;
  
  	private final AtomicInteger hotDrinkCounter = new AtomicInteger();
  
  	private final AtomicInteger coldDrinkCounter = new AtomicInteger();
  
  
  	public void setHotDrinkDelay(long hotDrinkDelay) {
  		this.hotDrinkDelay = hotDrinkDelay;
  	}
  
  	public void setColdDrinkDelay(long coldDrinkDelay) {
  		this.coldDrinkDelay = coldDrinkDelay;
  	}
  
  	@ServiceActivator(inputChannel="hotDrinkBarista", outputChannel="preparedDrinks")
  	public Drink prepareHotDrink(OrderItem orderItem) {
  		try {
  			Thread.sleep(this.hotDrinkDelay);
  			logger.info(Thread.currentThread().getName()
  					+ " prepared hot drink #" + hotDrinkCounter.incrementAndGet() + " for order #"
  					+ orderItem.getOrderNumber() + ": " + orderItem);
  			return new Drink(orderItem.getOrderNumber(), orderItem.getDrinkType(), orderItem.isIced(),
  					orderItem.getShots());
  		} catch (InterruptedException e) {
  			Thread.currentThread().interrupt();
  			return null;
  		}
  	}
  
  	@ServiceActivator(inputChannel="coldDrinkBarista", outputChannel="preparedDrinks")
  	public Drink prepareColdDrink(OrderItem orderItem) {
  		try {
  			Thread.sleep(this.coldDrinkDelay);
  			logger.info(Thread.currentThread().getName()
  					+ " prepared cold drink #" + coldDrinkCounter.incrementAndGet() + " for order #"
  					+ orderItem.getOrderNumber() + ": " + orderItem);
  			return new Drink(orderItem.getOrderNumber(), orderItem.getDrinkType(), orderItem.isIced(),
  					orderItem.getShots());
  		} catch (InterruptedException e) {
  			Thread.currentThread().interrupt();
  			return null;
  		}
  	}
  
  }
  ```

- 信息聚合（Aggregator）：将多个饮品项根据订单号聚合为一个完整的消息

  ```java
  @MessageEndpoint
  public class Waiter {
  
  	@Aggregator(inputChannel = "preparedDrinks", outputChannel = "deliveries")
  	public Delivery prepareDelivery(List<Drink> drinks) {
  		return new Delivery(drinks);
  	}
  
  	@CorrelationStrategy
  	public int correlateByOrderNumber(Drink drink) {
  		return drink.getOrderNumber();
  	}
  
  }
  ```

这个样例给出了XML配置、Java Config、Java DSL三种配置方式的示例代码，比较好地展示出了Spring Integration的特性。

## 9. 应用体会

最后分享几点应用过程中的心得体会以及一些总结。

1. 必须了解不同通道处理消息的方式以及特性，对消息执行时所处的线程有清晰的认识，否则运行时表现的行为往往会和预想的有所不同。不同于消息队列中间件可以推送消息至消费者，作为一个框架在内部分发消息时，消费者消费消息执行的方式为线程内的方法调用、提交线程池运行、主动从队列中拉取。
2. 通道一般都需要直接进行定义，对于一个消息系统来说通常不会采用默认的DirectChannel，这样就导致需要定义大量的通道Bean，适当采用MessageTemplate通过Bean名称来发送消息（需要配置`DestinationResolver`）而不是通过通道对象，这样会使代码更加简洁明了。
3. 如果要使用Aggregator则需注意聚合时Message通常以ID作为key来存储，注意同一消息是否被发往多个通道然后被Aggregator接收多次。
4. 对于现在的应用情景来说，尽管Spring Integration的目的是整合各种各样的外部系统，但是现有成熟的消息队列中间件往往是更好的选择，完备的持久化、分布式解决方案以及开箱即用的监控UI界面都是SI不具备的。此外在微服务大行其道的情况下，这种集中式策略就更显有些格格不入，但是话又说回来，微服务也是有点吹的过头，在没有达到规模以及性能瓶颈的情况下微服务只会徒增复杂度，对于逐步发展的集成项目来说，其实不妨先采用集中式架构。对于一些需要在内部分发、路由消息的单体应用倒是很适合使用SI。
5. 可以发现有很多类的包是`org.springframework.messaging`，并不在SI自己的包下，有些类甚至有重复。简单查阅来看，Spring Messaging这个包创建的时间是晚于Spring Integration的，也就是说Spring已经将消息编程模型的概念抽离了出来，以更好地适配消息中间件以及其他消息应用，SI成为了其上层的应用。而Spring Cloud Stream则是对SI的再一次封装，增加了许多Binding（绑定）的功能，可以看到SI在这之中也是控制应用内部的消息流转。
6. 本人也是在特定场景下寻找一款工具来集成内部的不同系统而发现的Spring Integration，资料确实少。编写过程中出于组件重用的考虑感觉并不如想象中的方便，流程写完了也就写死了，并没有灵活变更的感觉，不过SI还是有其用武之地的。

