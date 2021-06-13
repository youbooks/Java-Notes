https://lulaoshi.info/flink/chapter-system-design/flink-core.html

## 核心组件

主从架构
    - Master：主进程，协调管理。
    - Task Manager：执行计算任务的进程，Flink将多个计算任务分发给多个Task Manager上执行。

## Flink作业提交过程

![](https://image.ldbmcs.com/2021-06-09-q3huJH.jpg)

在一个作业提交前，Master和TaskManager等进程需要先被启动。我们可以在Flink主目录中执行脚本来启动这些进程：`bin/start-cluster.sh`。Master和TaskManager被启动后，TaskManager需要将自己`注册`给Master中的ResourceManager。这个初始化和资源注册过程发生在单个作业提交前，我们称之为第0步。

接下来我们根据上图，逐步分析一个Flink作业如何被提交：

1. 用户编写应用程序代码，并通过Flink客户端（Client）提交作业。程序一般为Java或Scala语言，调用Flink API，构建逻辑视角数据流图。代码和相关配置文件被编译打包，被提交到Master的Dispatcher，形成一个应用作业（Application）。

2. Dispatcher接收到这个作业，启动JobManager，这个JobManager会负责本次作业的各项协调工作。

3. JobManager向ResourceManager申请本次作业所需资源。

4. 由于在第0步中TaskManager已经向ResourceManager中注册了资源，这时闲置的TaskManager会被反馈给JobManager。

5. JobManager将用户作业中的逻辑视图转化为图所示的并行化的物理执行图，将计算任务分发部署到多个TaskManager上。至此，一个Flink作业就开始执行了。

6. TaskManager在执行计算任务过程中可能会与其他TaskManager交换数据，会使用图中的一些数据交换策略。同时，TaskManager也会将一些任务状态信息会反馈给JobManager，这些信息包括任务启动、运行或终止的状态，快照的元数据等。
