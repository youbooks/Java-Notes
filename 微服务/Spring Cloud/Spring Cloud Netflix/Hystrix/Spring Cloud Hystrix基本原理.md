> 转载：[Spring Cloud Hystrix基本原理](https://www.cnblogs.com/rickiyang/p/11853315.html)

本篇学习Spring Cloud家族中的重要成员：**Hystrix**。分布式系统中一个服务可能依赖着很多其他服务，在高并发的场景下，如何保证依赖的某些服务如果出了问题不会导致主服务宕机这个问题就会变得异常重要。

针对这个问题直观想到的解决方案就是做**依赖隔离**。将不同的依赖分配到不同的调用链中，某一条链发生失败不会影响别的链。今天要说的Hystrix就提供了这样的功能。**Hystrix的作用就是处理服务依赖，帮助我们做服务治理和服务监控**。

那么Hystrix是如何解决依赖隔离呢？从官网上看到这样一段：

1. Hystrix使用命令模式`HystrixCommand`(Command)包装依赖调用逻辑，每个命令在单独线程中/信号授权下执行。
2. 可配置依赖调用**超时时间**，超时时间一般设为比99.5%平均时间略高即可.当调用超时时，直接返回或执行fallback逻辑。
3. 为每个依赖提供一个小的线程池（或信号），如果线程池已满调用将被立即拒绝，默认不采用排队，加速失败判定时间。
4. 依赖调用结果分：成功，失败（抛出异常），超时，线程拒绝，短路。 **请求失败(异常，拒绝，超时，短路)时执行fallback(降级)逻辑**。
5. 提供熔断器组件，可以自动运行或手动调用，停止当前依赖一段时间(10秒)，熔断器默认错误率阈值为50%，超过将自动运行。

另外在学习之前大家需要注意的是，Hystrix现在已经停止更新，意味着你在生产环境如果想使用的话就要考虑现有功能是否能够满足需求。另外开源界现在也有别的更优秀的服务治理组件：**Resilience4j** 和 **Sentinel**，如果你有需要可以去看一下它们现在的使用情况。当然这里并不影响我们继续学习Hystrix，毕竟作为分布式依赖隔离的鼻祖，它的设计思想还是需要吃透的。

## 1. Hystrix如何实现依赖隔离

### 1.1 命令模式

将所有请求外部系统（或者叫依赖服务）的逻辑封装到 `HystrixCommand` 或者 `HystrixObservableCommand` 对象中。

Run()方法为实现业务逻辑，这些逻辑将会在独立的线程中被执行当请求依赖服务时出现拒绝服务、超时或者短路（多个依赖服务顺序请求，前面的依赖服务请求失败，则后面的请求不会发出）时，执行该依赖服务的失败回退逻辑(Fallback)。

### 1.2 隔离策略

Hystrix 为每个依赖项维护一个小线程池（或信号量）;如果它们达到设定值（触发隔离），则发往该依赖项的请求将立即被拒绝，执行失败回退逻辑（Fallback），而不是排队。

**隔离策略分线程池隔离和信号隔离**。

#### 1.2.1 线程池隔离

第三方客户端（执行Hystrix的run()方法）会在单独的线程执行，会与调用的该任务的线程进行隔离，以此来防止调用者调用依赖所消耗的时间过长而阻塞调用者的线程。

使用线程隔离的好处：

- **应用程序可以不受失控的第三方客户端的威胁**，如果第三方客户端出现问题，可以通过降级来隔离依赖。
- 当失败的客户端服务恢复时，线程池将会被清除，应用程序也会恢复，而不至于使整个Tomcat容器出现故障。
- 如果一个客户端库的配置错误，线程池可以很快的感知这一错误（通过增加错误比例，延迟，超时，拒绝等），并可以在不影响应用程序的功能情况下来处理这些问题（可以通过动态配置来进行实时的改变）。
- 如果一个客户端服务的性能变差，可以通过改变线程池的指标（错误、延迟、超时、拒绝）来进行属性的调整，并且这些调整可以不影响其他的客户端请求。

简而言之，由线程供的隔离功能可以使客户端和应用程序优雅的处理各种变化，而不会造成中断。

**线程池的缺点**

线程最主要的缺点就是**增加了CPU的计算开销**，每个command都会在单独的线程上执行，这样的执行方式会涉及到命令的排队、调度和上下文切换。

Netflix在设计这个系统时，决定接受这个开销的代价，来换取它所提供的好处，并且认为这个开销是足够小的，不会有重大的成本或者是性能影响。

#### 1.2.2 信号隔离

信号隔离是通过限制依赖服务的并发请求数，来控制隔离开关。**信号隔离方式下，业务请求线程和执行依赖服务的线程是同一个线程**（例如Tomcat容器线程）。

### 1.3 观察者模式

- Hystrix通过观察者模式对服务进行状态监听。
- 每个任务都包含有一个对应的Metrics，所有Metrics都由一个ConcurrentHashMap来进行维护，Key是`CommandKey.name()`。
- 在任务的不同阶段会往Metrics中写入不同的信息，Metrics会对统计到的历史信息进行统计汇总，供熔断器以及Dashboard监控时使用。

### 1.4 Metrics

- Metrics内部又包含了许多内部用来管理各种状态的类，所有的状态都是由这些类管理的。
- 各种状态的内部也是用ConcurrentHashMap来进行维护的。

Metrics在统计各种状态时，运用**滑动窗口思想**进行统计的，在一个滑动窗口时间中又划分了若干个Bucket（滑动窗口时间与Bucket成整数倍关系），滑动窗口的移动是以Bucket为单位进行滑动的。

### 1.5 熔断机制

熔断机制是一种保护性机制，当系统中某个服务失败率过高时，将开启熔断器，对该服务的后续调用，直接拒绝，进行Fallback操作。

熔断所依靠的数据即是Metrics中的`HealthCount`所统计的错误率。

**如何判断是否应该开启熔断器？**

必须同时满足两个条件：

1. 请求数达到设定的阀值；
2. 请求的失败数 / 总请求数 > 错误占比阀值%。

### 1.6 降级策略

当construct()或run()执行失败时，Hystrix调用fallback执行回退逻辑，回退逻辑包含了通用的响应信息，这些响应从内存缓存中或者其他固定逻辑中得到，而不应有任何的网络依赖。

如果一定要在失败回退逻辑中包含网络请求，必须将这些网络请求包装在另一个 HystrixCommand 或 HystrixObservableCommand 中，即多次降级。

失败降级也有频率限时，如果同一fallback短时间请求过大，则会抛出拒绝异常。

### 1.7 缓存机制

同一对象的不同HystrixCommand实例，只执行一次底层的run()方法，并将第一个响应结果缓存起来，其后的请求都会从缓存返回相同的数据。

由于请求缓存位于construct()或run()方法调用之前，所以，它减少了线程的执行，消除了线程、上下文等开销。

## 2. Demo

首先引入jar包：

```xml
 <dependency>
   <groupId>com.netflix.hystrix</groupId>
   <artifactId>hystrix-core</artifactId>
   <version>1.5.18</version>
</dependency>
```

如果我们有一个被依赖的服务想要被Hystrix封装，继而使用Hystrix提供的依赖隔离服务，使用方式很简单，你只需要在你的实现类上继承 `HystrixCommand/HystrixObservableCommand`即可，重写`run()/construct()`,封装你要调用的逻辑，然后调用该类执行 `execute()/queue()/observe()/toObservable()`即可。

`HystrixCommand`用于获取只有一条返回结果的情况：

```java
package com.rickiyang.learn.service;

import com.netflix.hystrix.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import rx.Observable;

import java.util.concurrent.Future;

/**
 * 一段简单的使用HystrixCommand封装服务隔离调用的实例
 */
public class QueryOrderIdCommand extends HystrixCommand<Integer> {
    private final static Logger logger = LoggerFactory.getLogger(QueryOrderIdCommand.class);
    private String orderId = "";

    /**
     * 构造函数中封装了一些参数设置
     * @param orderId
     */
    public QueryOrderIdCommand(String orderId) {
        super(HystrixCommand.Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("orderService"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("queryByOrderId"))
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withCircuitBreakerRequestVolumeThreshold(10)//至少有10个请求，熔断器才进行错误率的计算
                        .withCircuitBreakerSleepWindowInMilliseconds(5000)//熔断器中断请求5秒后会进入半打开状态,放部分流量过去重试
                        .withCircuitBreakerErrorThresholdPercentage(50)//错误率达到50开启熔断保护
                        )
              .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("orderServicePool"))
                .andThreadPoolPropertiesDefaults(HystrixThreadPoolProperties
                        .Setter().withCoreSize(10)));
        this.orderId = orderId;
    }

    /**
     * run 方法中是你真正想要执行逻辑的地方
     * @return
     */
    @Override
    protected Integer run() {
        return 1;
    }

    @Override
    public Integer execute() {
        Thread t = Thread.currentThread();
        System.out.println("thread " + t.getName() + ": now " + orderId + " execute queue()...");
        return super.execute();
    }

    @Override
    public Future<Integer> queue() {
        Thread t = Thread.currentThread();
        System.out.println("thread " + t.getName() + ": now " + orderId + " execute queue()...");
        return super.queue();
    }

    @Override
    public Observable<Integer> observe() {
        Thread t = Thread.currentThread();
        System.out.println("thread " + t.getName() + ": now " + orderId + " execute observe()...");
        return super.observe();
    }

    @Override
    public Observable<Integer> toObservable() {
        Thread t = Thread.currentThread();
        System.out.println("thread " + t.getName() + ": now " + orderId + " execute toObservable()...");
        return super.toObservable();
    }

    /**
     * 如果发生失败在这里写回调的逻辑
     * @return
     */
    @Override
    protected Integer getFallback() {
        return -1;
    }


    /**
     * 这里是简单的模拟调用
     * @param args
     */
    public static void main(String[] args) {
        Integer r = new QueryOrderIdCommand("1").execute();
        logger.info("result:{}", r);
    }
}
```

上面的代码展示了HystrixCommand使用方式，在main函数中执行了 execute()方法，还有一个 queue，observe，toObservable 方法，其中 observe，toObservable方法是 HystrixObservableCommand 类 实现的，下面会说到。它们的区别是：

- **execute**：同步堵塞，调用了queue().get()方法，execute()执行完后，会创建一个新线程运行run()；
- **queue**：异步非堵塞，它调用了toObservable().toBlocking().toFuture()方法，queue()执行完后，会创建一个新线程运行run()。Future.get()是堵塞的，它等待run()运行完才返回结果；
- **observe()** ：异步热响应调用，它调用了toObservable().subscribe(subject)方法，observe()执行完后，会创建一个新线程运行run()。toBlocking().single()是堵塞的，需要等run()运行完才返回结果；
- **toObservable()**：异步的冷响应调用，该方法不会主动创建线程运行run()，只有当调用了toBlocking().single()或subscribe()时，才会去创建线程运行run()。

### 2.1 降级

HystrixCommand提供回退降级的方法：`getFallback`。在生产环境实现该方法的时候要注意该方法的响应要快尽量不要有网络依赖，这样才能保证降级一定是能成功。

假如说回退降级方法中还有网络依赖，那么就有失败的可能，这时候可以考虑多次降级，即在getFallback 方法调用中继续实现 新的 HystrixCommand 逻辑，保证调用不会失败。

### 2.2 熔断

上面示例中的`CircuitBreaker`设置就是跟熔断器相关的参数。

需要注意的是设置的熔断器参数是并的关系，即所有的熔断器条件都满足的情况下才会执行熔断逻辑。比如按照我们上面的设置：

整个链路请求数达到阀值（`circuitBreaker.requestVolumeThreshold`）=10，

并且请求的错误数比例大于阀值（`circuitBreaker.errorThresholdPercentage`）= 50%，则会打开熔断器。

如果熔断器处于打开状态，将会进入休眠期，在休眠期内，所有请求都将被拒绝，直接执行fallback逻辑。

根据 `Metrics` 的计算，可以判断熔断器的健康状态，从而决定是否应该关闭熔断器：

- 熔断器被打开后，根据 `circuitBreaker.sleepWindowInMilliseconds` 设置，会休眠一段时间，这段时间内的所有请求，都直接fallback；
- 休眠时间过后，Hystrix会将熔断器状态改为半开状态，然后尝试性的执行一次command，如果成功，则关闭熔断器，如果失败，继续打开熔断器，执行新的熔断周期；
- 熔断器打开后，熔断器的健康检查指标会重置，重新开始计算。

熔断器有以下几个特殊参数：

```properties
1、如果hystrix.command.default.circuitBreaker.enabled设置为false，将不使用断路器来跟踪健康状况，也不会在断路器跳闸时将其短路（即不会执行fallback）

2、如果hystrix.command.default.circuitBreaker.forceOpen设置为true，断路器将强制打开，所有请求将被拒绝，直接进入fallback

3、如果hystrix.command.default.circuitBreaker.forceClosed设置为true，断路器将强制关闭，无论错误百分比如何，都将允许请求（永远会执行run）
```

### 2.3 HystrixCommand参数设置

上面在构造函数中设置了一些Hystrix执行逻辑的参数，分别解释一下它们的含义：

- **CommandKey/CommandName** ：是一个依赖服务的command标识；
- **GroupKey**：将报告，警报，仪表板或团队/库所有权等命令组合在一起。一般可以根据服务模块或第三方客户端来分配GroupKey，一个GroupKey下可以有多个CommandKey；
- **ThreadPoolKey**：用于监视，度量标准发布，缓存和其他此类用途的HystrixThreadPool。可以一个CommandKey绑定一个ThreadPoolKey用，这样多个线程的CommandKey就会划分到同一个ThreadPoolKey。

没有定义ThreadPoolKey时，ThreadPoolKey使用GroupKey，定义了ThreadPoolKey时，则使用定义值（采用线程策略隔离的情况下）。

command在执行run()时，会创建一个线程，该线程的名称是ThreadPoolKey和序列号的组合，序列号是该线程在线程池中的创建顺序。

使用ThreadPoolKey的原因是多个command可能属于同一个所有权或逻辑功能『组』，但某些command又需要彼此隔离。

**注意：**

1. 同一个HystrixCommand对象只能执行一次run()；
2. observe()中，toBlocking().single()与subscribe()是可以共存的，因为run()是在observe()中被调用的，只调用了一次；
3. toObservable()中，toBlocking().single()与subscribe()不可共存，因为run()是在toBlocking().single()或subscribe()中被调用的；如果同时存在toBlocking().single()和subscribe()，相当于调用了2次run()，会报错。

**HystrixObservableCommand**适用于可能会有多条数据返回的场景：

```java
import com.netflix.hystrix.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import rx.Observable;
import rx.Observer;
import rx.Subscriber;
import rx.schedulers.Schedulers;

import java.util.concurrent.ExecutionException;

/**
 * 一段简单的使用HystrixCommand封装服务隔离调用的实例
 */
public class QueryOrderIdOtherCommand extends HystrixObservableCommand<String> {
    private final static Logger logger = LoggerFactory.getLogger(QueryOrderIdOtherCommand.class);
    private final String name;

    public QueryOrderIdOtherCommand(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected Observable<String> construct() {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> observer) {
                try {
                    if (!observer.isUnsubscribed()) {
                        observer.onNext("Hello");
                        observer.onNext(name + "!");
                        observer.onCompleted();
                    }
                } catch (Exception e) {
                    observer.onError(e);
                }
            }
        } ).subscribeOn(Schedulers.io());
    }

    /**
     * 这里是简单的模拟调用
     * @param args
     */
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        QueryOrderIdOtherCommand command = new QueryOrderIdOtherCommand("1");
        command.toObservable().subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {
                System.out.println("complate");
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(String s) {
                System.out.println("1");
            }
        });

        //HystrixObservableCommand不提供同步执行方法, 但是你想的话, 那么也可以用如下方式实现:
        command.observe().toBlocking().toFuture().get();
        command.toObservable().toBlocking().toFuture().get();

    }
}
```

使用 `HystrixObservableCommand` 调用逻辑被封装在contruct函数中，在这里用到了FxJava，后面会专门说一下为什么会在这里使用FxJava。

另外contruct函数中两次调用了next方法，每一次next调用表示你当前执行业务逻辑一次，那么在main函数中调用方式在上例main函数中subscribe订阅也会返回两次的调用结果，onNext会被调用两次。

使用toObservable方法默认是异步的方式调用，如果你想用同步的方式，也可以使用最后两行代码的方式进行调用。

### 2.4 隔离策略

隔离策略分 线程隔离 和 信号隔离。

`HystrixCommand` 默认采用的是线程隔离策略。当执行 `construct()` 或 `run()` 时，会创建一个线程。因为 `Hystrix` 用到了线程池，真实的流程是这样的：

1. 执行 `construct()` 或 `run()` 时，先判断线程池中是否有空闲的线程（每个Command都可以拥有自己的线程池而不会互相影响）;
2. 如果没有空闲的，则看当前线程数是否达到 `hystrix.threadpool.default.coreSize`，如果达到，则需要排队，当队列值大于 `hystrix.threadpool.default.maxQueueSize`, 会拒绝请求，执行回退逻辑，如果没有达到，则创建一个新的线程来执行；
3. 如果有空闲的，则直接从空闲的线程中取出一个来执行。

当然，我们也可以设置 `hystrix.threadpool.default.maximumSize`，动态的控制线程的大小。该参数表示一个 `HystrixCommand` 可以创建的最大线程数，当线程池中的线程在 `hystrix.threadpool.default.keepAliveTimeMinutes`时间内没有使用，则会关闭一些线程，使线程数等于在 `hystrix.threadpool.default.coreSize`。

**注意：**

必须将 `hystrix.threadpool.default.allowMaximumSizeToDivergeFromCoreSize` 设置为 `true`时，`hystrix.threadpool.default.maximumSize` 才会生效.

`hystrix.threadpool.default.coreSize` 的默认值为10，如果需要提高此值，按照以下公式计算：

```
Copy最大线程数 = QPS * 平均响应时间（单位秒）* 99% + 缓存数

举例说明：

某个接口的单台服务器QPS为10000，平均响应时间为20ms

最大线程数：10000 * 0.02 * 0.99 + 4 = 202
```

`Hystrix` 官方建议尽量将最大线程数设置的小一些，因为它是减少负载并防止资源在延迟发生时被阻塞的主要工具。线程数能设置多大，有什么影响，这个需要根据自身业务情况和实际压测结果来衡量。

### 2.5 信号隔离

`HystrixObservableCommand` 默认采用的是信号隔离。`HystrixCommand` 可以通过修改 `hystrix.command.default.execution.isolation.strategy` 参数调整为信号隔离。

- 信号隔离是对客户端请求线程的并发限制，采用信号隔离时，hystrix的线程相关配置将无效
- 当请求并发量大于 `hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests`时，请求执行fallback
- 当fallback的并发线程数大于`hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests`时，fallback将抛异常fallback execution rejected

信号隔离策略下，执行 `construct()` 或 `run()` 时，使用的是应用服务的父级线程（如Tomcat容器线程）。所以，一定要设置好并发量，有网络开销的调用，不建议使用该策略，容易导致容器线程排队堵塞，从而影响整个应用服务。

`HystrixObservableCommand` 与 `HystrixCommand` 的区别：

- 它们两个是 `Hystrix` 执行Command的两种方式；
- `HystrixCommand` 的执行封装在run()，fallback处理封装在getFallBack()；`HystrixObservableCommand` 的执行封装在contruct()，fallback处理封装在resumeWithFallback()；
- `HystrixObservableCommand`使用的信号隔离策略，所以，使用的是应用服务的父级线程调用contruct()；
- `HystrixObservableCommand` 在contruct()中可以定义多个onNext，当调用subscribe()注册成功后，将依次执行这些onNext()，后者只能在run()中返回一个值（即一个onNext）。可以理解为 `HystrixCommand` 一次只能发送单条数据返回，而`HystrixObservableCommand` 一次可以发送多条数据返回；
- 同 `HystrixCommand` 一样，`HystrixObservableCommand` 使用observe()，toBlocking().single()或subscribe()可以共存，而使用toObservable()，则不能共存。

### 2.6 线程池隔离技术和信号量隔离技术，分别在什么样的场景下去使用?

- 线程池：适合99%场景，线程池一般处理对依赖服务的网络请求的调用和访问，timeout这种问题。
- 信号量：适合不是对外部依赖的访问，而是对内部的一些比较复杂的业务逻辑的访问，但是像这种访问系统内部的代码，其实不涉及任何的网络请求。那么只要做信号量的普通限流就可以了，因为不需要去捕获timeout类似的问题，如果算法+数据结构的效率不是太高，并发量突然太高，因为这里稍微耗时一些，导致很多线程卡在这里的话是不太好的。所以进行基本的资源隔离和访问，避免内部复杂的低效率的代码，导致大量的线程被hang住。

如何修改隔离方案：

```java
super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
        .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
               .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
```

### 2.7 Hystrix cache

Hystrix支持将一个请求结果缓存起来，下一个具有相同key的请求将直接从缓存中取出结果，减少请求开销。要使用Hystrix cache功能，第一个要求是重写`getCacheKey()`，用来构造cache key；第二个要求是构建context，如果请求B要用到请求A的结果缓存，A和B必须同处一个context。通过`HystrixRequestContext.initializeContext()`和`context.shutdown()`可以构建一个context，这两条语句间的所有请求都处于同一个context。

```java
package com.rickiyang.learn.service;


import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;

/**
 * cache只有同在一个context中才生效
 * 通过HystrixRequestContext.initializeContext()初始化context，通过shutdown()关闭context
 */
public class HystrixCommand4RequestCacheTest extends HystrixCommand<Boolean> {

    private final int value;
    private final String value1;

    protected HystrixCommand4RequestCacheTest(int value, String value1) {
        super(HystrixCommandGroupKey.Factory.asKey("RequestCacheCommandGroup"));
        this.value = value;
        this.value1 = value1;
    }

    // 返回结果是cache的value
    @Override
    protected Boolean run() {
        return value == 0 || value % 2 == 0;
    }

    // 构建cache的key
    @Override
    protected String getCacheKey() {
        return String.valueOf(value) + value1;
    }

    public static void main(String[] args) {
        //这里模拟两个context的情况，在context1中执行的command2b是不能获取到command2a 中 key 的缓存的
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        HystrixCommand4RequestCacheTest command2a = new HystrixCommand4RequestCacheTest(2, "key");
        command2a.execute();
        command2a.isResponseFromCache();
        context.shutdown();


        HystrixRequestContext context1 = HystrixRequestContext.initializeContext();
        HystrixCommand4RequestCacheTest command2b = new HystrixCommand4RequestCacheTest(2, "key");
        command2b.execute();
        command2b.isResponseFromCache();
        context1.shutdown();

    }
}
```

上面这个例子模拟了两个context的情况，在context1中执行的command2b是不能获取到command2a 中 key 的缓存的。

HystrixCommandProperties配置：

```java
/* --------------统计相关------------------*/ 
// 统计滚动的时间窗口,默认:5000毫秒（取自circuitBreakerSleepWindowInMilliseconds）   
private final HystrixProperty metricsRollingStatisticalWindowInMilliseconds;   
// 统计窗口的Buckets的数量,默认:10个,每秒一个Buckets统计   
private final HystrixProperty metricsRollingStatisticalWindowBuckets; // number of buckets in the statisticalWindow   
// 是否开启监控统计功能,默认:true   
private final HystrixProperty metricsRollingPercentileEnabled;   
/* --------------熔断器相关------------------*/ 
// 熔断器在整个统计时间内是否开启的阀值，默认20。也就是在metricsRollingStatisticalWindowInMilliseconds（默认10s）内至少请求20次，熔断器才发挥起作用   
private final HystrixProperty circuitBreakerRequestVolumeThreshold;   
// 熔断时间窗口，默认:5秒.熔断器中断请求5秒后会进入半打开状态,放下一个请求进来重试，如果该请求成功就关闭熔断器，否则继续等待一个熔断时间窗口
private final HystrixProperty circuitBreakerSleepWindowInMilliseconds;   
//是否启用熔断器,默认true. 启动   
private final HystrixProperty circuitBreakerEnabled;   
//默认:50%。当出错率超过50%后熔断器启动
private final HystrixProperty circuitBreakerErrorThresholdPercentage;  
//是否强制开启熔断器阻断所有请求,默认:false,不开启。置为true时，所有请求都将被拒绝，直接到fallback 
private final HystrixProperty circuitBreakerForceOpen;   
//是否允许熔断器忽略错误,默认false, 不开启   
private final HystrixProperty circuitBreakerForceClosed; 
/* --------------信号量相关------------------*/ 
//使用信号量隔离时，命令调用最大的并发数,默认:10   
private final HystrixProperty executionIsolationSemaphoreMaxConcurrentRequests;   
//使用信号量隔离时，命令fallback(降级)调用最大的并发数,默认:10   
private final HystrixProperty fallbackIsolationSemaphoreMaxConcurrentRequests; 
/* --------------其他------------------*/ 
//使用命令调用隔离方式,默认:采用线程隔离,ExecutionIsolationStrategy.THREAD   
private final HystrixProperty executionIsolationStrategy;   
//使用线程隔离时，调用超时时间，默认:1秒   
private final HystrixProperty executionIsolationThreadTimeoutInMilliseconds;   
//线程池的key,用于决定命令在哪个线程池执行   
private final HystrixProperty executionIsolationThreadPoolKeyOverride;   
//是否开启fallback降级策略 默认:true   
private final HystrixProperty fallbackEnabled;   
// 使用线程隔离时，是否对命令执行超时的线程调用中断（Thread.interrupt()）操作.默认:true   
private final HystrixProperty executionIsolationThreadInterruptOnTimeout; 
// 是否开启请求日志,默认:true   
private final HystrixProperty requestLogEnabled;   
//是否开启请求缓存,默认:true   
private final HystrixProperty requestCacheEnabled;
```

