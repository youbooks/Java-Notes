> 转载：[JAVA GC日志分析](https://juejin.im/post/6844903861556084744)

这里以Java8为例。

## 1. GC环境模拟

首先我们给出如下代码用来触发GC：

```java
public static void main(String[] args) {
    // 每100毫秒创建100线程，每个线程创建一个1M的对象，即每100ms申请100M堆空间
    Executors.newScheduledThreadPool(1).scheduleAtFixedRate(() -> {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    //  申请1M
                    byte[] temp = new byte[1024 * 1024];
                    Thread.sleep(new Random().nextInt(1000)); // 随机睡眠1秒以内
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }, 1000, 100, TimeUnit.MILLISECONDS);
}
```

我们要模拟的场景是年轻代不断地Young GC，并有一部分对象晋升到老年代，当老年代空间不足时触发Full GC。

程序逻辑：每100毫秒创建100个线程，每个线程创建一个1M的对象，即每100ms申请100M堆空间。之所以每个线程随机睡眠1s，是为了避免对象朝生夕灭，保证可以有一部分对象能晋升到老年代，更好的触发Young GC 和 Full GC，注意这个睡眠时间如果大了，会导致OOM，如果小了，很难触发FULL GC。

### 1.1 虚拟机参数解释

启动Java进程：`java -Xms200m -Xmx200m -Xmn100m -verbose:gc -XX:+PrintGCDetails -Xloggc:./gc.log -XX:+PrintGCDateStamps -jar demo-0.0.1-SNAPSHOT.jar`

- `-Xms200m -Xmx200m` 最小/最大堆内存 200M

- `-Xmn100m` 年轻代内存 100M

- `-verbose:gc` 开启GC日志

- `-XX:+PrintGCDetails -Xloggc:./gc.log -XX:+PrintGCDateStamps` 将GC日志详情输入到gc.log中

## 2. jmap分析

jcmd 获取我们Java进程的Id：6264。

`jmap -heap 6264`查看堆信息。

第一次查看，我们发现 Eden区是98M，S0、S1是1M：

![2020-11-15-Z3dii9](https://image.ldbmcs.com/2020-11-15-Z3dii9.jpg)

第二次查看， Eden区是99M，S0、S1是0.5M：

![2020-11-15-1wB4MY](https://image.ldbmcs.com/2020-11-15-1wB4MY.jpg)

Eden区与Survivor区的比例在动态的变化，并不是默认的8:1:1。

原来我们使用默认的垃圾收集器`Parallel Scavenge+Parallel Old`组合，而该收集器下`-XX:+UseAdaptiveSizePolicy`是默认开启的，即Eden区与Survivor区比例根据GC情况会自适应变化。

我们加上参数，关闭年轻代自适应，年轻代比例设置为8:1:1

`-XX:-UseAdaptiveSizePolicy -XX:SurvivorRatio=8`

另外为了尽早的触发FULL GC，我们新增虚拟机参数

`-XX:MaxTenuringThreshold=10`

晋升年龄由默认的15修改为10，使得年轻代的对象更容易晋升到老年代

重启虚拟机查看jmap：

![2020-11-15-zVBg0M](https://image.ldbmcs.com/2020-11-15-zVBg0M.jpg)

### 2.1 年轻代

- Eden区80M 已使用51M，当前使用率63.8%
- S0区10M 已使用0.43M，使用率4.37%
- S1区10M 使用率为空

### 2.2 老年代

- 100M 已使用18.39M，使用率18.9%

## 3. GC日志内容分析

查看我们输出的GC日志gc.log，选取其中两段

> 2019-06-09T02:55:30.993+0800: 330.811: [GC (Allocation Failure) [PSYoungGen: 82004K->384K(92160K)] 184303K->102715K(194560K), 0.0035647 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 2019-06-09T02:55:30.997+0800: 330.815: [Full GC (Ergonomics) [PSYoungGen: 384K->0K(92160K)] [ParOldGen: 102331K->5368K(102400K)] 102715K->5368K(194560K), [Metaspace: 16941K->16914K(1064960K)], 0.0213953 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]

### 3.1 Young GC

> [GC (Allocation Failure) [PSYoungGen: 82004K->384K(92160K)] 184303K->102715K(194560K), 0.0035647 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

Young GC日志：

![2020-11-15-z4t1WF](https://image.ldbmcs.com/2020-11-15-z4t1WF.jpg)

解释:

- 年轻代GC：[GC前年轻代80.08M->GC后0.37M(年轻代总大小90M)]GC前堆179.98M->GC后堆100.3M(堆总大小190M)，用时]
- 其中年轻代总大小是90M而不是100M，这里我理解是年轻代当前最大申请到90M
- 100M*80%=80M 是Eden区大小
- 80M*80% = 64M Eden区默认占用超过8成即64M就会触发YoungGC

### 3.2 Full GC

> [Full GC (Ergonomics) [PSYoungGen: 384K->0K(92160K)] [ParOldGen: 102331K->5368K(102400K)] 102715K->5368K(194560K), [Metaspace: 16941K->16914K(1064960K)], 0.0213953 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]

Full GC日志：

![2020-11-15-7FXDMp](https://image.ldbmcs.com/2020-11-15-7FXDMp.jpg)

解释：

- [GC前年轻代0.375M->GC后年轻代0M(年轻代总大小90M)][GC前老年代99.93M->GC后老年代5.24M(老年代总大小100M)]GC前堆100.3M->GC后堆5.24M（堆总大小190M）,[元数据区:GC前16.5,GC后16.5(元数据区总大小1040M)]，用时]
- 可以推测出此次FullGC原因是年轻代晋升老年代空间不足导致

## 4. 利用可视化工具分析

这里我们利用 [gceasy.io/](http://gceasy.io/) 分析一下：

1. 统计年轻代、老年代、元数据区最大可用空间以及峰值，这里元数据区大小在我们的虚拟机参数没有配置，所以取的是默认值1040M。

   ![2020-11-15-uX84GU](https://image.ldbmcs.com/2020-11-15-uX84GU.jpg)

2. 吞吐量、GC平均延迟、最大延迟以及延迟区间的统计

   ![2020-11-15-yarEev](https://image.ldbmcs.com/2020-11-15-yarEev.jpg)

3. 堆所用大小的实时分析，红色位置是发生了FullGC使得堆总量直线下降

   ![2020-11-15-Nfb9Aq](https://image.ldbmcs.com/2020-11-15-Nfb9Aq.jpg)

   会发现虚拟机在刚启动不久的阶段触发大量的FULL GC，我的理解是我们申请的对象都随机睡眠一秒以内，刚启动时大部分还存在线程的引用，GCRoot可达。在刚启动的时候触发FULL GC并不会完整清理掉老年代空间并由于空间不足不断触发FULL GC。

4. GC空间总量和时间的统计

   ![2020-11-15-q9qygi](https://image.ldbmcs.com/2020-11-15-q9qygi.jpg)

5. 各类GC时间、GC次数、GC总量等指标

   ![2020-11-15-6MDrXt](https://image.ldbmcs.com/2020-11-15-6MDrXt.jpg)

## 5. 总结

GC日志分析可以帮助我们宏观的监控GC运行情况。一方面如果频繁的FullGC会有严重的性能问题(STW)，另一方面过于频繁的GC，即GC占用系统正常运行的比重过多，吞吐量低，则是一定程度上的性能资源浪费。若系统存在性能问题，根据GC分析各项指标的作为参考，我们也可以适当的在程序里或虚拟机参数做些调优。