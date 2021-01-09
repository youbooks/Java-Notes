# JVM 基础 - 来自JDK12的全新并发压缩垃圾回收器Shenandoah GC

[TOC]

> 转载： [Shenandoah GC：一个来自JDK12的全新并发压缩垃圾回收器](https://cloud.tencent.com/developer/article/1547019)

是不是才听说了JDK11的ZGC，并且还没搞懂？不好意思，OpenJDK12马不停蹄的带来了Shenandoah GC。

## 1. 概述

JDK12新增的一个名为Shenandoah的GC算法，它的evacuation阶段工作能通过与正在运行中Java工作线程同时进行（即并发，concurrent），从而减少GC的停顿时间。

Shenandoah的停顿时间和堆的大小没有任何关系，这就意味着无论你的堆是200MB，2GB还是200GB，停顿时间是一样的。

![2021-01-09-1mRCq2](https://image.ldbmcs.com/2021-01-09-1mRCq2.jpg)

如上图所示，Shenandoah GC每个GC周期由2个STW（Stop The World）阶段和2个并发阶段组成。在初始化标记阶段，扫描root集合的时候会STW。然后并发标记阶段，Shenandoah GC和Java工作线程一起运行，最后，在最终标记阶段，又会STW，然后执行一个并发evacuation阶段。

>  Root集合包括：thread local variables, references embedded in generated code, interned Strings, references from classloaders (e.g. static final references), JNI references, JVMTI references.

## 2. 深入剖析

**Shenandoah是一个基于Region设计的垃圾收集器**，这点和G1类似，它把整个堆当作Region集合来维护。但是，Shenandoah不需要remember set或者card table来记录跨region引用。

其中一个原因是（无条件）card mark可能引起false sharing，而Brooks pointer分散在每个对象头上，比较不容易引起false sharing：

 ![2021-01-09-6fa18B](https://image.ldbmcs.com/2021-01-09-6fa18B.jpg)

 一个常规的Shenandoah GC周期大概是这样的（跟G1也有点相似）:

![2021-01-09-O5O1Pt](https://image.ldbmcs.com/2021-01-09-O5O1Pt.jpg)

GC日志如下：

```Java
GC(3) Pause Init Mark 0.771msGC(3) Concurrent marking 76480M->77212M(102400M) 633.213msGC(3) Pause Final Mark 1.821msGC(3) Concurrent cleanup 77224M->66592M(102400M) 3.112msGC(3) Concurrent evacuation 66592M->75640M(102400M) 405.312msGC(3) Pause Init Update Refs 0.084msGC(3) Concurrent update references  75700M->76424M(102400M) 354.341msGC(3) Pause Final Update Refs 0.409msGC(3) Concurrent cleanup 76244M->56620M(102400M) 12.242ms
```
每个阶段要做的事情如下：

1. Init Mark 并发标记的初始化阶段，它为并发标记准备堆和应用线程，然后扫描root集合。这是整个GC生命周期第一次停顿，这个阶段主要工作是root集合扫描，所以停顿时间主要取决于root集合大小。

2. Concurrent Marking 贯穿整个堆，以root集合为起点，跟踪可达的所有对象。 这个阶段和应用程序一起运行，即并发（concurrent）。这个阶段的持续时间主要取决于存活对象的数量，以及堆中对象图的结构。由于这个阶段，应用依然可以分配新的数据，所以在并发标记阶段，堆占用率会上升。

3. Final Mark 清空所有待处理的标记/更新队列，重新扫描root集合，结束并发标记。. 这个阶段还会搞明白需要被清理（evacuated）的region（即垃圾收集集合），并且通常为下一阶段做准备。最终标记是整个GC周期的第二个停顿阶段，这个阶段的部分工作能在并发预清理阶段完成，这个阶段最耗时的还是清空队列和扫描root集合。

4. Concurrent Cleanup 回收即时垃圾区域 -- 这些区域是指并发标记后，探测不到任何存活的对象。

5. Concurrent Evacuation 从垃圾收集集合中拷贝存活的对到其他的region中，这是有别于OpenJDK其他GC主要的不同点。这个阶段能再次和应用一起运行，所以应用依然可以继续分配内存，这个阶段持续时间主要取决于选中的垃圾收集集合大小（比如整个堆划分128个region，如果有16个region被选中，其耗时肯定超过8个region被选中）。

6. Init Update Refs 初始化更新引用阶段，它除了确保所有GC线程和应用线程已经完成并发Evacuation阶段，以及为下一阶段GC做准备以外，其他什么都没有做。这是整个GC周期中，第三次停顿，也是时间最短的一次。

7. Concurrent Update References 再次遍历整个堆，更新那些在并发evacuation阶段被移动的对象的引用。这也是有别于OpenJDK其他GC主要的不同，这个阶段持续时间主要取决于堆中对象的数量，和对象图结构无关，因为这个过程是线性扫描堆。这个阶段是和应用一起并发运行的。

8. Final Update Refs 通过再次更新现有的root集合完成更新引用阶段，它也会回收收集集合中的region，因为现在的堆已经没有对这些region中的对象的引用。

这是整个GC周期最后一个阶段，它的持续时间主要取决于root集合的大小。

Concurrent Cleanup 回收那些现在没有任何引用的Region集合。

## 3. 目标

Shenandoah不是一个要一统天下的GC，有一些其他的吞吐量优先，或者内存占用优先的GC算法，它们并不是把响应性放在第一位（即不是主要考虑缩短停顿时间）。

**Shenandoah是一个对那些更看重响应性和可预测短暂停顿的应用来说，更合适的GC算法。**它的目标不是要解决所有JVM的停顿问题，由于GC之外的其他原因（例如到达安全点时间（TTSP--Time To Safe Point）问题）而暂停时间超出了此JEP的范围。

现代服务器比以前拥有更多的内存和处理器，SLA应用需要保证RP在10～500ms。为了达到
 最苛刻的目标（保证RP在10ms以内），我们需要GC的算法足够高效，允许程序在可用内存中运行，并且经过优化后，永远不会让正在运行的程序的停顿时间超过5毫秒（a handful of milliseconds，一只手就5根手指头，所以是5ms）。

Shenandoah就是这样一个OpenJDK为更近这个目标而设计的开源、低停顿时间的垃圾回收器。

## 4. 替代方案

1. Zing/Azul是一个没有停顿的垃圾收集器，但是不会贡献给OpenJDK。

2. 基于`colored pointers`设计的ZGC也是一个拥有很低停顿时间的垃圾收集器，Shenandoah期望能与之一战。

3. G1很多工作都是并行或者并发的，但是evacuation阶段不能并发执行。

4. CMS能并发标记，但是它执行年轻代拷贝时，需要STW，并且不会压缩老年代，这就会导致花费更多时间来管理老年代中的可用空间以及碎片问题。

## 5. 使用

这还是一个体验功能，需要增加-XX:+UnlockExperimentalVMOptions参数才能开启Shenandoah GC：

```Java
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
```

## 6. 常规测试

RedHat已经做了大量的测试，OpenJDK也为Shenandoah开发了很多测试用例。而且从Fedora 24开始Shenandoah在Fedora中随着JDK一起发布，并在Rhel7.4中作为技术预览. 通过`-XX:+UseShenandoahGC`运行标准的OpenJDK完全足够。

## 7. 压力测试

关于CMS，G1，ParallelOld，Shenandoah的延迟测试对比，如下图所示：

![2021-01-09-a9WNP9](https://image.ldbmcs.com/2021-01-09-a9WNP9.jpg)
