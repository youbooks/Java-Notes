> 转载：[What is Garbage collection log, Thread dump, Heap dump?](https://blog.fastthread.io/2020/10/15/what-is-garbage-collection-log-thread-dump-heap-dump/)

![2020-11-03-3STeoD](https://image.ldbmcs.com/2020-11-03-3STeoD.jpg)

Java Virtual Machine (JVM) generates 3 critical artifacts that are useful for optimizing the performance and troubleshooting production problems. Those artifacts are:

1. Garbage collection (GC) log
2. Thread Dump
3. Heap Dump

In this article let us try to understand these 3 critical artifacts, where to use them, how does it look, how to capture them, how to analyze them and their differences.

## 1. Garbage Collection Log

### 1.1 What is GC Log? 

GC Log contains garbage collection events related information. It will indicate how many GC events ran, what type of GC events they are (i.e. Young GC or Full GC), how long did each GC event pause the application, how much objects did each GC event reclaim. 

### 1.2 How does GC Log look?

Sample garbage collection log file can be [found here. ](https://tier1app.com/dist/sample/g1-repeatedGC.txt)

### 1.3 Where is GC Log used?

Garbage collection logs are used to study the application’s GC and memory performance. It’s used to optimize the GC pause times, it’s used to identify optimal memory size for your application, it’s also used to troubleshoot memory related problems.

### 1.4 How to generate GC Log?

You can generate garbage collection log, by passing following JVM arguments:

For Java versions until 8:

```bash
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:<file-path>
```

For Java version starting from 9:

```bash
-Xlog:gc*:file=<file-path>
```

file-path: is the location where Garbage Collection log file will be written.

### 1.5 How to understand GC Log?

Garbage collection log format varies depending on who is your JVM vendor (Oracle, HP, IBM, Azul, ..), Java version (1.5, 5, 6, 7, 8, 9, 10, 11, 12,…), garbage collection algorithm (Serial, Parallel, CMS, G1, Shenandoah, Z GC) and JVM arguments you pass. Thus there is not one standardized format available. However here is a [video tutorial](https://www.youtube.com/watch?v=6G0E4O5yxks), which attempts to help you to understand the GC log file format.

### 1.6 What Tools are used to analyze GC Log?

There are multiple Garbage collection log analysis tools. Some of the popular ones are given here: [GCeasy](https://gceasy.io/), [IBM GC & Memory visualizer](https://developer.ibm.com/javasdk/tools/), [HP JMeter](https://h20392.www2.hpe.com/portal/swdepot/displayProductInfo.do?productNumber=HPJMETER), [Google Garbage Cat](https://code.google.com/archive/a/eclipselabs.org/p/garbagecat)

## 2. Thread dump

### 2.1 What is Thread dump? 

Thread dump is snapshot of all threads running in the application at point in time. It contains all the information about each thread in the application such as: thread state, thread Id, native Id, thread name, stack trace, priority.

### 2.2 How does Thread dump look?

Sample thread dump can be [found here](https://tier1app.com/dist/sample/threaddump_QC1-031214.txt).

### 2.3 Where is Thread dump used?

Thread dumps are primarily used for troubleshooting production problems such as CPU spikes, unresponsiveness in the application, poor response time, hung threads, high memory consumption.

### 2.4 How to generate Thread dump?

Thread dumps can be captured from the running application using [8 different options](https://blog.fastthread.io/2016/06/06/how-to-take-thread-dumps-7-options/). Most common option to take thread dump is to use the ‘jstack’ tool. jstack tool is shipped in JDK_HOME\bin folder. Here is the command that you need to issue to capture thread dump:

```bash
jstack -l  <pid> > <file-path>
```

where

pid: is the Process Id of the application, whose thread dump should be captured

file-path: is the file path where thread dump will be written in to. 

### 2.5  How to understand Thread dump?

Here is a [video talk](https://www.youtube.com/watch?v=wbcT7sa15Lo), that gives good detailed overview on how to understand the thread dumps. 

### 2.6 What Tools are used to analyze Thread dump?

Here are the most widely used thread dump analysis tools: [fastThread](https://fastthread.io/), [Samurai](https://github.com/yusuke/samurai), [IBM Thread & Monitor analyzer](https://developer.ibm.com/javasdk/tools/), [Visual VM](https://visualvm.github.io/)

## 3. Heap dump

### 3.1 What is Heap dump? 

Heap dump is a snapshot of your application’s memory in a point in time. It contains information such as what are the objects in memory, what values do they carry, what is their size, what other objects do they reference. 

### 3.2 How does Heap dump look?

Sample heap dump can be [found here](https://tier1app.com/dist/sample/small-hd.bin). (Note: It is going to be in binary format. So you actually can’t read it).

### 3.3 Where is Heap dump used?

Heap dumps are primarily used for troubleshooting memory related, OutOfMemoryError problems.

### 3.4 How to generate Heap dump?

Heap dump can be captured from the running application using [7 different options](https://blog.heaphero.io/2017/10/13/how-to-capture-java-heap-dumps-7-options/). Most common option to take heap dump is to use the ‘jmap’ tool. jmap tool is shipped in JDK_HOME\bin folder. Here is the command that you need to issue to capture:

```bash
jmap -dump:format=b,
file=<file-path> <pid> 
```

where

pid: is the Java Process Id, whose heap dump should be captured

file-path: is the file path where heap dump will be written in to.

### 3.5 How to understand Heap dump?

Heap dump files are in binary format and tend to be large in size. Besides that, their format heavily lacks documentation. Thus, you have to use the heap dump analysis tools (given in next question) to analyze and understand them.

### 3.6 What Tools are used to analyze Heap dump?

 Here are the most widely used heap dump analysis tools: [Eclipse MAT](https://www.eclipse.org/mat/), [HeapHero](https://heaphero.io/), [JVisualVM](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jvisualvm.html).

