# 频繁GC \(Allocation Failure\)及young gc时间过长分析

原文链接: [https://juejin.im/post/5a9b811a6fb9a028e46e1c88](https://juejin.im/post/5a9b811a6fb9a028e46e1c88)

## 0. 序

本文主要分析一个频繁GC \(Allocation Failure\)及young gc时间过长的case。

## 1. 症状

* gc throughput percent逐步下降，从一般的99.96%逐步下降，跌破99%，进入98%，最低点能到94%
* young gc time逐步增加，从一般的十几毫秒逐步上升，突破50，再突破100，150，200，250
* 在8.5天的时间内，发生了9000多次gc，其中full gc为4次，平均将近8秒，大部分是young gc\(`allocation failure为主`\)，平均270多毫秒，最大值将近7秒
* 平均对象创建速率为10.63 mb/sec，平均的晋升速率为2 kb/sec，cpu使用率正常，没有明显的飙升。

### 1.1 jvm参数

```text
-XX:+UseParallelGC -XX:+UseParallelOldGC -XX:ParallelGCThreads=4 -XX:+UseAdaptiveSizePolicy -XX:MaxHeapSize=2147483648 -XX:MaxNewSize=1073741824 -XX:NewSize=1073741824 -XX:+PrintGCDetails -XX:+PrintTenuringDistribution -XX:+PrintGCTimeStamps
```

### 1.2 jdk版本

```text
java -version
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.66-b17, mixed mode)
```

## 2. full gc

```text
27.066: [Full GC (Metadata GC Threshold) [PSYoungGen: 19211K->0K(917504K)] [ParOldGen: 80K->18440K(1048576K)] 19291K->18440K(1966080K), [Metaspace: 20943K->20943K(1069056K)], 0.5005658 secs] [Times: user=0.24 sys=0.01, real=0.50 secs] 
100.675: [Full GC (Metadata GC Threshold) [PSYoungGen: 14699K->0K(917504K)] [ParOldGen: 18464K->23826K(1048576K)] 33164K->23826K(1966080K), [Metaspace: 34777K->34777K(1081344K)], 0.7937738 secs] [Times: user=0.37 sys=0.01, real=0.79 secs]
195.073: [Full GC (Metadata GC Threshold) [PSYoungGen: 24843K->0K(1022464K)] [ParOldGen: 30048K->44782K(1048576K)] 54892K->44782K(2071040K), [Metaspace: 58220K->58220K(1101824K)], 3.7936515 secs] [Times: user=1.86 sys=0.02, real=3.79 secs] 
242605.669: [Full GC (Ergonomics) [PSYoungGen: 67276K->0K(882688K)] [ParOldGen: 1042358K->117634K(1048576K)] 1109635K->117634K(1931264K), [Metaspace: 91365K->90958K(1132544K)], 22.1573804 secs] [Times: user=2.50 sys=3.51, real=22.16 secs]
```

可以发现发生的4次full gc，前三次都是由于Metadata GC Threshold造成的，只有最后一次是由于Ergonomics引发的。

### 2.1 Full GC \(Metadata GC Threshold\)

这里使用的是java8，参数没有明确指定metaspace的大小和上限，查看一下:

```text
jstat -gcmetacapacity 7
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1136640.0    99456.0        0.0  1048576.0    12160.0 38009    16  275.801 14361.992
```

* 忽略后面的FGC，因为分析的日志只是其中四分之一
* 这里可以看到MCMX\(Maximum metaspace capacity \(kB\)\)有一个多G，而MC\(Metaspace capacity \(kB\)\)才97M左右，为啥会引起Full GC \(Metadata GC Threshold\)

**相关参数**

* -XX:MetaspaceSize，初始空间大小\(也是初始的阈值，即初始的high-water-mark\)，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
* -XX:MaxMetaspaceSize，最大空间，默认是没有限制的，取决于本地系统空间容量。
* -XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比\(`即元数据在当前分配大小的最大占用大小`\)，如果空闲比小于这个参数\(`即超过了最大占用大小`\)，那么将对meta space进行扩容。
* -XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比\(`即元数据在当前分配大小的最小占用大小`\)，如果空闲比大于这个参数\(`即小于最小占用大小`\)，那么将对meta space进行缩容.

**由于没有设置，在机器上的默认值为：**

```text
java -XX:+PrintFlagsFinal | grep Meta
    uintx InitialBootClassLoaderMetaspaceSize       = 4194304                             {product}
    uintx MaxMetaspaceExpansion                     = 5451776                             {product}
    uintx MaxMetaspaceFreeRatio                     = 70                                  {product}
    uintx MaxMetaspaceSize                          = 18446744073709547520                    {product}
    uintx MetaspaceSize                             = 21807104                            {pd product}
    uintx MinMetaspaceExpansion                     = 339968                              {product}
    uintx MinMetaspaceFreeRatio                     = 40                                  {product}
     bool TraceMetadataHumongousAllocation          = false                               {product}
     bool UseLargePagesInMetaspace                  = false                               {product}
```

**可以看到MinMetaspaceFreeRatio为40，MaxMetaspaceFreeRatio为70，MetaspaceSize为20M，Full GC \(Metadata GC Threshold\)主要分为了三次**

* 第一次，\[Metaspace: 20943K-&gt;20943K\(1069056K\)\]
* 第二次，\[Metaspace: 34777K-&gt;34777K\(1081344K\)\]
* 第三次，\[Metaspace: 58220K-&gt;58220K\(1101824K\)\]

可以看到metaspace的阈值不断动态调整，至于具体调整的逻辑，官方文档貌似没讲，这里暂时不深究。只要没有超过Max值就没有致命影响，但是对于低延时的应用来讲，是要尽量避免动态调整引起的gc耗时，可以根据调优计算并设置初始阈值来解决。

### 2.2 Full GC \(Ergonomics\)

这里可以到full gc的reason是Ergonomics，是因为开启了UseAdaptiveSizePolicy，jvm自己进行自适应调整引发的full gc。

## 3. GC \(Allocation Failure\)

分析完full gc之后我们看下young gc，看log里头99%都是GC \(Allocation Failure\)造成的young gc。Allocation Failure表示向young generation\(eden\)给新对象申请空间，但是young generation\(eden\)剩余的合适空间不够所需的大小导致的minor gc。

### 3.1 -XX:+PrintTenuringDistribution

```text
Desired survivor size 75497472 bytes, new threshold 2 (max 15)
- age   1:   68407384 bytes,   68407384 total
- age   2:   12494576 bytes,   80901960 total
- age   3:      79376 bytes,   80981336 total
- age   4:    2904256 bytes,   83885592 total
```

* 这个Desired survivor size表示survivor区域允许容纳的最大空间大小为75497472 bytes
* 下面的对象列表为此次gc之后，survivor当前存活对象的年龄大小分布，total大小为83885592 &gt; 75497472，而age1大小为68407384 &lt; 75497472，因此new threshold变为2\(`作用于下次gc`\)。下次gc如果对象没释放的话，超过阈值的对象将晋升到old generation。

### 3.2 age list为空

```text
59.463: [GC (Allocation Failure) 
Desired survivor size 134217728 bytes, new threshold 7 (max 15)
[PSYoungGen: 786432K->14020K(917504K)] 804872K->32469K(1966080K), 0.1116049 secs] [Times: user=0.10 sys=0.01, real=0.20 secs]
```

这里Desired survivor size这行下面并没有各个age对象的分布，那就表示此次gc之后，当前survivor区域并没有age小于max threshold的存活对象。而这里一个都没有输出，表示此次gc回收对象之后，没有存活的对象可以拷贝到新的survivor区。

gc之后survivor有对象的例子

```text
jstat -gcutil -h10 7 10000 10000
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00  99.99  90.38  29.82  97.84  96.99    413  158.501     4   14.597  173.098
 11.60   0.00  76.00  29.83  97.84  96.99    414  158.511     4   14.597  173.109
 11.60   0.00  77.16  29.83  97.84  96.99    414  158.511     4   14.597  173.109
  0.00  13.67  60.04  29.83  97.84  96.99    415  158.578     4   14.597  173.176
  0.00  13.67  61.05  29.83  97.84  96.99    415  158.578     4   14.597  173.176
```

* 在ygc之前young generation = eden + S1；ygc之后，young generation = eden + S0
* 观察可以看到ygc之后old generation空间没变，表示此次ygc，没有对象晋升到old generation。
* gc之后，存活对象搬移到了另外一个survivor区域
* 这里由于是每个10秒采样一次，存在延迟，即gc之后，立马有新对象在eden区域分配了，因此这里看到的eden区域有对象占用。

### 3.3 real time &gt; usr time + sys time

```text
722914.974: [GC (Allocation Failure) 
Desired survivor size 109576192 bytes, new threshold 15 (max 15)
[PSYoungGen: 876522K->8608K(941568K)] 1526192K->658293K(1990144K), 0.0102709 secs] [Times: user=0.03 sys=0.00, real=0.01 secs] 
722975.207: [GC (Allocation Failure) 
Desired survivor size 103284736 bytes, new threshold 15 (max 15)
[PSYoungGen: 843168K->39278K(941568K)] 1492853K->688988K(1990144K), 0.3607036 secs] [Times: user=0.17 sys=0.00, real=0.36 secs]
```

里头有大于将近300次的gc的real time时间大于usr time + sys time。

* real：指的是操作从开始到结束所经过的墙钟时间（WallClock Time）
* user：指的是用户态消耗的CPU时间；
* sys：指的是内核态消耗的CPU时间。

> 墙钟时间包括各种非运算的等待耗时，例如等待磁盘I/O、等待线程阻塞，而CPU时间不包括这些耗时，但当系统有多CPU或者多核的话，多线程操作会叠加这些CPU时间，所以看到user或sys时间超过real时间是完全正常的。

user + sys 就是CPU花费的实际时间，注意这个值统计了所有CPU上的时间，如果进程工作在多线程的环境下，叠加了多线程的时间，这个值是会超出 real 所记录的值的，即 user + sys &gt;= real 。

这里300多次real time时间大于usr time + sys time，表明可能有两个问题，一个是IO操作密集，另一个是cpu\(`分配`\)的额度不够。

## 4. 新生代垃圾回收机制

新对象尝试栈上分配，不行再尝试TLAB分配，不行则考虑是否直接绕过eden区在年老代分配空间\(`-XX:PretenureSizeThreshold设置大对象直接进入年老代的阈值，当对象大小超过这个值时，将直接在年老代分配。`\)，不行则最后考虑在eden申请空间。

![](https://image.ldbmcs.com/2019-07-19-070810.jpg)

* 向eden申请空间创建新对象，eden没有合适的空间，因此触发minor gc
* minor gc将eden区及from survivor区域的存活对象进行处理
  * 如果这些对象年龄达到阈值，则直接晋升到年老代
  * 若要拷贝的对象太大，那么不会拷贝到to survivor，而是直接进入年老代
  * 若to survivor区域空间不够/或者复制过程中出现不够，则发生survivor溢出，直接进入年老代
  * 其他的，若to survivor区域空间够，则存活对象拷贝到to survivor区域
* 此时eden区及from survivor区域的剩余对象为垃圾对象，直接抹掉回收，释放的空间成为新的可分配的空间
* minor gc之后，若eden空间足够，则新对象在eden分配空间；若eden空间仍然不够，则新对象直接在年老代分配空间。

## 5. 小结

从上面的分析可以看出，young generation貌似有点大，ygc时间长；另外每次ygc之后survivor空间基本是空的，说明新生对象产生快，生命周期也短，原本设计的survivor空间没有派上用场。因此可以考虑缩小下young generation的大小，或者改为G1试试。

关于-XX:+PrintTenuringDistribution有几个要点，要明确一下：

* 这个打印的哪个区域的对象分布\(`survivor`\)
* 是在gc之前打印，还是在gc之后打印\(`gc之后打印`\)
* 一个新生对象第一次到survivor时其age算0还是算1。

  对象的年龄就是他经历的MinorGC次数，对象首次分配时，年龄为0，第一次经历MinorGC之后，若还没有被回收，则年龄+1，由于是第一次经历MinorGC，因此进入survivor区。因此对象第一次进入survivor区域的时候年龄为1.

* 晋升阈值\(new threshold\)动态调整

  如果底下age的total大小大于Desired survivor size的大小，那么就代表了survivor空间溢出了，被填满，然后会重新计算threshold。

## 6. doc

* [jstat](https://link.juejin.im/?target=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Ftechnotes%2Ftools%2Funix%2Fjstat.html)
* [Size of Huge Objects directly allocated to Old Generation](https://link.juejin.im/?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F24618467%2Fsize-of-huge-objects-directly-allocated-to-old-generation)
* [Java对象分配简要流程](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000004606059)
* [记一次JVM优化过程](https://link.juejin.im/?target=http%3A%2F%2Fsunxiang0918.cn%2F2014%2F06%2F27%2F%E8%AE%B0%E4%B8%80%E6%AC%A1JVM%E4%BC%98%E5%8C%96%E8%BF%87%E7%A8%8B%2F)
* [Survivor空间溢出实例](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000004657756)
* [Java 垃圾回收的log，为什么 from和to大小不等？](https://link.juejin.im/?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F65601024%2Fanswer%2F236656917)
* [Useful JVM Flags – Part 5 \(Young Generation Garbage Collection\)](https://link.juejin.im/?target=https%3A%2F%2Fblog.codecentric.de%2Fen%2F2012%2F08%2Fuseful-jvm-flags-part-5-young-generation-garbage-collection%2F)
* [JDK-6453675 : Request for documentation of -XX:+PrintTenuringDistribution output](https://link.juejin.im/?target=https%3A%2F%2Fbugs.java.com%2Fbugdatabase%2Fview_bug.do%3Fbug_id%3D6453675)
* [How to read the output of +PrintTenuringDistribution](https://link.juejin.im/?target=https%3A%2F%2Fmarc.info%2F%3Fl%3Dopenjdk-hotspot-gc-use%26m%3D138267652801021%26w%3D2)
* [一次GC Tuning小记](https://link.juejin.im/?target=https%3A%2F%2Fneway6655.github.io%2Fjava%2C%20gc%20tuning%2F2016%2F09%2F24%2Fgc-tuning.html)
* [JDK8 的FullGC 之 metaspace](https://link.juejin.im/?target=http%3A%2F%2Ftech.dianwoda.com%2F2018%2F01%2F10%2Fjdk8-de-full-gc-zhi-metaspace%2F)
* [Java PermGen 去哪里了?](https://link.juejin.im/?target=http%3A%2F%2Fifeve.com%2Fjava-permgen-removed%2F)
* [Metaspace](https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fxlnjulp%2Farticle%2Fdetails%2F46763045)
* [MetaspaceSize的坑](https://link.juejin.im/?target=http%3A%2F%2Fatbug.com%2Fjava8-metaspace-size-issue%2F)
* [JVM源码分析之Metaspace解密](https://link.juejin.im/?target=http%3A%2F%2Flovestblog.cn%2Fblog%2F2016%2F10%2F29%2Fmetaspace%2F)
* [About G1 Garbage Collector, Permanent Generation and Metaspace](https://link.juejin.im/?target=https%3A%2F%2Fblogs.oracle.com%2Fpoonam%2Fabout-g1-garbage-collector%2C-permanent-generation-and-metaspace)
* [聊聊jvm的PermGen与Metaspace](https://link.juejin.im/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000012577387)
* [GC LOGGING – USER, SYS, REAL – WHICH TIME TO USE? & GANDHI](https://link.juejin.im/?target=https%3A%2F%2Fblog.gceasy.io%2F2016%2F04%2F06%2Fgc-logging-user-sys-real-which-time-to-use%2F)
* [REAL TIME IS GREATER THAN USER AND SYS TIME](https://link.juejin.im/?target=https%3A%2F%2Fblog.gceasy.io%2F2016%2F12%2F08%2Freal-time-greater-than-user-and-sys-time%2F)
* [GC日志时间分析: user + sys &lt; real](https://link.juejin.im/?target=http%3A%2F%2Ftang.love%2F2017%2F10%2F22%2Fgc_analysis_user_sys_real%2F)
* [What is promotion rate?](https://link.juejin.im/?target=https%3A%2F%2Fplumbr.io%2Fblog%2Fgarbage-collection%2Fwhat-is-promotion-rate)

