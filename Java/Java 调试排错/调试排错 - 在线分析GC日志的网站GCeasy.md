# 调试排错 - 在线分析GC日志的网站GCeasy

> 原文链接：[一文学会使用GCeasy——一款超好用的在线分析GC日志的网站](https://blog.csdn.net/CoderBruis/article/details/101234738)

## 1. 前言

- 此次的GC分析，用的是ParallelGC的日志
- JDK1.8

可能很多读者都不知道如何打印出程序的GC日志，下面来介绍分别使用IDEA输出GC日志和直接从Tomcat服务器上输出GC日志。其实这两种方式都使用了同样的JVM命令：

```java
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:./gc.log
```

- -XX:+PrintGCDetails：表示的是打印GC日志详情
- -XX:+PrintGCTimeStamps：表示打印GC时间戳
- -Xloggc: ./gc.log：表示在当前目录下生成gc.log文件

### 1.1 使用IDEA输出GC日志

![2021-02-18-hcFjX8](https://image.ldbmcs.com/2021-02-18-hcFjX8.jpg)

输出日志结果可以在图中Application server设置的Tomcat 7.0.94中的bin目录下，如图：

![2021-02-18-Uuv7NP](https://image.ldbmcs.com/2021-02-18-Uuv7NP.jpg)

### 1.2 在Tomcat服务器上输出GC日志

将上述设置的JVM参数，配置到Tomcat/bin/catalina.sh中，如图：

![2021-02-18-2WJozN](https://image.ldbmcs.com/2021-02-18-2WJozN.jpg)

导出结果同样的在tomcat/bin/目录下。

打印出GC日志之后，就可以拿去[GCeasy官网](https://www.gceasy.io/)上进行GC可视化分析了。下文将详细讲解下GCeasy的图形化分析结果。
进入GCeasy官网之后，选择需要导入的GC日志文件导入即可。

![2021-02-18-DF0KOu](https://image.ldbmcs.com/2021-02-18-DF0KOu.jpg)

## 2. 正文

### 2.1 JVM Heap Size

这一部分分别使用了表格和图形界面来展示了JVM堆内存大小。如图所示：

![2021-02-18-rmr5fF](https://image.ldbmcs.com/2021-02-18-rmr5fF.jpg)

左侧分别展示了年轻代的内存分配分配空间大小（Allocated）和年轻代内存分配空间大小的最大峰值（Peek），然后依次是老年代（Old Generation）、元数据区（Meta Space）、堆区和非堆区（Young + Old + Meta Space）总大小。**值得注意的是，每一代的最大内存利用率都会超过分配的大小，但是图中的内存分配利用率已经超过了峰值内存了。**

### 2.2 Key Performance Indicators

这一部分是关键的性能指标，如图：

![2021-02-18-HZg3Le](https://image.ldbmcs.com/2021-02-18-HZg3Le.jpg)

- Throughput表示的是吞吐量
- Latency表示响应时间
  - Avg Pause GC Time 平均GC时间
  - Max Pause GC TIme 最大GC时间

### 2.3 Interactive Graphs

![2021-02-18-4DhtEo](https://image.ldbmcs.com/2021-02-18-4DhtEo.jpg)

第一部分是Heap after GC，GC后堆的内存图，堆是用来存储对象的，从图中可以看出，随着GC的进行，垃圾回收器把对象都回收掉了，因此堆的大小逐渐增大。

![2021-02-18-sYRRvm](https://image.ldbmcs.com/2021-02-18-sYRRvm.jpg)

第二部分是Heap before GC，这是GC前堆的使用率，可以看出随着程序的运行，堆使用率越来越高，堆被对象占用的内存越来越大。

![2021-02-18-JLlFeT](https://image.ldbmcs.com/2021-02-18-JLlFeT.jpg)

第三部分是GC Duration Time，就是GC持续时间。从图中可以看到，发生Full GC的时间持续的比较端，而Young GC持续的时间比较长。图中横坐标表示GC发生的时间段，纵坐标表示的是GC持续时间。

![2021-02-18-5O37Xf](https://image.ldbmcs.com/2021-02-18-5O37Xf.jpg)

第四部分表示的是GC导致程序停顿持续的时间，一个GC事件的发生具有多个阶段，而不同的垃圾回收器又有不同的阶段，这里展示不作细分。这些阶段（例如并发标记，并发清除等）与程序线程一起并发运行，此时不会暂停程序线程。但是某些阶段（例如初始标记，清除等）会暂停整个应用程序，所以此图标描述的仅暂停阶段所花费的时间。

![2021-02-18-Xj22A4](https://image.ldbmcs.com/2021-02-18-Xj22A4.jpg)

第五部分表示的是GC回收掉的垃圾对象的内存大小。从图中可以看出，Full GC仅仅回收不到1M的对象，而Young GC则回收的对象比较多，大部分发生在12.40左右。

![2021-02-18-WOTV8K](https://image.ldbmcs.com/2021-02-18-WOTV8K.jpg)

第六部分表示的是Young Gen，年轻代的内存分配情况。对象都是朝生夕死，年轻代存放的就是刚刚产生的对象，每进行一次GC，都会GC掉很多垃圾对象，剩下的就是右GC Root关联的对象，这些对象会年龄会逐渐增加，达到了一定阈值就会晋升为老年代的对象。可以看到before GC表示的图线随着时间的进行逐渐增大，也就是年轻代中对象越来越多，而GC事件发生后，年轻代中对象就会减少，也就是after GC图线表示的内存变化趋势。

![2021-02-18-woQdyc](https://image.ldbmcs.com/2021-02-18-woQdyc.jpg)

第七部分是Old Gen，表示的是老年代的内存分配情况。细心的读者会发现，为啥一开始before GC的内存大小比after GC的内存分配要少呢？这里得先知道老年代存放的都是年龄大的对象，意思就是经过了多次GC都没有被GC掉的对象，就会晋升为老年代的对象。所以这就解释了为啥after GC内存要比before GC内存要大，因为每次GC过后，都会有年轻代的对象晋升为老年代对象。

![2021-02-18-MQVcfE](https://image.ldbmcs.com/2021-02-18-MQVcfE.jpg)

第八部分是Meta Space，表示的是元数据区内存分配情况。

![2021-02-18-0bXERF](https://image.ldbmcs.com/2021-02-18-0bXERF.jpg)

第九部分表示的是堆内存分配和晋升情况，从图中可以看出，随着时间的推移，堆区中对象分配越来越多，同事晋升对象也越来越多。

### 2.4 GC Statistics

![2021-02-18-NrpfmW](https://image.ldbmcs.com/2021-02-18-NrpfmW.jpg)

左图：表示的是堆内存中Minor GC和Full GC回收垃圾对象的内存。
中图：总计GC时间，包括Minor GC和Full GC，时间单位为ms。
右图：GC平均时间，包括了Minor GC和Full GC。

![2021-02-18-4zIUKT](https://image.ldbmcs.com/2021-02-18-4zIUKT.jpg)

接下来则分别表示的是总GC统计，MinorGC的统计，FullGC的统计，GC暂停程序的统计。

### 2.5 Object Stats

![2021-02-18-PFz754](https://image.ldbmcs.com/2021-02-18-PFz754.jpg)

接下来是对象统计，Total Created Bytes表示的是创建的字节总数，Total promoted bytes表示的是晋升的字节总数，Avg creation rate表示的是平均创建字节率，Avg promotion rate表示平均的晋升率。

### 2.6 Memory Leak

由于记录的程序没有内存泄漏，所以这里就没有内存泄漏的日志信息。此处可以诊断8种OOM中的5种（Java堆内存溢出，超出GC开销限制，请求数组大小超过JVM限制，Permgen空间，元空间）。

![2021-02-18-AGOjrN](https://image.ldbmcs.com/2021-02-18-AGOjrN.jpg)

CG所花费的时间，也就是停顿线程的时间。

除了上面介绍的以外，还有Consecutive Full GC、Safe Point Duration、Tenuring Summary以及COmmand Line Flags等等，有兴趣的读者可以自己深入学习下。

