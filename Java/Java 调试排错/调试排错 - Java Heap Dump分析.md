# 调试排错 - Java Heap Dump分析

> 转载：[JVM heap dump分析](https://www.jianshu.com/p/c34af977ade1)

## 1. 前言

在故障定位(尤其是out of memory)和性能分析的时候，经常会用到一些文件辅助我们排除代码问题。这些文件记录了JVM运行期间的内存占用、线程执行等情况，这就是我们常说的dump文件。常用的有heap dump和thread dump（也叫javacore，或java dump）。我们可以这么理解：heap dump记录内存信息的，thread dump记录CPU信息。

## 2. 概念

heap dump： heap dump文件是一个二进制文件，它保存了某一时刻JVM堆中对象使用情况。HeapDump文件是指定时刻的Java堆栈的快照，是一种镜像文件。

> Heap dump is a snapshot of your application’s memory in a point in time. It contains information such as what are the objects in memory, what values do they carry, what is their size, what other objects do they reference. 

虚拟机：指以软件的方式模拟具有完整硬件系统功能、运行在一个完全隔离环境中的完整计算机系统 ，是物理机的软件实现。常用的虚拟机有VMWare，Visual Box，Java Virtual Machine（Java虚拟机，简称JVM）。

Java虚拟机阵营：Sun HotSpot VM、BEA JRockit VM、IBM J9 VM、Azul VM、Apache Harmony、Google Dalvik VM、Microsoft JVM…

### 2.1 为何会内存溢出？

![2021-01-10-NbEath](https://image.ldbmcs.com/2021-01-10-NbEath.jpg)

我们知道JVM根据generation(代)来进行GC，根据上图所示，一共被分为young generation(年轻代)、tenured generation(老年代)。

![2021-01-10-O1L2fE](https://image.ldbmcs.com/2021-01-10-O1L2fE.jpg)

绝大多数的对象都在young generation被分配，也在young generation被收回，当young generation的空间被填满，GC会进行minor collection(次回收)，速度非常快。其中，young generation中未被回收的对象被转移到tenured generation，当tenured generation被填满时，即触发major collection(FULL GC主回收)，整个应用程序都会停止下来直到回收完成。

因此，产生heap dump(内存溢出)错误原因一般出于以下原因：

1. JVM内存过小

2. 程序不严密

3. 产生过多的垃圾无法回收

### 2.2 heap dump文件如何生成？

#### 2.2.1 使用 jmap 命令生成

jmap 命令是JDK提供的用于生成堆内存信息的工具，切换到JDK_HOME/bin目录下后，执行下面的命令生成Heap Dump：

- windows环境：

  ```bash
  jmap -dump:live,format=b,file=heap.hprof <pid>
  ```

- linux环境：

  ```bash
  ./jmap -dump:live,format=b,file=heap.hprof <pid>
  ```

其中pid是JVM进程的id，heap.hprof是生成的heap dump文件,在执行命令的目录下面。推荐此种方法。

#### 2.2.2 使用 JConsole 生成

JConsole是JDK提供的一个基于GUI查看JVM系统信息的工具，既可以管理本地的JVM，也可以管理远程的JVM，可以通过下图的 dumpHeap 按钮生成 Heap Dump文件。

![2021-01-10-ZRBoFy](https://image.ldbmcs.com/2021-01-10-ZRBoFy.jpg)

SAP也使用自封装的远程监控jvm服务端口，从而监控jvm运行状态，

在JVM的配置参数中可以添加 `-XX:+HeapDumpOnOutOfMemoryError` 参数，当应用抛出 `OutOfMemoryError` 时自动生成dump文件。

### 2.3 常见heapdump文件分析工具

#### 2.3.1 jhat

jhat 是JDK自带的用于分析JVM Heap Dump文件的工具，使用下面的命令可以将堆文件的分析结果以HTML网页的形式进行展示：

```java
jhat <heap-dump-file>
```

其中 heap-dump-file 是文件的路径和文件名，可以使用 `-J-Xmx512m` 参数设置命令的内存大小。执行成功之后显示如下结果：

```java
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

这个时候访问 http://localhost:7000/ 即可以看到结果了。

#### 2.3.2 Eclipse Memory Analyzer(MAT)

**Eclipse Memory Analyzer(MAT)**是Eclipse提供的一款用于Heap Dump分析的工具，用来辅助发现内存泄漏减少内存占用，从数以百万计的对象中快速计算出对象的 Retained Size，查看并自动生成一个 Leak Suspect（内存泄露可疑点）报表。

相关操作下面将详细进行介绍。

Eclipse Memory Analyzer（MAT）：启动打开 File - Open Heap Dump... 菜单，然后选择生成的Heap DUmp文件，选择 "Leak Suspects Report"，然后点击 "Finish" 按钮。

![2021-01-10-XltFU2](https://image.ldbmcs.com/2021-01-10-XltFU2.jpg)

第一次打开因需要分析dump文件，所以需要等待一段时间进行分析，分析完成之后dump文件目录下面的文件信息如下：

![2021-01-10-63HSIe](https://image.ldbmcs.com/2021-01-10-63HSIe.jpg)

上图中 heap-27311.bin 文件是原始的Heap Dump文件，zip文件是生成的html形式的报告文件。其中zip文件可以直接打开通过网页打开查看概况。

打开如下所示：

![2021-01-10-aid9Ch](https://image.ldbmcs.com/2021-01-10-aid9Ch.jpg)

界面常用到功能：

- Overview（视图）：概要界面，显示了概要的信息，并展示了MAT常用的一些功能。

  > 1. Details 显示了一些统计信息，包括整  个堆内存的大小、类（Class）的数量、对象（Object）的数量、类加载器（Class Loader)的数量。
  > 2. Biggest Objects by Retained Size 使用饼图的方式直观地显示了在JVM堆内存中最大的几个对象，当光标移到饼图上的时候会在左边Inspector和Attributes窗口中显示详细的信息。
  > 3. Actions 这里显示了几种常用到的操作，算是功能的快捷方式，包括 Histogram、Dominator Tree、Top Consumers、Duplicate Classes，具体的含义和用法见下面；
  > 4. Reports 列出了常用的报告信息，包括 Leak Suspects和Top Components，具体的含义和内容见下；
  > 5. Step By Step 以向导的方式引导使用功能。

- Histogram（直方图）：可以查看每个类的实例（即对象）的数量和大小。

- Dominator Tree（支配树）：列出Heap Dump中处于活跃状态中的最大的几个对象，默认按 retained size进行排序，因此很容易找到占用内存最多的对象。

- OQL（MAT提供对象查询语言）：跟SQL语言类似，将类当作表、对象当作记录行、成员变量当作表中的字段，通过OQL可以方便快捷的查询一些需要的信息。

- Thread Overview：查看生成Heap Dump文件的时候线程的运行情况，用于线程的分析。

- Run Expert System Test：查看分析完成的HTML形式的报告，子菜单项如下图所示：

  ![2021-01-10-4cCNWx](https://image.ldbmcs.com/2021-01-10-4cCNWx.jpg)

  常用的主要有Leak Suspects和Top Components两种报告：

  1. Leak Suspects ：该报告分析了Heap Dump并尝试找出内存泄漏点，最后在生成的报告中对检测到的可疑点做详细的说明；
  2. Top Components ：列出占用总堆内存超过1%的对象。

### 2.3.3 [HeapHero](https://heaphero.io/)

Java & Android Heap Dump Analyzer

### 2.4 案例分析

OOM应用场景：当出现OutofMemory时，将会在相应的目录下生成一份dump文件，而如果不指定选项“XX:HeapDumpPath”，则在当前目录下生成dump文件。在此需要注意，尽管不借助jmap工具，MAT工具也能够直接生成dump文件，但是考虑到生产环境中几乎不可能在线对其进行分析，大都是采用离线分析，因此使用jmap+MAT工具最常见最科学的组合。

当上述程序执行时，必然会触发OutofMemory，然后在所指定的目录下找到生成的dump文件后，我们便可以通过MAT工具来进行分析了。当成功启动MAT后，通过菜单选项“File->Open heap dump...”打开指定的dump文件后，将会生成Overview选项，如图1-1所示：

![2021-01-10-E7xiaw](https://image.ldbmcs.com/2021-01-10-E7xiaw.jpg)

在Overview选项中，以饼状图的形式列举出了程序内存消耗的一些基本信息，其中每一种不同颜色的饼块都代表了不同比例的内存消耗情况。如果说需要定位内存泄露的代码点，我们可以通过Dominator Tree菜单选项来进行排查，因此仔细观察和分析才能够找到问题所在)，如图1-2所示：

![2021-01-10-ngOAE6](https://image.ldbmcs.com/2021-01-10-ngOAE6.jpg)

#### 2.4.1 案例一

**问题现象**：客户系统频繁重启，从系统log中发现OOM Error: Server process shutting down with exit code [666]; memory allocation error [OutOfMemoryError]，需要通过分析heap dump定位问题原因。

**分析过程：**查看heap dump中消耗内存大的对象分布情况，总大小：2.7G，其中：有两个大块内存占比大分别：1.7G和376.5M，如下图：

![2021-01-10-gGHHIt](https://image.ldbmcs.com/2021-01-10-gGHHIt.jpg)

在Overview选项中，以饼状图的形式列举出了程序内存消耗的一些基本信息。

查看Leak Suspects（内存泄露可疑点）报告分析：

![2021-01-10-tljtS3](https://image.ldbmcs.com/2021-01-10-tljtS3.jpg)

查看信息初步怀疑与J2EEsession超时检查有关，并涉及Hashtable实例的system class loader类，进一步通过Dominator Tree（支配树）分析定位：

![2021-01-10-6NaJ1p](https://image.ldbmcs.com/2021-01-10-6NaJ1p.jpg)

发现有804860条session数存在HashTable中无法被清除，导致无法回收，让开发参与进来确认分析。

![2021-01-10-uCBb3K](https://image.ldbmcs.com/2021-01-10-uCBb3K.jpg)

对可疑点（2）分析发现这1.8G也与可疑点（1）状况类似，均是存在Hash Table中一直积累而未清除。

**根本原因及解决方案**：造成J2EESessionTimeoutChecker中HashTable占用内存过大的原因是在低版本的NetWeaver中session管理不够完善，历史session无法被清除，累积在HashTable中，同时与session相关的信息也被存储在char类型的数组中，无法被回收。建议对NetWeaver进行升级从而完善session管理，避杜绝此类问题再次出现。