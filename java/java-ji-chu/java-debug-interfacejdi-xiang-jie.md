# Java Debug Interface\(JDI\)详解

> [使用 Java Debug Interface（JDI）调试多线程应用程序](https://www.ibm.com/developerworks/cn/java/j-lo-jdi/index.html)

多线程环境下的程序调试是让开发者头痛的问题。在 IDE 中通过添加断点的方式调试程序，往往会因为停在某一条线程的某个断点上而错失了其他线程的执行，线程之间的调度往往无法预期，并且会因为断点影响了实际的线程执行顺序。因此，在调试多线程程序时，开发者往往会选择打印 `Trace Log` 的方式来帮助调试。

使用 Log 来帮助调试的问题在于，开发者往往无法预期哪些关键点需要记录，于是在整个程序的调试过程中，需要不断的加入 Log 调用，编译生成可执行程序并部署，这对于大尺寸的软件开发项目无疑是噩梦，会直接影响到开发效率。

有没有一种办法，可以独立于程序代码，能在运行期间绑定到程序上并获取程序运行过程当中的关键信息呢？更重要的，这种方法应该是可定制的，开发者可以通过少量的努力，就可以达到特定的调试目的。答案是肯定的。**通过使用 Java Debug Interface（JDI），开发者可以快速开发定制出适用于自己的线程 Profiling 工具**。这样的工具独立于主程序，并且可高度定制。在接下来的文章中，我们将介绍如何实现该工具。

## 1. 认识 JPDA 和 JDI

从 J2SE 1.3 开始，Java 开始提供了一套叫做 `Java Platform Debugger Architecture（JPDA）` 的架构，开发者可以通过这套架构来开发调试用程序。这套架构被主流的 Java IDE（如 Eclipse、NetBeans 等）广泛地采用。

具体来说，JPDA 不仅仅是一套 API 的组合，也不只是一个具体的工具。这套架构提供了从目标程序、调试双方的信息协议，到供开发者使用的结构调用，都一一做出了定义。在 J2SE 5.0 中，它由三个部分组成：

1. Java Virtual Machine Tools Interface（JVMTI），是一套低级别的 native 接口。它定义了 Java 虚拟机所必需为调试提供的服务接口。JVMTI 在 Java 5.0 之前的前身是 JVMDI（Jave Virtual Machine Debug Interface）。
2. Java Debug Wire Protocol（JDWP），定义了调试双方信息和请求的文本格式。
3. Java Debuger Interface（JDI），定义了代码级别的调试接口。

从开发者的角度来看，**调试工具的开发既可以基于 JVMTI 也可以基于 JDI**。JVMTI 是 native 接口，使用起来相对复杂，并且需要 C 语言的基础，因此，在本文中，我们将介绍如何使用 JDI 这种最上层的方式来开发 Java 调试程序。

> 更多关于 JPDA 的详细介绍，可以参见 [JPDA 官方文档](http://java.sun.com/javase/6/docs/technotes/guides/jpda/index.html) 以及 [“深入 Java 调试体系”系列文章](http://www.ibm.com/developerworks/cn/views/java/libraryview.jsp?search_by=深入+Java+调试体系)。

## 2. 需求分析

在接下的部分，我们将介绍如何使用 JDI 来开发一个用来调试多线程程序的工具。在开始前，让我们先列出这个工具需要满足的功能：

1. **独立于目标应用程序的**。
2. **应该足够简单**，并且能在通过少量的代码修改就能完成集中配置，这样是帮助开发者不需要付出太多的努力就能开始调试自己的多线程程序。
3. **能够抓取足够的信息**，比如说异常的信息，程序调用过程中的变量值等等。
4. **所生成的 Log 应该足够清晰**，能够按不同的线程来分离记录，而不是按照时间的顺序来生成每一条记录，否则会给调试带来不便。

## 3. 实现

在文章最后的 [示例代码](https://www.ibm.com/developerworks/cn/java/j-lo-jdi/index.html#artdownload) 中，我们展示了一个典型的基于 JDI 的调试工具逻辑，并且用它来 Profile 一个简单的多线程程序的执行。根据前面所提到的需求，代码展示了**线程运行栈快照、方法调用的入口参数值收集、异常过滤定制、类过滤配置、线程 Log 记录**等功能。具体来说：

1. 独立于目标程序

   分析工具可以通过如下方式启动：

   `java Trace options class args`

   支持的 `options` 参数：

   `-output` 文件名：工具生成的 Log 的路径

   `class` 是目标程序的入口类，`args` 为目标程序的输入参数

2. 简洁配置
   1. 异常过滤配置：

      您可以在 `ExceptionConfig.properties` 属性文件中配置所需记录异常类型。在 Demo 代码中配置了对于 `NullPointerException` 和 `UserDefinedException` 两种异常，分析工具将追踪这两种异常情况。

      `ExceptionName = exceptions.UserDefinedException;java.lang.NullPointerException`

   2. 类过滤配置：

      您可以在 `ClassExcludeConfig.properties` 属性文件中配置被过滤的类模式，分析工具将不会处理被过滤类的任何事件。

      `ExcludedClassPattern=java.*;javax.*;sun.*;com.sun.*;com.ibm.*`
3. 运行

   在目标的主程序的生命周期中，分析器完成以下操作：

   1. 绑定，分析工具和目标调试程序的虚拟机实例绑定；
   2. 事件注册，分析工具向虚拟机实例注册相关事件请求，整个分析过程采取基于事件驱动的模式。
   3. 线程运行时信息挖掘。
   4. 分类信息生成。

   以上四点操作满足了需求：通过采用绑定机制实现调试程序和工具程序的独立，分析工具和目标程序以监听端口、共享内存等方式进行通信，无须目标程序进行任何代码修改即可实现调试。采用基于事件的机制可以帮助开发者依据实际需要集中注册和处理事件。作为基础框架，分析工具注册了支持异常、执行流程等事件，并提供了异常时运行栈快照，方法进出参数记录等功能实现信息抓取。支持单线程为单位的 Log 记录，将开发者从无序不可预测的多线程执行中摆脱出来，对调试程序提供帮助。

下面将详细阐述实现步骤：

### 3.1 绑定

JDI 支持四种对目标程序的绑定方式，分别为：

1. 分析器启动目标程序虚拟机实例
2. 分析器绑定到已运行的目标程序虚拟机实例
3. 目标程序虚拟机实例绑定到已运行的分析器
4. 目标程序虚拟机实例启动分析器

JDI 支持一个分析器绑定多个目标程序，但一个目标程序只能绑定一个分析器。为支持以上绑定，JDI 对应有 `LaunchingConnector`，`AttachingConnector` 和 `ListeningConnector`，具体类介绍可以参照 [文档](http://java.sun.com/j2se/1.5.0/docs/guide/jpda/jdi/com/sun/jdi/connect/package-frame.html)。

本文采用第一种绑定方式阐述如何开发定制的多线程分析器，其它绑定方式可以参照 [文档](http://java.sun.com/javase/6/docs/technotes/guides/jpda/conninv.html)。

绑定过程分为三个步骤：

1. 获取连接实例

   ```java
   LaunchingConnector findLaunchingConnector() {
       List connectors = Bootstrap.virtualMachineManager().allConnectors();
       Iterator iter = connectors.iterator();
       while (iter.hasNext()) {
           Connector connector = (Connector) iter.next();
           if ("com.sun.jdi.CommandLineLaunch".equals(connector.name())) {
               return (LaunchingConnector) connector;
           }
   　　}
   }
   ```

   `Bootstrap.virtualMachineManager().allConnectors()` 返回所有已知的 Connector 对象实例。选择返回 `com.sun.jdi.CommandLineLaunch` 连接实例，表示使用第一种绑定方式。

2. 设置连接参数

   ```java
   /**参数：
   * connector为清单1.中获取的Connector连接实例
   * mainArgs为目标程序main函数所在的类
   **/
   Map connectorArguments(LaunchingConnector connector, String mainArgs) { 
       Map arguments = connector.defaultArguments();   
       Connector.Argument mainArg = (Connector.Argument) arguments.get("main");    
       if (mainArg == null) {      
           throw new Error("Bad launching connector"); 
       }
       mainArg.setValue(mainArgs); 
       return arguments;
   }
   ```

   每个连接实例都有对应的默认参数，启动连接之前需要设置必须的参数，对于 `CommandLineLaunch` 连接实例需要设置主程序启动目标程序虚拟机实例所需的参数。

3. 启动连接，获取目标程序虚拟机实例

   ```java
   /**参数：
   * mainArgs为目标程序main函数所在的类
   **/
   VirtualMachine launchTarget(String mainArgs) {
       //findLaunchingConnector：获取连接
       LaunchingConnector connector = findLaunchingConnector();
       //connectorArguments：设置连接参数
       Map arguments = connectorArguments(connector, mainArgs);
       try {       
           return connector.launch(arguments);//启动连接   
       } catch (IOException exc) {     
           throw new Error("Unable to launch target VM: " + exc);  
       } catch (IllegalConnectorArgumentsException exc) {
           throw new Error("Internal error: " + exc);  
       } catch (VMStartException exc) {
           throw new Error("Target VM failed to initialize: " + exc.getMessage());
       }
   }
   ```

   1和2分别获取连接实例和启动所需的变量，通过调用 `connector.launch(arguments)`启动连接，实现了分析器和目标程序的绑定。

### 3.2 注册事件

分析器和目标程序之间采用基于事件的模式进行通信。分析器向虚拟机实例注册所关注的事件。事件发生时，虚拟机将相关事件信息放入事件队列中，采用 [生产者 - 消费者](http://en.wikipedia.org/wiki/Producer-consumer_problem) 的模式与分析器同步。

1. 注册事件

   EventRequestManager 管理事件请求，它支持创建、删除和查询事件请求。EventRequest 支持三种挂起策略：

   * EventRequest.SUSPEND\_ALL : 事件发生时，挂起所有线程
   * EventRequest.SUSPEND\_EVENT\_THREAD : 事件发生时，挂起事件源线程
   * EventRequest.SUSPEND\_NONE : 事件发生时，不挂起任何线程

   JDI 支持多种类型的 EventRequest，如 ExceptionRequest，MethodEntryRequest，MethodExitRequest，ThreadStartRequest 等，可以参考 [文档](http://java.sun.com/j2se/1.5.0/docs/guide/jpda/jdi/com/sun/jdi/request/package-frame.html)。

   ```java
   EventRequestManager mgr = vm.eventRequestManager(); 
   // 注册异常事件
   ExceptionRequest excReq = mgr.createExceptionRequest(null, true, true); 
   excReq.setSuspendPolicy(EventRequest.SUSPEND_EVENT_THREAD); 
   excReq.enable(); 
   // 注册进方法事件
   MethodEntryRequest menr = mgr.createMethodEntryRequest(); 
   menr.setSuspendPolicy(EventRequest.SUSPEND_NONE); 
   menr.enable(); 
   // 注册出方法事件
   MethodExitRequest mexr = mgr.createMethodExitRequest(); 
   mexr.setSuspendPolicy(EventRequest.SUSPEND_NONE); 
   mexr.enable(); 
   // 注册线程启动事件
   ThreadStartRequest tsr = mgr.createThreadStartRequest(); 
   tsr.setSuspendPolicy(EventRequest.SUSPEND_EVENT_THREAD); 
   tsr.enable(); 
   // 注册线程结束事件
   ThreadDeathRequest tdr = mgr.createThreadDeathRequest(); 
   tdr.setSuspendPolicy(EventRequest.SUSPEND_EVENT_THREAD); 
   tdr.enable();
   ```

2. 分析器从事件队列中获取事件

   EventQueue 用来管理目标虚拟机实例的事件，事件会被加入 EventQueue 中。分析器调用 EventQueue.remove\(\)，如果事件队列中存在事件，则返回不可修改的 EventSet 实例，否则分析器会被挂起直到有新的事件发生。处理完 EventSet 中的事件后，调用其 resume\(\) 方法唤醒 EventSet 中所有事件发生时可能挂起的线程。

   ```java
   public void run() { 
       EventQueue queue = vm.eventQueue(); 
       while (connected) { 
           try { 
               EventSet eventSet = queue.remove(); 
               EventIterator it = eventSet.eventIterator(); 
               while (it.hasNext()) { 
                   handleEvent(it.nextEvent()); 
               } 
               eventSet.resume(); 
           } catch (InterruptedException exc) {// Ignore 
           } catch (VMDisconnectedException discExc) { 
               handleDisconnectedException(); 
               break; 
           } 
       } 
   }
   ```

### 3.3 获取多线程执行信息

执行流程和变量信息是调试程序最重要的两方面。无论是通过 IDE 设置断点的调试方式，还是通过在程序中记 Log 的调试方式，它们的主要目的是向开发者提供以上两方面信息。本文分析器以单个线程为单位，来记录线程运行信息：

1. 执行流程。分析器以方法作为最小颗粒度单位。分析器按照实际的线程执行顺序记录方法进出。
2. 变量值。对于单个方法而言，其程序逻辑固定，方法的输入值决定了方法内部执行流程。分析器将在方法入口和出口分别记录该方法作用域内可见变量，便于开发者调试。
3. 执行栈信息记录。当异常发生时，执行栈中完好地保存了调用帧信息。分析器获取线程栈中的所有帧，并记录每个帧记录的信息，其中包含可见变量值、帧调用名称等信息。StackFrame 中变量信息的获取也是 JDI 所提供的特殊能力之一。

   > 关于帧的详细介绍，可以参见 [这里](http://java.sun.com/javase/6/docs/jdk/api/jpda/jdi/com/sun/jdi/StackFrame.html)。

与 IDE 设置断点的方法相比，提供的数据信息量相当，但分析器提供执行流程信息更加的清晰；与在程序中记录 Log 的方式相比，分析器在执行流程和信息量两方面都胜出。

以下将详细介绍上面三方面信息抓取：

1. 线程执行流程

   线程执行流程可划分：线程启动→ run\(\) →进入方法→ ... →退出方法→线程结束。通过向虚拟机实例注册 `ThreadStartRequest`，`MethodEntryRequest`，`MethodExitRequest` 和 `ThreadDeathRequest` 事件的方式记录执行过程。事件注册详细见清单 4，清单 6 列出分析器对于以上事件的处理方法。

   ```java
   // 清单 6. 获取执行流程
   void threadStartEvent(ThreadStartEvent event) { 
       println("Thread " + event.thread().name() + " Start"); 
   } 
   void methodEntryEvent(MethodEntryEvent event) { 
       println("Enter Method:" + event.method().name() + "  --  "
           + event.method().declaringType().name()); 
       // 进入方法记录可见变量值
       this.printVisiableVariables();
   } 
   void methodExitEvent(MethodExitEvent event) {
       println("Exit Method:" + event.method().name() + "  --  "
           + event.method().declaringType().name());
       // 退出方法记录可见变量值
       this.printVisiableVariables();
   } 
   void threadDeathEvent(ThreadDeathEvent event) {
       println("Thread " + event.thread().name() + " Dead");
   }
   ```

2. 可见变量信息抓取

   ```java
   // 清单 7. 可见变量信息抓取
   private void printVisiableVariables()
   {
       try{
           this.thread.suspend();
           if(this.thread.frameCount()>0)   {
               //获取当前方法所在的帧
               StackFrame frame = this.thread.frame(0);
               List<LocalVariable> lvs = frame.visibleVariables();
               for (LocalVariable lv : lvs) {
                   println("Name:" + lv.name() + "\t" + "Type:"
                       + lv.typeName() + "\t" + "Value:"
                       + frame.getValue(lv));
               }

           }   
       } catch(Exception e){//ignore}
       finally{this.thread.resume();}
   }
   ```

   通过 `this.thread.frame(0)` 获取当前方法对应的帧，调用 `frame.visibleVariables()` 取出当前方法帧的所有可见变量。

3. 异常时线程栈快照

   ```java
   // 清单 8. 异常事件线程栈快照
   private void printStackSnapShot() {
       try {
           this.thread.suspend();
           //获取线程栈
           List<StackFrame> frames = this.thread.frames();
           //获取线程栈信息
           for (StackFrame frame : frames) {
               if (frame.thisObject() != null) {
                   //获取当前对象应该的所有字段信息
                   List<Field> fields = frame.thisObject().referenceType().allFields();
                   for (Field field : fields) {
                       println(field.name() + "\t" + field.typeName()+ "\t"
                           + frame.thisObject().getValue(field));
                   }
               }
               //获取帧的可见变量信息
               List<LocalVariable> lvs = frame.visibleVariables();
               for (LocalVariable lv : lvs) {
                   println(lv.name() + "\t" + lv.typeName() + "\t" 
                       + frame.getValue(lv));
               }
           }
       } catch (Exception e) {}
       finally { this.thread.resume();}
   }
   ```

   通过 `this.thread.frames()` 获取异常发生时线程栈中所有帧信息，调用 `frame.thisObject()` 获取对 this 指针的引用，进而获取对象字段信息；对于帧信息的抓取与清单 7 类似。

### 3.4 分类信息生成 Log

以单线程为记录单元是分析器的特点，下面将从分析器 Log 实现结构、目标程序所模拟的场景及分析结果三方面对示例代码进行介绍。

1. 分析器 Log 实现结构

   Trace 为分析器入口类，它负责创建绑定连接，生成目标程序虚拟机实例；`EventThread` 负责从虚拟机实例的事件队列中获取事件，交由对应的 `ThreadTrace` 处理，它同时维护着一张 `ThreadReference` 和 `ThreadTrace` 一一对应关系的映射表；`ThreadTrace` 负责分析 `ThreadReference` 信息，并将结果记录在 `logRecord` 的缓存中，每个 `ThreadTrace` 实现了单个线程信息的追踪，详见图 1。

   ![2020-08-03-I6yWkh](https://image.ldbmcs.com/2020-08-03-I6yWkh.jpg)

2. 目标程序

   目标程序由两个核心类组成：`MainThread` 和 `CounterThread`。`MainThread` 是程序的主类，它负责启动两个 CounterThread 线程实例并抛出两类异常：用户自定义异常 `UserDefinedException` 和运行时异常 `NullPointerException`；`CounterThread` 是一个简单的计数线程。整个目标程序模拟的是多线程和异常的环境。

3. 分析结果

   Log 依照目标程序的调用层次进行缩进，清晰地展现每个线程的执行逻辑和变量信息，详见清单 9。为了方便理解，我们在 log 中加入了注释。

   ```java
   // 清单 9. Log
   -- VM Started -- 
      ====== main ====== 
           Enter Method:main// 
               Enter Method:<init>//MainThread 构造函数
                   a      int      0 
                   b      int      0 
                   c      int      0 
               Exit Method:<init> 
               Enter Method:makeABusinessException//makeABusinessException 方法调用
                   a      int      0 
                   b      int      1 
                   c      int      2 
                   Enter Method:<init>//UserDefinedException 构造函数
                       ... 
                   Exit Method:<init> 
    //UserDefinedException 异常发生，抓取线程栈中所有帧信息
                   exceptions.UserDefinedException(id=62) catch: MainThread:30 
                       Frame(MainThread:44)     
                           a      int      0 
                           b      int      1 
                           c      int      2 
                           i      int      0 
                           d      int      4 
                       Frame(MainThread:23) 
                              e      int      4 
                           g      int      5 
                           mt      MainThread      instance of MainThread(id=59) 
                           i      int      0 
                     // NullPointerException 异常发生，抓取线程栈信息
               java.lang.NullPointerException(id=70) catch: MainThread:30 
                              ... 
    // 以下是两个 CounterThread 线程的构造
               Enter Method:<init> 
                   name      java.lang.String      null 
                   index      int      0 
               Exit Method:<init> 
               Enter Method:<init> 
                   name      java.lang.String      null 
                   index      int      0 
               Exit Method:<init> 
           Exit Method:main 
      ====== main end ====== 
      ====== Thread-1 ====== 
           Enter Method:run//run 方法调用
               name      java.lang.String     "thread1"
               index      int      0 
    // 以下是 3 次 updateIndex 方法调用
               Enter Method:updateIndex 
                   name      java.lang.String     "thread1"
                   index      int      0 
               Exit Method:updateIndex 
               Enter Method:updateIndex 
                   name      java.lang.String     "thread1"
                   index      int      2 
               Exit Method:updateIndex 
               Enter Method:updateIndex 
                   name      java.lang.String     "thread1"
                   index      int      4 
               Exit Method:updateIndex 
           Exit Method:run 
      ====== Thread-1 end ====== 
      ====== Thread-2 ====== 
             Enter Method:run//run 方法调用
               name      java.lang.String     "thread2"
               index      int      0 
    // 以下是 3 次 updateIndex 方法调用
               Enter Method:updateIndex 
                   name      java.lang.String     "thread2"
                   index      int      1 
               Exit Method:updateIndex 
               Enter Method:updateIndex 
                   name      java.lang.String     "thread2"
                   index      int      3 
               Exit Method:updateIndex 
               Enter Method:updateIndex 
                   name      java.lang.String     "thread2"
                   index      int      5 
               Exit Method:updateIndex 
           Exit Method:run 
      ====== Thread-2 end ======
   ```

## 4. 结语

当开发多线程程序时，至少有两个理由让你选择 JDI 来协助调试：

1. 线程执行的时序变得越来越不可预测，在 IDE 中通过添加断点来调试的方法已经不能正确地反映程序运行状况。
2. 程序规模大，每一次 trace 语句的添加都会造成程序的再编译，而这样的编译需要花上很多时间。

因此，使用 JDI 开发自己的调试程序，有时会为开发者节省更多的时间。通过本文的介绍和示例代码的解读，读者可以着手开发自己的多线程调试程序了。

## 5. 下载资源

* [本文示例代码](https://www.ibm.com/developerworks/cn/java/j-lo-jdi/DemoCode.zip) \(DemoCode.zip \| 36 KB\)

## 6. 相关主题

* 在 [JPDA 官方的主页](http://java.sun.com/javase/technologies/core/toolsapis/jpda/)上，您可以查看关于 JPDA 标准实现的细节。
* 您可以参与 [JPDA 官方论坛](http://forums.sun.com/forum.jspa?forumID=543)上的参与。
* 参看文章“[使用 Eclipse 远程调试 Java 应用程序](http://www.ibm.com/developerworks/cn/opensource/os-eclipse-javadebug/)”，了解 JDI 是如何应用到 Eclipse 的调试界面上。
* 参考“[Inside the Java Virtual Machine](http://www.artima.com/insidejvm/ed2/)”，了解更多 Java 虚拟机的机制。
* [developerWorks Java 技术专区](http://www.ibm.com/developerworks/cn/java/)：查找数百篇有关 Java 编程各方面的文章。

