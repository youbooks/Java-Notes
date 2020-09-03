# Hystrix技术解析

> 转载：[Hystrix技术解析](https://www.jianshu.com/p/3e11ac385c73)

## 1. 认识Hystrix

Hystrix是Netflix开源的一款容错框架，包含常用的容错方法：**线程池隔离、信号量隔离、熔断、降级回退**。在高并发访问下，系统所依赖的服务的稳定性对系统的影响非常大，依赖有很多不可控的因素，比如网络连接变慢，资源突然繁忙，暂时不可用，服务脱机等。我们要构建稳定、可靠的分布式系统，就必须要有这样一套容错方法。 **本文将逐一分析线程池隔离、信号量隔离、熔断、降级回退这四种技术的原理与实践。**

## 2. 线程隔离

### 2.1 为什么要做线程隔离

比如我们现在有3个业务调用分别是查询订单、查询商品、查询用户，且这三个业务请求都是依赖第三方服务-订单服务、商品服务、用户服务。三个服务均是通过RPC调用。当查询订单服务，假如线程阻塞了，这个时候后续有大量的查询订单请求过来，那么容器中的线程数量则会持续增加直致CPU资源耗尽到100%，整个服务对外不可用，集群环境下就是雪崩。如下图：

![2020-08-11-YvgPYg](https://image.ldbmcs.com/2020-08-11-YvgPYg.jpg)

![2020-08-11-MXuBIP](https://image.ldbmcs.com/2020-08-11-MXuBIP.jpg)

### 2.2 线程隔离-线程池

#### 2.2.1 Hystrix是如何通过线程池实现线程隔离的

Hystrix通过命令模式，将每个类型的业务请求封装成对应的命令请求，比如查询订单-&gt;订单Command，查询商品-&gt;商品Command，查询用户-&gt;用户Command。每个类型的Command对应一个线程池。创建好的线程池是被放入到ConcurrentHashMap中，比如查询订单：

```java
final static ConcurrentHashMap<String, HystrixThreadPool> threadPools = new ConcurrentHashMap<String, HystrixThreadPool>();
threadPools.put(“hystrix-order”, new HystrixThreadPoolDefault(threadPoolKey, propertiesBuilder));
```

当第二次查询订单请求过来的时候，则可以直接从Map中获取该线程池。具体流程如下图：

![2020-08-11-UXS20Z](https://image.ldbmcs.com/2020-08-11-UXS20Z.jpg)

创建线程池中的线程的方法，查看源代码如下：

```java
public ThreadPoolExecutor getThreadPool(final HystrixThreadPoolKey threadPoolKey, HystrixProperty<Integer> corePoolSize, HystrixProperty<Integer> maximumPoolSize, HystrixProperty<Integer> keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    ThreadFactory threadFactory = null;
    if (!PlatformSpecific.isAppEngineStandardEnvironment()) {
        threadFactory = new ThreadFactory() {
            protected final AtomicInteger threadNumber = new AtomicInteger(0);

            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r, "hystrix-" + threadPoolKey.name() + "-" + threadNumber.incrementAndGet());
                thread.setDaemon(true);
                return thread;
            }

        };
    } else {
        threadFactory = PlatformSpecific.getAppEngineThreadFactory();
    }

    final int dynamicCoreSize = corePoolSize.get();
    final int dynamicMaximumSize = maximumPoolSize.get();

    if (dynamicCoreSize > dynamicMaximumSize) {
        logger.error("Hystrix ThreadPool configuration at startup for : " + threadPoolKey.name() + " is trying to set coreSize = " +
                dynamicCoreSize + " and maximumSize = " + dynamicMaximumSize + ".  Maximum size will be set to " +
                dynamicCoreSize + ", the coreSize value, since it must be equal to or greater than the coreSize value");
        return new ThreadPoolExecutor(dynamicCoreSize, dynamicCoreSize, keepAliveTime.get(), unit, workQueue, threadFactory);
    } else {
        return new ThreadPoolExecutor(dynamicCoreSize, dynamicMaximumSize, keepAliveTime.get(), unit, workQueue, threadFactory);
    }
}
```

执行Command的方式一共四种，直接看官方文档\([https://github.com/Netflix/Hystrix/wiki/How-it-Works](https://link.jianshu.com?t=https://github.com/Netflix/Hystrix/wiki/How-it-Works)\)，具体区别如下：

* `execute()`：以同步堵塞方式执行run\(\)。调用execute\(\)后，hystrix先创建一个新线程运行run\(\)，接着调用程序要在execute\(\)调用处一直堵塞着，直到run\(\)运行完成。
* `queue()`：以异步非堵塞方式执行run\(\)。调用queue\(\)就直接返回一个Future对象，同时hystrix创建一个新线程运行run\(\)，调用程序通过Future.get\(\)拿到run\(\)的返回结果，而Future.get\(\)是堵塞执行的。
* `observe()`：事件注册前执行run\(\)/construct\(\)。第一步是事件注册前，先调用observe\(\)自动触发执行run\(\)/construct\(\)（如果继承的是HystrixCommand，hystrix将创建新线程非堵塞执行run\(\)；如果继承的是HystrixObservableCommand，将以调用程序线程堵塞执行construct\(\)），第二步是从observe\(\)返回后调用程序调用subscribe\(\)完成事件注册，如果run\(\)/construct\(\)执行成功则触发onNext\(\)和onCompleted\(\)，如果执行异常则触发onError\(\)。
* `toObservable()`：事件注册后执行run\(\)/construct\(\)。第一步是事件注册前，调用toObservable\(\)就直接返回一个`Observable<String>`对象，第二步调用subscribe\(\)完成事件注册后自动触发执行run\(\)/construct\(\)（如果继承的是HystrixCommand，hystrix将创建新线程非堵塞执行run\(\)，调用程序不必等待run\(\)；如果继承的是HystrixObservableCommand，将以调用程序线程堵塞执行construct\(\)，调用程序等待construct\(\)执行完才能继续往下走），如果run\(\)/construct\(\)执行成功则触发onNext\(\)和onCompleted\(\)，如果执行异常则触发onError\(\)

**注：**

execute\(\)和queue\(\)是`HystrixCommand`中的方法，observe\(\)和toObservable\(\)是`HystrixObservableCommand`中的方法。从底层实现来讲，HystrixCommand其实也是利用Observable实现的（如果我们看Hystrix的源码的话，可以发现里面大量使用了RxJava），虽然HystrixCommand只返回单个的结果，但HystrixCommand的queue方法实际上是调用了`toObservable().toBlocking().toFuture()`，而execute方法实际上是调用了queue\(\).get\(\)。

#### 2.2.2 如何应用到实际代码中

```java
package myHystrix.threadpool;

import com.netflix.hystrix.*;
import org.junit.Test;

import java.util.List;
import java.util.concurrent.Future;

/**
 * Created by wangxindong on 2017/8/4.
 */
public class GetOrderCommand extends HystrixCommand<List> {

    OrderService orderService;

    public GetOrderCommand(String name){
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ThreadPoolTestGroup"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("testCommandKey"))
                .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(name))
                .andCommandPropertiesDefaults(
                        HystrixCommandProperties.Setter()
                                .withExecutionTimeoutInMilliseconds(5000)
                )
                .andThreadPoolPropertiesDefaults(
                        HystrixThreadPoolProperties.Setter()
                                .withMaxQueueSize(10)   //配置队列大小
                                .withCoreSize(2)    // 配置线程池里的线程数
                )
        );
    }

    @Override
    protected List run() throws Exception {
        return orderService.getOrderList();
    }

    public static class UnitTest {
        @Test
        public void testGetOrder(){
//            new GetOrderCommand("hystrix-order").execute();
            Future<List> future =new GetOrderCommand("hystrix-order").queue();
        }

    }
}
```

#### 2.2.3 线程隔离-线程池小结

执行依赖代码的线程与请求线程\(比如Tomcat线程\)分离，请求线程可以自由控制离开的时间，这也是我们通常说的异步编程，Hystrix是结合`RxJava`来实现的异步编程。通过设置线程池大小来控制并发访问量，当线程饱和的时候可以拒绝服务，防止依赖问题扩散。

![2020-08-11-kd630e](https://image.ldbmcs.com/2020-08-11-kd630e.jpg)

**线程池隔离的优点:**

1. 应用程序会被完全保护起来，即使依赖的一个服务的线程池满了，也不会影响到应用程序的其他部分。
2. 我们给应用程序引入一个新的风险较低的客户端lib的时候，如果发生问题，也是在本lib中，并不会影响到其他内容，因此我们可以大胆的引入新lib库。
3. 当依赖的一个失败的服务恢复正常时，应用程序会立即恢复正常的性能。
4. 如果我们的应用程序一些参数配置错误了，线程池的运行状况将会很快显示出来，比如延迟、超时、拒绝等。同时可以通过动态属性实时执行来处理纠正错误的参数配置。
5. 如果服务的性能有变化，从而需要调整，比如增加或者减少超时时间，更改重试次数，就可以通过线程池指标动态属性修改，而且不会影响到其他调用请求。
6. 除了隔离优势外，hystrix拥有专门的线程池可提供内置的并发功能，使得可以在同步调用之上构建异步的外观模式，这样就可以很方便的做异步编程（Hystrix引入了Rxjava异步框架）。

**尽管线程池提供了线程隔离，我们的客户端底层代码也必须要有超时设置，不能无限制的阻塞以致线程池一直饱和。**

**线程池隔离的缺点:**

**线程池的主要缺点就是它增加了计算的开销**，每个业务请求（被包装成命令）在执行的时候，会涉及到请求排队，调度和上下文切换。不过Netflix公司内部认为线程隔离开销足够小，不会产生重大的成本或性能的影响。

> The Netflix API processes 10+ billion Hystrix Command executions per day using thread isolation. Each API instance has 40+ thread-pools with 5–20 threads in each \(most are set to 10\).

Netflix API每天使用线程隔离处理10亿次Hystrix Command执行。 每个API实例都有40多个线程池，每个线程池中有5-20个线程（大多数设置为10个）。

**对于不依赖网络访问的服务，比如只依赖内存缓存这种情况下，就不适合用线程池隔离技术，而是采用信号量隔离。**

### 2.3 线程隔离-信号量

#### 2.3.1 线程池和信号量的区别

上面谈到了线程池的缺点，当我们依赖的服务是极低延迟的，比如访问内存缓存，就没有必要使用线程池的方式，那样的话开销得不偿失，而是推荐使用信号量这种方式。下面这张图说明了线程池隔离和信号量隔离的主要区别：**线程池方式下业务请求线程和执行依赖的服务的线程不是同一个线程；信号量方式下业务请求线程和执行依赖服务的线程是同一个线程**。

![2020-08-11-ocGfSM](https://image.ldbmcs.com/2020-08-11-ocGfSM.jpg)

#### 2.3.2 如何使用信号量来隔离线程

将属性`execution.isolation.strategy`设置为`SEMAPHORE` ，像这样 `ExecutionIsolationStrategy.SEMAPHORE`，则Hystrix使用信号量而不是默认的线程池来做隔离。

```java
public class CommandUsingSemaphoreIsolation extends HystrixCommand<String> {

    private final int id;

    public CommandUsingSemaphoreIsolation(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                // since we're doing work in the run() method that doesn't involve network traffic
                // and executes very fast with low risk we choose SEMAPHORE isolation
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    @Override
    protected String run() {
        // a real implementation would retrieve data from in memory data structure
        // or some other similar non-network involved work
        return "ValueFromHashMap_" + id;
    }

}
```

#### 2.3.3 线程隔离-信号量小结

信号量隔离的方式是限制了总的并发数，每一次请求过来，请求线程和调用依赖服务的线程是同一个线程，那么如果不涉及远程RPC调用（没有网络开销）则使用信号量来隔离，更为轻量，开销更小。

## 3. 熔断

### 3.1 熔断器\(Circuit Breaker\)介绍

熔断器，现实生活中有一个很好的类比，就是家庭电路中都会安装一个保险盒，当电流过大的时候保险盒里面的保险丝会自动断掉，来保护家里的各种电器及电路。Hystrix中的熔断器\(Circuit Breaker\)也是起到这样的作用，Hystrix在运行过程中会向每个commandKey对应的熔断器报告**成功、失败、超时和拒绝**的状态，熔断器维护计算统计的数据，根据这些统计的信息来确定熔断器是否打开。如果打开，后续的请求都会被截断。然后会隔一段时间默认是5s，尝试半开，放入一部分流量请求进来，相当于对依赖服务进行一次健康检查，如果恢复，熔断器关闭，随后完全恢复调用。如下图：

![2020-08-11-5oCPrM](https://image.ldbmcs.com/2020-08-11-5oCPrM.jpg)

说明，上面说的commandKey，就是在初始化的时候设置的`andCommandKey(HystrixCommandKey.Factory.asKey("testCommandKey"))`

再来看下熔断器在整个Hystrix流程图中的位置，从**步骤4**开始，如下图：

![2020-08-11-FP5aIZ](https://image.ldbmcs.com/2020-08-11-FP5aIZ.jpg)

Hystrix会检查`Circuit Breaker`的状态。如果`Circuit Breaker`的状态为开启状态，Hystrix将不会执行对应指令，而是直接进入失败处理状态（图中8 Fallback）。如果`Circuit Breaker`的状态为关闭状态，Hystrix会继续进行线程池、任务队列、信号量的检查（图中5）

### 3.2 如何使用熔断器\(Circuit Breaker\)

由于Hystrix是一个容错框架，因此我们在使用的时候，要达到熔断的目的只需配置一些参数就可以了。但我们要达到真正的效果，就必须要了解这些参数。Circuit Breaker一共包括如下6个参数。 1. `circuitBreaker.enabled` 是否启用熔断器，默认是TURE。

1. `circuitBreaker.forceOpen` 熔断器强制打开，始终保持打开状态。默认值FLASE。
2. `circuitBreaker.forceClosed` 熔断器强制关闭，始终保持关闭状态。默认值FLASE。
3. `circuitBreaker.errorThresholdPercentage` **设定错误百分比**，默认值50%，例如一段时间（10s）内有100个请求，其中有55个超时或者异常返回了，那么这段时间内的错误百分比是55%，大于了默认值50%，这种情况下触发熔断器-打开。
4. `circuitBreaker.requestVolumeThreshold` 默认值20.意思是至少有20个请求才进行`errorThresholdPercentage`错误百分比计算。比如一段时间（10s）内有19个请求全部失败了。错误百分比是100%，但熔断器不会打开，因为requestVolumeThreshold的值是20. **这个参数非常重要，熔断器是否打开首先要满足这个条件，源代码如下**:

   ```java
   // check if we are past the statisticalWindowVolumeThreshold
   if (health.getTotalRequests() < properties.circuitBreakerRequestVolumeThreshold().get()) {
       // we are not past the minimum volume threshold for the statisticalWindow so we'll return false immediately and not calculate anything
       return false;
   }

   if (health.getErrorPercentage() < properties.circuitBreakerErrorThresholdPercentage().get()) {
       return false;
   }
   ```

5. `circuitBreaker.sleepWindowInMilliseconds` 半开试探休眠时间，默认值5000ms。当熔断器开启一段时间之后比如5000ms，会尝试放过去一部分流量进行试探，确定依赖服务是否恢复。

   测试代码（模拟10次调用，错误百分比为5%的情况下，打开熔断器开关。）：

   ```java
   package myHystrix.threadpool;

   import com.netflix.hystrix.*;
   import org.junit.Test;

   import java.util.Random;

   /**
   * Created by wangxindong on 2017/8/15.
   */
   public class GetOrderCircuitBreakerCommand extends HystrixCommand<String> {

      public GetOrderCircuitBreakerCommand(String name){
          super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ThreadPoolTestGroup"))
                  .andCommandKey(HystrixCommandKey.Factory.asKey("testCommandKey"))
                  .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(name))
                  .andCommandPropertiesDefaults(
                          HystrixCommandProperties.Setter()
                                  .withCircuitBreakerEnabled(true)//默认是true，本例中为了展现该参数
                                  .withCircuitBreakerForceOpen(false)//默认是false，本例中为了展现该参数
                                  .withCircuitBreakerForceClosed(false)//默认是false，本例中为了展现该参数
                                  .withCircuitBreakerErrorThresholdPercentage(5)//(1)错误百分比超过5%
                                  .withCircuitBreakerRequestVolumeThreshold(10)//(2)10s以内调用次数10次，同时满足(1)(2)熔断器打开
                                  .withCircuitBreakerSleepWindowInMilliseconds(5000)//隔5s之后，熔断器会尝试半开(关闭)，重新放进来请求
   //                                .withExecutionTimeoutInMilliseconds(1000)
                  )
                  .andThreadPoolPropertiesDefaults(
                          HystrixThreadPoolProperties.Setter()
                                  .withMaxQueueSize(10)   //配置队列大小
                                  .withCoreSize(2)    // 配置线程池里的线程数
                  )
          );
      }

      @Override
      protected String run() throws Exception {
          Random rand = new Random();
          //模拟错误百分比(方式比较粗鲁但可以证明问题)
          if(1==rand.nextInt(2)){
   //            System.out.println("make exception");
              throw new Exception("make exception");
          }
          return "running:  ";
      }

      @Override
      protected String getFallback() {
   //        System.out.println("FAILBACK");
          return "fallback: ";
      }

      public static class UnitTest{

          @Test
          public void testCircuitBreaker() throws Exception{
              for(int i=0;i<25;i++){
                  Thread.sleep(500);
                  HystrixCommand<String> command = new GetOrderCircuitBreakerCommand("testCircuitBreaker");
                  String result = command.execute();
                  //本例子中从第11次，熔断器开始打开
                  System.out.println("call times:"+(i+1)+"   result:"+result +" isCircuitBreakerOpen: "+command.isCircuitBreakerOpen());
                  //本例子中5s以后，熔断器尝试关闭，放开新的请求进来
              }
          }
      }
   }
   ```

   测试结果：

> call times:1 result:fallback: isCircuitBreakerOpen: false call times:2 result:running: isCircuitBreakerOpen: false call times:3 result:running: isCircuitBreakerOpen: false call times:4 result:fallback: isCircuitBreakerOpen: false call times:5 result:running: isCircuitBreakerOpen: false call times:6 result:fallback: isCircuitBreakerOpen: false call times:7 result:fallback: isCircuitBreakerOpen: false call times:8 result:fallback: isCircuitBreakerOpen: false call times:9 result:fallback: isCircuitBreakerOpen: false call times:10 result:fallback: isCircuitBreakerOpen: false _熔断器打开_ call times:11 result:fallback: isCircuitBreakerOpen: true call times:12 result:fallback: isCircuitBreakerOpen: true call times:13 result:fallback: isCircuitBreakerOpen: true call times:14 result:fallback: isCircuitBreakerOpen: true call times:15 result:fallback: isCircuitBreakerOpen: true call times:16 result:fallback: isCircuitBreakerOpen: true call times:17 result:fallback: isCircuitBreakerOpen: true call times:18 result:fallback: isCircuitBreakerOpen: true call times:19 result:fallback: isCircuitBreakerOpen: true call times:20 result:fallback: isCircuitBreakerOpen: true _5s后熔断器关闭_ call times:21 result:running: isCircuitBreakerOpen: false call times:22 result:running: isCircuitBreakerOpen: false call times:23 result:fallback: isCircuitBreakerOpen: false call times:24 result:running: isCircuitBreakerOpen: false call times:25 result:running: isCircuitBreakerOpen: false

### 3.3 熔断器\(Circuit Breaker\)源代码`HystrixCircuitBreaker.java`分析

![2020-08-11-AmzthX](https://image.ldbmcs.com/2020-08-11-AmzthX.jpg)

**Factory 是一个工厂类，提供HystrixCircuitBreaker实例**

```java
public static class Factory {
        //用一个ConcurrentHashMap来保存HystrixCircuitBreaker对象
        private static ConcurrentHashMap<String, HystrixCircuitBreaker> circuitBreakersByCommand = new ConcurrentHashMap<String, HystrixCircuitBreaker>();

//Hystrix首先会检查ConcurrentHashMap中有没有对应的缓存的断路器，如果有的话直接返回。如果没有的话就会新创建一个HystrixCircuitBreaker实例，将其添加到缓存中并且返回
        public static HystrixCircuitBreaker getInstance(HystrixCommandKey key, HystrixCommandGroupKey group, HystrixCommandProperties properties, HystrixCommandMetrics metrics) {

            HystrixCircuitBreaker previouslyCached = circuitBreakersByCommand.get(key.name());
            if (previouslyCached != null) {
                return previouslyCached;
            }


            HystrixCircuitBreaker cbForCommand = circuitBreakersByCommand.putIfAbsent(key.name(), new HystrixCircuitBreakerImpl(key, group, properties, metrics));
            if (cbForCommand == null) {
                return circuitBreakersByCommand.get(key.name());
            } else {
                return cbForCommand;
            }
        }


        public static HystrixCircuitBreaker getInstance(HystrixCommandKey key) {
            return circuitBreakersByCommand.get(key.name());
        }

        static void reset() {
            circuitBreakersByCommand.clear();
        }
}
```

`HystrixCircuitBreakerImpl`是HystrixCircuitBreaker的实现，allowRequest\(\)、isOpen\(\)、markSuccess\(\)都会在HystrixCircuitBreakerImpl有默认的实现。

```java
static class HystrixCircuitBreakerImpl implements HystrixCircuitBreaker {
        private final HystrixCommandProperties properties;
        private final HystrixCommandMetrics metrics;

        /* 变量circuitOpen来代表断路器的状态，默认是关闭 */
        private AtomicBoolean circuitOpen = new AtomicBoolean(false);

        /* 变量circuitOpenedOrLastTestedTime记录着断路恢复计时器的初始时间，用于Open状态向Close状态的转换 */
        private AtomicLong circuitOpenedOrLastTestedTime = new AtomicLong();

        protected HystrixCircuitBreakerImpl(HystrixCommandKey key, HystrixCommandGroupKey commandGroup, HystrixCommandProperties properties, HystrixCommandMetrics metrics) {
            this.properties = properties;
            this.metrics = metrics;
        }

        /*用于关闭熔断器并重置统计数据*/
        public void markSuccess() {
            if (circuitOpen.get()) {
                if (circuitOpen.compareAndSet(true, false)) {
                    //win the thread race to reset metrics
                    //Unsubscribe from the current stream to reset the health counts stream.  This only affects the health counts view,
                    //and all other metric consumers are unaffected by the reset
                    metrics.resetStream();
                }
            }
        }

        @Override
        public boolean allowRequest() {
            //是否设置强制开启
            if (properties.circuitBreakerForceOpen().get()) {
                return false;
            }
            if (properties.circuitBreakerForceClosed().get()) {//是否设置强制关闭
                isOpen();
                // properties have asked us to ignore errors so we will ignore the results of isOpen and just allow all traffic through
                return true;
            }
            return !isOpen() || allowSingleTest();
        }

        public boolean allowSingleTest() {
            long timeCircuitOpenedOrWasLastTested = circuitOpenedOrLastTestedTime.get();
            //获取熔断恢复计时器记录的初始时间circuitOpenedOrLastTestedTime，然后判断以下两个条件是否同时满足：
            // 1) 熔断器的状态为开启状态(circuitOpen.get() == true)
            // 2) 当前时间与计时器初始时间之差大于计时器阈值circuitBreakerSleepWindowInMilliseconds(默认为 5 秒)
            //如果同时满足的话，表示可以从Open状态向Close状态转换。Hystrix会通过CAS操作将circuitOpenedOrLastTestedTime设为当前时间，并返回true。如果不同时满足，返回false，代表熔断器关闭或者计时器时间未到。
            if (circuitOpen.get() && System.currentTimeMillis() > timeCircuitOpenedOrWasLastTested + properties.circuitBreakerSleepWindowInMilliseconds().get()) {
                // We push the 'circuitOpenedTime' ahead by 'sleepWindow' since we have allowed one request to try.
                // If it succeeds the circuit will be closed, otherwise another singleTest will be allowed at the end of the 'sleepWindow'.
                if (circuitOpenedOrLastTestedTime.compareAndSet(timeCircuitOpenedOrWasLastTested, System.currentTimeMillis())) {
                    // if this returns true that means we set the time so we'll return true to allow the singleTest
                    // if it returned false it means another thread raced us and allowed the singleTest before we did
                    return true;
                }
            }
            return false;
        }

        @Override
        public boolean isOpen() {
            if (circuitOpen.get()) {//获取断路器的状态
                // if we're open we immediately return true and don't bother attempting to 'close' ourself as that is left to allowSingleTest and a subsequent successful test to close
                return true;
            }

            // Metrics数据中获取HealthCounts对象
            HealthCounts health = metrics.getHealthCounts();

            // 检查对应的请求总数(totalCount)是否小于属性中的请求容量阈值circuitBreakerRequestVolumeThreshold,默认20，如果是的话表示熔断器可以保持关闭状态，返回false
            if (health.getTotalRequests() < properties.circuitBreakerRequestVolumeThreshold().get()) {

                return false;
            }

            //不满足请求总数条件，就再检查错误比率(errorPercentage)是否小于属性中的错误百分比阈值(circuitBreakerErrorThresholdPercentage，默认 50),如果是的话表示断路器可以保持关闭状态，返回 false
            if (health.getErrorPercentage() < properties.circuitBreakerErrorThresholdPercentage().get()) {
                return false;
            } else {
                // 如果超过阈值，Hystrix会判定服务的某些地方出现了问题，因此通过CAS操作将断路器设为开启状态，并记录此时的系统时间作为定时器初始时间，最后返回 true
                if (circuitOpen.compareAndSet(false, true)) {
                    circuitOpenedOrLastTestedTime.set(System.currentTimeMillis());
                    return true;
                } else {
                    return true;
                }
            }
        }

    }
```

### 3.4 熔断器小结

每个熔断器默认维护10个bucket,每秒一个bucket,每个blucket记录成功,失败,超时,拒绝的状态，默认错误超过50%且10秒内超过20个请求进行中断拦截。下图显示HystrixCommand或`HystrixObservableCommand`如何与HystrixCircuitBreaker及其逻辑和决策流程进行交互，包括计数器在断路器中的行为。

## 4. 回退降级

### 4.1 降级

所谓降级，就是指在在Hystrix执行非核心链路功能失败的情况下，我们如何处理，比如我们返回默认值等。如果我们要回退或者降级处理，代码上需要实现`HystrixCommand.getFallback()`方法或者是`HystrixObservableCommand. HystrixObservableCommand()`。

```java
public class CommandHelloFailure extends HystrixCommand<String> {

    private final String name;

    public CommandHelloFailure(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        throw new RuntimeException("this command always fails");
    }

    @Override
    protected String getFallback() {
        return "Hello Failure " + name + "!";
    }
}
```

### 4.2 Hystrix的降级回退方式

Hystrix一共有如下几种降级回退模式：

#### 4.2.1 Fail Fast 快速失败

```java
 @Override
    protected String run() {
        if (throwException) {
            throw new RuntimeException("failure from CommandThatFailsFast");
        } else {
            return "success";
        }
    }
```

如果我们实现的是HystrixObservableCommand.java则 重写`resumeWithFallback`方法。

```java
@Override
    protected Observable<String> resumeWithFallback() {
        if (throwException) {
            return Observable.error(new Throwable("failure from CommandThatFailsFast"));
        } else {
            return Observable.just("success");
        }
    }
```

#### 4.2.2 Fail Silent 无声失败

返回null，空Map，空List。

![2020-08-11-oirDOp](https://image.ldbmcs.com/2020-08-11-oirDOp.jpg)

```java
@Override
protected String getFallback() {
    return null;
}
@Override
protected List<String> getFallback() {
  return Collections.emptyList();
}
@Override
protected Observable<String> resumeWithFallback() {
  return Observable.empty();
}
```

#### 4.2.3 Fallback: Static 返回默认值

回退的时候返回静态嵌入代码中的默认值，这样就不会导致功能以Fail Silent的方式被清楚，也就是用户看不到任何功能了。而是按照一个默认的方式显示。

```java
@Override
protected Boolean getFallback() {
  return true;
}
@Override
protected Observable<Boolean> resumeWithFallback() {
  return Observable.just( true );
}
```

#### 4.2.4 Fallback: Stubbed 自己组装一个值返回

当我们执行返回的结果是一个包含多个字段的对象时，则会以Stubbed 的方式回退。Stubbed 值我们建议在实例化Command的时候就设置好一个值。以countryCodeFromGeoLookup为例，countryCodeFromGeoLookup的值，是在我们调用的时候就注册进来初始化好的。`CommandWithStubbedFallback command = new CommandWithStubbedFallback(1234, "china");`主要代码如下：

```java
public class CommandWithStubbedFallback extends HystrixCommand<UserAccount> {

protected CommandWithStubbedFallback(int customerId, String countryCodeFromGeoLookup) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.customerId = customerId;
        this.countryCodeFromGeoLookup = countryCodeFromGeoLookup;
}
  @Override
  protected UserAccount getFallback() {
    /**
         * Return stubbed fallback with some static defaults, placeholders,
         * and an injected value 'countryCodeFromGeoLookup' that we'll use
         * instead of what we would have retrieved from the remote service.
         */
    return new UserAccount(customerId, "Unknown Name",
                           countryCodeFromGeoLookup, true, true, false);
  }
```

#### 4.2.5 Fallback: Cache via Network 利用远程缓存

通过远程缓存的方式。在失败的情况下再发起一次remote请求，不过这次请求的是一个缓存比如redis。**由于是又发起一起远程调用，所以会重新封装一次Command，这个时候要注意，执行fallback的线程一定要跟主线程区分开，也就是重新命名一个ThreadPoolKey。**

![2020-08-11-Ej2dVs](https://image.ldbmcs.com/2020-08-11-Ej2dVs.jpg)

```java
public class CommandWithFallbackViaNetwork extends HystrixCommand<String> {
    private final int id;

    protected CommandWithFallbackViaNetwork(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueCommand")));
        this.id = id;
    }

    @Override
    protected String run() {
        //        RemoteServiceXClient.getValue(id);
        throw new RuntimeException("force failure for example");
    }

    @Override
    protected String getFallback() {
        return new FallbackViaNetwork(id).execute();
    }

    private static class FallbackViaNetwork extends HystrixCommand<String> {
        private final int id;

        public FallbackViaNetwork(int id) {
            super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueFallbackCommand"))
                    // use a different threadpool for the fallback command
                    // so saturating the RemoteServiceX pool won't prevent
                    // fallbacks from executing
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("RemoteServiceXFallback")));
            this.id = id;
        }

        @Override
        protected String run() {
            MemCacheClient.getValue(id);
        }

        @Override
        protected String getFallback() {
            // the fallback also failed
            // so this fallback-of-a-fallback will 
            // fail silently and return null
            return null;
        }
    }
}
```

#### 4.2.6 Primary + Secondary with Fallback 主次方式回退（主要和次要）

这个有点类似我们日常开发中需要上线一个新功能，但为了防止新功能上线失败可以回退到老的代码，我们会做一个开关比如使用zookeeper做一个配置开关，可以动态切换到老代码功能。那么Hystrix它是使用通过一个配置来在两个command中进行切换。

![2020-08-11-Aqg8zn](https://image.ldbmcs.com/2020-08-11-Aqg8zn.jpg)

```java
/**
 * Sample {@link HystrixCommand} pattern using a semaphore-isolated command
 * that conditionally invokes thread-isolated commands.
 */
public class CommandFacadeWithPrimarySecondary extends HystrixCommand<String> {

    private final static DynamicBooleanProperty usePrimary = DynamicPropertyFactory.getInstance().getBooleanProperty("primarySecondary.usePrimary", true);

    private final int id;

    public CommandFacadeWithPrimarySecondary(int id) {
        super(Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("PrimarySecondaryCommand"))
                .andCommandPropertiesDefaults(
                        // we want to default to semaphore-isolation since this wraps
                        // 2 others commands that are already thread isolated
                        // 采用信号量的隔离方式
                        HystrixCommandProperties.Setter()
                                .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    //通过DynamicPropertyFactory来路由到不同的command
    @Override
    protected String run() {
        if (usePrimary.get()) {
            return new PrimaryCommand(id).execute();
        } else {
            return new SecondaryCommand(id).execute();
        }
    }

    @Override
    protected String getFallback() {
        return "static-fallback-" + id;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }

    private static class PrimaryCommand extends HystrixCommand<String> {

        private final int id;

        private PrimaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("PrimaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("PrimaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 600ms timeout for primary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(600)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform expensive 'primary' service call
            return "responseFromPrimary-" + id;
        }

    }

    private static class SecondaryCommand extends HystrixCommand<String> {

        private final int id;

        private SecondaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("SecondaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("SecondaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 100ms timeout for secondary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(100)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform fast 'secondary' service call
            return "responseFromSecondary-" + id;
        }

    }

    public static class UnitTest {

        @Test
        public void testPrimary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                //将属性"primarySecondary.usePrimary"设置为true，则走PrimaryCommand；设置为false，则走SecondaryCommand
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", true);
                assertEquals("responseFromPrimary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }

        @Test
        public void testSecondary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                //将属性"primarySecondary.usePrimary"设置为true，则走PrimaryCommand；设置为false，则走SecondaryCommand
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", false);
                assertEquals("responseFromSecondary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }
    }
}
```

### 4.3 回退降级小结

降级的处理方式，返回默认值，返回缓存里面的值（包括远程缓存比如redis和本地缓存比如jvmcache）。 但回退的处理方式也有不适合的场景：

1. 写操作
2. 批处理
3. 计算

以上几种情况如果失败，则程序就要将错误返回给调用者。

## 5. 总结

Hystrix为我们提供了一套线上系统容错的技术实践方法，我们通过在系统中引入Hystrix的jar包可以很方便的使用线程隔离、熔断、回退等技术。同时它还提供了监控页面配置，方便我们管理查看每个接口的调用情况。像spring cloud这种微服务构建模式中也引入了Hystrix，我们可以放心使用Hystrix的线程隔离技术，来防止雪崩这种可怕的致命性线上故障。

