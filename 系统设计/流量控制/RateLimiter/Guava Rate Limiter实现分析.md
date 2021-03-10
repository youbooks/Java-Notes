转载：[RateLimiter](https://www.cnblogs.com/UniqueColor/p/11534184.html)

Google开源工具包Guava提供了限流工具类`RateLimiter`，该类基于**令牌桶算法**实现流量限制，使用十分方便，而且十分高效。

## 1. RateLimiter使用

```java
public void testAcquire() {
      RateLimiter limiter = RateLimiter.create(1);

      for(int i = 1; i < 10; i = i + 2 ) {
          double waitTime = limiter.acquire(i);
          System.out.println("cutTime=" + System.currentTimeMillis() + " acq:" + i + " waitTime:" + waitTime);
      }
  }
```

结果：

```java
cutTime=1535439657427 acq:1 waitTime:0.0
cutTime=1535439658431 acq:3 waitTime:0.997045
cutTime=1535439661429 acq:5 waitTime:2.993028
cutTime=1535439666426 acq:7 waitTime:4.995625
cutTime=1535439673426 acq:9 waitTime:6.999223
```

首先通过`RateLimiter.create(1);`创建一个限流器，参数代表每秒生成的令牌数，通过`limiter.acquire(i);`来以阻塞的方式获取令牌，当然也可以通过`tryAcquire(int permits, long timeout, TimeUnit unit)`来设置等待超时时间的方式获取令牌，如果超timeout为0，则代表非阻塞，获取不到立即返回。

从输出来看，RateLimiter支持预消费，比如在acquire(5)时，等待时间是3秒，是上一个获取令牌时预消费了3个两排，固需要等待3*1秒，然后又预消费了5个令牌，以此类推。

RateLimiter通过限制后面请求的等待时间，来支持一定程度的突发请求(预消费)。

源码注释中的一个例子,比如我们有很多任务需要执行,但是我们不希望每秒超过两个任务执行,那么我们就可以使用RateLimiter：

```java
final RateLimiter rateLimiter = RateLimiter.create(2.0);
void submitTasks(List<Runnable> tasks, Executor executor) {
    for (Runnable task : tasks) {
        rateLimiter.acquire(); // may wait
        executor.execute(task);
    }
}
```

另外一个例子，假如我们会产生一个数据流，然后我们想以每秒5kb的速度发送出去。我们可以每获取一个令牌(permit)就发送一个byte的数据,这样我们就可以通过一个每秒5000个令牌的RateLimiter来实现:

```java
final RateLimiter rateLimiter = RateLimiter.create(5000.0);
void submitPacket(byte[] packet) {
    rateLimiter.acquire(packet.length);
    networkService.send(packet);
}
```

另外，我们也可以使用非阻塞的形式达到降级运行的目的，即使用非阻塞的tryAcquire()方法:

```java
if(limiter.tryAcquire()) { //未请求到limiter则立即返回false
    doSomething();
}else{
    doSomethingElse();
}
```

## 2. 设计思路

考虑一下RateLimiter是如何设计的，并且为什么要这样设计。

RateLimiter的主要功能就是提供一个稳定的速率，实现方式就是通过限制请求流入的速度，比如计算请求等待合适的时间阈值。

实现QPS速率的最简单的方式就是记住上一次请求的最后授权时间，然后保证1/QPS秒内不允许请求进入。比如QPS=5，如果我们保证最后一个被授权请求之后的200ms的时间内没有请求被授权，那么我们就达到了预期的速率。如果一个请求现在过来但是最后一个被授权请求是在100ms之前，那么我们就要求当前这个请求等待100ms。按照这个思路,请求15个新令牌(许可证)就需要3秒。

有一点很重要：上面这个设计思路的RateLimiter记忆非常的浅，它的脑容量非常的小，只记得上一次被授权的请求的时间。如果RateLimiter的一个被授权请求q之前很长一段时间没有被使用会怎么样？这个RateLimiter会立马忘记过去这一段时间的利用不足，而只记得刚刚的请求q.

过去一段时间的利用不足意味着有过剩的资源是可以利用的.这种情况下,RateLimiter应该加把劲(speed up for a while)将这些过剩的资源利用起来.比如在向网络中发生数据的场景(限流),过去一段时间的利用不足可能意味着网卡缓冲区是空的,这种场景下,我们是可以加速发送来将这些过程的资源利用起来.

另一方面,过去一段时间的利用不足可能意味着处理请求的服务器对即将到来的请求是准备不足的(less ready for future requests),比如因为很长一段时间没有请求当前服务器的cache是陈旧的,进而导致即将到来的请求会触发一个昂贵的操作(比如重新刷新全量的缓存).

为了处理这种情况,RateLimiter中增加了一个维度的信息,就是过去一段时间的利用不足(past underutilization),代码中使用storedPermits变量表示.当没有利用不足这个变量为0,最大能达到maxStoredPermits(maxStoredPermits表示完全没有利用).因此,请求的令牌可能从两个地方来:

```
1.过去剩余的令牌(stored permits, 可能没有)
2.现有的令牌(fresh permits,当前这段时间还没用完的令牌)
```

我们将通过一个例子来解释它是如何工作的:

对一个每秒产生一个令牌的RateLimiter,每有一个没有使用令牌的一秒,我们就将storedPermits加1,如果RateLimiter在10秒都没有使用,则storedPermits变成10.0.这个时候,一个请求到来并请求三个令牌(acquire(3)),我们将从storedPermits中的令牌为其服务,storedPermits变为7.0.这个请求之后立马又有一个请求到来并请求10个令牌,我们将从storedPermits剩余的7个令牌给这个请求,剩下还需要三个令牌,我们将从RateLimiter新产生的令牌中获取.我们已经知道,RateLimiter每秒新产生1个令牌,就是说上面这个请求还需要的3个请求就要求其等待3秒.

想象一个RateLimiter每秒产生一个令牌,现在完全没有使用(处于初始状态),限制一个昂贵的请求acquire(100)过来.如果我们选择让这个请求等待100秒再允许其执行,这显然很荒谬.我们为什么什么也不做而只是傻傻的等待100秒,一个更好的做法是允许这个请求立即执行(和acquire(1)没有区别),然后将随后到来的请求推迟到正确的时间点.这种策略,我们允许这个昂贵的任务立即执行,并将随后到来的请求推迟100秒.这种策略就是让任务的执行和等待同时进行.

一个重要的结论:RateLimiter不会记最后一个请求,而是即下一个请求允许执行的时间.这也可以很直白的告诉我们到达下一个调度时间点的时间间隔.然后定一个一段时间未使用的Ratelimiter也很简单:下一个调度时间点已经过去,这个时间点和现在时间的差就是Ratelimiter多久没有被使用,我们会将这一段时间翻译成storedPermits.所有,如果每秒钟产生一个令牌(rate==1),并且正好每秒来一个请求,那么storedPermits就不会增长.

## 3. 设计原理

Guava有两种限流模式，一种为稳定模式(SmoothBursty:令牌生成速度恒定)，一种为渐进模式(SmoothWarmingUp:令牌生成速度缓慢提升直到维持在一个稳定值) 两种模式实现思路类似，主要区别在等待时间的计算上，本篇重点介绍SmoothBursty

### 3.1 RateLimiter的创建

通过调用RateLimiter的`create`接口来创建实例，实际是调用的`SmoothBuisty`稳定模式创建的实例。

```java
public static RateLimiter create(double permitsPerSecond) {
    return create(permitsPerSecond, SleepingStopwatch.createFromSystemTimer());
}

static RateLimiter create(double permitsPerSecond, SleepingStopwatch stopwatch) {
    RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0 /* maxBurstSeconds */);
    rateLimiter.setRate(permitsPerSecond);
    return rateLimiter;
}
```

`SmoothBursty`中的两个构造参数含义：

- SleepingStopwatch：guava中的一个时钟类实例，会通过这个来计算时间及令牌
- maxBurstSeconds：官方解释，在ReteLimiter未使用时，最多保存几秒的令牌，默认是1

在解析SmoothBursty原理前，重点解释下SmoothBursty中几个属性的含义：

```java
/**
 * The work (permits) of how many seconds can be saved up if this RateLimiter is unused?
 * 在RateLimiter未使用时，最多存储几秒的令牌
 * */
 final double maxBurstSeconds;
 

/**
 * The currently stored permits.
 * 当前存储令牌数
 */
double storedPermits;

/**
 * The maximum number of stored permits.
 * 最大存储令牌数 = maxBurstSeconds * stableIntervalMicros(见下文)
 */
double maxPermits;

/**
 * The interval between two unit requests, at our stable rate. E.g., a stable rate of 5 permits
 * per second has a stable interval of 200ms.
 * 添加令牌时间间隔 = SECONDS.toMicros(1L) / permitsPerSecond；(1秒/每秒的令牌数)
 */
double stableIntervalMicros;

/**
 * The time when the next request (no matter its size) will be granted. After granting a request,
 * this is pushed further in the future. Large requests push this further than small requests.
 * 下一次请求可以获取令牌的起始时间
 * 由于RateLimiter允许预消费，上次请求预消费令牌后
 * 下次请求需要等待相应的时间到nextFreeTicketMicros时刻才可以获取令牌
 */
private long nextFreeTicketMicros = 0L; // could be either in the past or future
```

关键函数

- setRate

  ```java
  public final void setRate(double permitsPerSecond) {
    checkArgument(
        permitsPerSecond > 0.0 && !Double.isNaN(permitsPerSecond), "rate must be positive");
    synchronized (mutex()) {
      doSetRate(permitsPerSecond, stopwatch.readMicros());
    }
  }
  ```

  通过这个接口设置令牌通每秒生成令牌的数量，内部时间通过调用`SmoothRateLimiter`的`doSetRate`来实现

- doSetRate

  ```java
  final void doSetRate(double permitsPerSecond, long nowMicros) {
      resync(nowMicros);
      double stableIntervalMicros = SECONDS.toMicros(1L) / permitsPerSecond;
      this.stableIntervalMicros = stableIntervalMicros;
      doSetRate(permitsPerSecond, stableIntervalMicros);
  }
  ```

  这里先通过调用`resync`生成令牌以及更新下一期令牌生成时间，然后更新stableIntervalMicros，最后又调用了`SmoothBursty`的`doSetRate`

- resync

  ```java
  /**
   * Updates {@code storedPermits} and {@code nextFreeTicketMicros} based on the current time.
   * 基于当前时间，更新下一次请求令牌的时间，以及当前存储的令牌(可以理解为生成令牌)
   */
  void resync(long nowMicros) {
      // if nextFreeTicket is in the past, resync to now
      if (nowMicros > nextFreeTicketMicros) {
        double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
        storedPermits = min(maxPermits, storedPermits + newPermits);
        nextFreeTicketMicros = nowMicros;
      }
  }
  ```

  根据令牌桶算法，桶中的令牌是持续生成存放的，有请求时需要先从桶中拿到令牌才能开始执行，谁来持续生成令牌存放呢？

  一种解法是，开启一个定时任务，由定时任务持续生成令牌。这样的问题在于会极大的消耗系统资源，如，某接口需要分别对每个用户做访问频率限制，假设系统中存在6W用户，则至多需要开启6W个定时任务来维持每个桶中的令牌数，这样的开销是巨大的。

  另一种解法则是延迟计算，如上`resync`函数。该函数会在每次获取令牌之前调用，其实现思路为，若当前时间晚于nextFreeTicketMicros，则计算该段时间内可以生成多少令牌，将生成的令牌加入令牌桶中并更新数据。这样一来，只需要在获取令牌时计算一次即可。

- SmoothBursty的doSetRate

  ```java
  @Override
  void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
    double oldMaxPermits = this.maxPermits;
    maxPermits = maxBurstSeconds * permitsPerSecond;
    if (oldMaxPermits == Double.POSITIVE_INFINITY) {
      // if we don't special-case this, we would get storedPermits == NaN, below
      // Double.POSITIVE_INFINITY 代表无穷啊
      storedPermits = maxPermits;
    } else {
      storedPermits =
          (oldMaxPermits == 0.0)
              ? 0.0 // initial state
              : storedPermits * maxPermits / oldMaxPermits;
    }
  }
  ```

  桶中可存放的最大令牌数由maxBurstSeconds计算而来，其含义为最大存储maxBurstSeconds秒生成的令牌。
  该参数的作用在于，可以更为灵活地控制流量。如，某些接口限制为300次/20秒，某些接口限制为50次/45秒等。也就是流量不局限于qps。

## 4. RateLimiter几个常用接口分析

**在了解以上概念后，就非常容易理解RateLimiter暴露出来的接口**

```java
@CanIgnoreReturnValue
public double acquire() {
  return acquire(1);
}

/**
* 获取令牌，返回阻塞的时间
**/
@CanIgnoreReturnValue
public double acquire(int permits) {
  long microsToWait = reserve(permits);
  stopwatch.sleepMicrosUninterruptibly(microsToWait);
  return 1.0 * microsToWait / SECONDS.toMicros(1L);
}

final long reserve(int permits) {
  checkPermits(permits);
  synchronized (mutex()) {
    return reserveAndGetWaitLength(permits, stopwatch.readMicros());
  }
}
```

`acquire`函数主要用于获取permits个令牌，并计算需要等待多长时间，进而挂起等待，并将该值返回，主要通过`reserve`返回需要等待的时间，`reserve`中通过调用`reserveAndGetWaitLength`获取等待时间。

```java
/**
 * Reserves next ticket and returns the wait time that the caller must wait for.
 *
 * @return the required wait time, never negative
 */
final long reserveAndGetWaitLength(int permits, long nowMicros) {
  long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
  return max(momentAvailable - nowMicros, 0);
}
```

最后调用了reserveEarliestAvailable。

```java
@Override
final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
  resync(nowMicros);
  long returnValue = nextFreeTicketMicros;
  double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
  double freshPermits = requiredPermits - storedPermitsToSpend;
  long waitMicros =
      storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
          + (long) (freshPermits * stableIntervalMicros);

  this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
  this.storedPermits -= storedPermitsToSpend;
  return returnValue;
}
```

首先通过resync生成令牌以及同步nextFreeTicketMicros时间戳，freshPermits从令牌桶中获取令牌后还需要的令牌数量，通过storedPermitsToWaitTime计算出获取freshPermits还需要等待的时间，在稳定模式中，这里就是(long) (freshPermits * stableIntervalMicros) ，然后更新nextFreeTicketMicros以及storedPermits，这次获取令牌需要的等待到的时间点， reserveAndGetWaitLength返回需要等待的时间间隔。

从`reserveEarliestAvailable`可以看出RateLimiter的预消费原理，以及获取令牌的等待时间时间原理（可以解释示例结果），再获取令牌不足时，并没有等待到令牌全部生成，而是更新了下次获取令牌时的nextFreeTicketMicros，从而影响的是下次获取令牌的等待时间。

`reserve`这里返回等待时间后，`acquire`通过调用`stopwatch.sleepMicrosUninterruptibly(microsToWait);`进行sleep操作，这里不同于Thread.sleep(), 这个函数的sleep是uninterruptibly的，内部实现：

```java
public static void sleepUninterruptibly(long sleepFor, TimeUnit unit) {
    //sleep 阻塞线程 内部通过Thread.sleep()
  boolean interrupted = false;
  try {
    long remainingNanos = unit.toNanos(sleepFor);
    long end = System.nanoTime() + remainingNanos;
    while (true) {
      try {
        // TimeUnit.sleep() treats negative timeouts just like zero.
        NANOSECONDS.sleep(remainingNanos);
        return;
      } catch (InterruptedException e) {
        interrupted = true;
        remainingNanos = end - System.nanoTime();
        //如果被interrupt可以继续，更新sleep时间，循环继续sleep
      }
    }
  } finally {
    if (interrupted) {
      Thread.currentThread().interrupt();
      //如果被打断过，sleep过后再真正中断线程
    }
  }
}
```

sleep之后，`acquire`返回sleep的时间，阻塞结束，获取到令牌。

```java
public boolean tryAcquire(int permits) {
  return tryAcquire(permits, 0, MICROSECONDS);
}

public boolean tryAcquire() {
  return tryAcquire(1, 0, MICROSECONDS);
}

public boolean tryAcquire(int permits, long timeout, TimeUnit unit) {
  long timeoutMicros = max(unit.toMicros(timeout), 0);
  checkPermits(permits);
  long microsToWait;
  synchronized (mutex()) {
    long nowMicros = stopwatch.readMicros();
    if (!canAcquire(nowMicros, timeoutMicros)) {
      return false;
    } else {
      microsToWait = reserveAndGetWaitLength(permits, nowMicros);
    }
  }
  stopwatch.sleepMicrosUninterruptibly(microsToWait);
  return true;
}

private boolean canAcquire(long nowMicros, long timeoutMicros) {
  return queryEarliestAvailable(nowMicros) - timeoutMicros <= nowMicros;
}

@Override
final long queryEarliestAvailable(long nowMicros) {
  return nextFreeTicketMicros;
}
```

`tryAcquire`函数可以尝试在timeout时间内获取令牌，如果可以则挂起等待相应时间并返回true，否则立即返回false
`canAcquire`用于判断timeout时间内是否可以获取令牌，通过判断当前时间+超时时间是否大于nextFreeTicketMicros 来决定是否能够拿到足够的令牌数，如果可以获取到，则过程同acquire，线程sleep等待，如果通过`canAcquire`在此超时时间内不能回去到令牌，则可以快速返回，不需要等待timeout后才知道能否获取到令牌。