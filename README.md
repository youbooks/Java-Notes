![2020-08-26-BackendDeveloper](https://image.ldbmcs.com/2020-08-26-Backend Developer.png)

# Backend Developer

## 计算机基础

### 操作系统

- Linux
- 计算机原理
- CPU

	- 多级缓存

		- L1
		- L2
		- L3
		- 主内存

- 进程
- 线程

### 网络编程

- 网络模型

	- 为什么要分层？

		- 简化问题难度和复杂度
		- 灵活性好
		- 易于实现和维护
		- 促进标准化工作

	- 数据包的封装和分用
	- OSI七层网络模型(由下到上)(复杂又不实用)

		- 媒介层(硬件)

			- 物理层
			- 数据链路层
			- 网络层

		- 主机层(软件)

			- 传输层
			- 会话层
			- 表示层
			- 应用层

	- TCP/IP四层网络模型(由上到下)

		- 应用层

			- HTTP

				- 请求方式

					- GET：数据的读取，幂等
					- POST：向指定资源提交数据，非幂等
					- PUT：向指定资源位置上传其最新内容，幂等
					- DELETE：请求服务器删除所请求URI所标识的资源，幂等

				- 长连接 vs 短连接
				- HTTP vs HTTPS
				- 请求报文、响应报文

					- 请求报文

						- 请求行
						- 请求头部
						- 空行
						- 请求主体

					- 响应报文

						- 状态行

							- 状态码

								- 1XX(信息性状态码)：接收的请求正在处理
								- 2XX(成功状态码)：请求正常处理完毕
								- 3XX(重定向状态码)：需要进行附加操作以完成请求
								- 4XX(客户端错误状态码)：服务器无法处理请求
								- 5XX(服务器错误状态码)：服务器请求处理出错

						- 响应头部
						- 空行
						- 响应主体

				- HTTP 1.1 vs 2.0

			- SSH
			- FTP
			- WebSocket
			- P2P协议
			- DNS协议

			  用于 TCP/IP 网络，它从事将主机名或域名转换为实际 IP 地址的工作。

			- ARP协议

			  用于实现从 IP 地址到 MAC 地址的映射。

		- 传输层

			- UDP协议

				- 特点

					- 数据可能丢失，顺序传输无法保证
					- 无状态，不需要像TCP那样要建立连接
					- 没有拥塞控制，来一个包就发一个

				- 使用场景

					- 需要资源少，在网络情况比较好的内网，或者对对包不敏感的场合
					- 广播场景，不需要一对一建立连接
					- 需要时延低，允许丢包，不关注网络拥塞的场景

			- TCP协议

				- 特点

					- 数据的顺序传输
					- 丢包重传，保证可靠
					- 连接维护
					- 流量控制，保证稳定
					- 拥塞控制，及时调整，最大程度保证传输正常进行

				- 连接管理

					- TCP三次握手
					- TCP四次挥手

				- 数据传输

					- 如何保证可靠传输：ACK+序列号
					- 流量控制与窗口管理
					- 超时重传机制
					- 流量控制

						- ARQ协议
						- 滑动窗口

					- 拥塞控制

						- 慢开始
						- 拥塞避免
						- 快重传
						- 快恢复

		- 网络层

			- IP协议

				- 不可靠的传输协议
				- IP协议是无连接的

			- ICMP协议

			  ICMP并不为IP网络提供可靠性，它只是用于反馈各种故障和配置信息。丢包不会触发ICMP。

				- ping程序

			- 路由器

				- 三层设备

		- 链路层

			- MAC地址

			  MAC地址就是适配器地址或适配器标识符，当适配器插入到某台计算机之后，适配器上的标识符就成为这台计算机的MAC地址了。

			- ARP协议

			  ARP为IP地址到硬件地址之间提供了动态映射。

			- 交换机

				- 二层设备

			- IP地址 vs MAC地址

			  MAC地址就好像是我们的身份证，IP就像是我们的住址，可以根据住址寄送快递，但是不能根据身份证号码寄快递，别人不知道怎么走呢。

### 编译原理

### 安全

- WEB安全

	- DDos攻击

		- 减少SYN timeout时间
		- 限制同时打开的SYN半连接数目

	- XSS攻击
	- SQL注入攻击

- 授权、认证

	- Apache Shiro
	- Spring Security

- 加密算法

	- 哈希算法

		- MD
		- SHA

	- 对称加密

		- DES
		- AES
		- IDEA

	- 非对称加密

		- RSA

	- HTTPS-SSL

## 数据结构与算法

### 数据结构

- 数组
- 链表
- 栈
- 队列
- 散列表
- 树

	- 二叉树
	- 平衡二叉树
	- 二叉查找树
	- 红黑树
	- B，B+，B*树

- 图

### 算法

- 查找算法
- 排序算法
- 数组系列
- 链表系列
- 动态规划系列
- 字符串系列
- 二叉树系列
- 回溯系列

## Java

### Java 基础

- 数据类型

	- 基本类型

		- byte/8
		- char/16
		- short/16
		- int/32
		- float/32
		- long/64
		- double/64
		- boolean/~

	- 包装类型
	- 缓存池

- String

	- 不可变
	- String, StringBuffer and StringBuilder
	- String Pool

- Object 通用方法

	- equals()
	- hashCode()
	- toString()
	- clone()

- 抽象类与接口
- Java反射

	- 动态语言
	- 反射机制概念
	- 反射的应用场合
	- Java 反射 API
	- 反射使用步骤(获取 Class 对象、调用对象方法)
	- 获取 Class 对象的 3 种方法
	- 创建对象的两种方法

- Java异常分类及处理

	- 概念
	- 异常分类

		- Error
		- Exception(RuntimeException、CheckedException)

			- RuntimeException
			- CheckedException

	- 异常的处理方式

		- 抛出异常有三种形式，一是 throw,一个 throws，还有一种系统自动抛异常。
		- Throw 和 throws 的区别

- Java泛型

	- 泛型方法(<E>)
	- 泛型类<T>
	- 类型通配符?
	- 类型擦除

- Java注解

	- 概念
	- 4 种标准元注解
	- 注解处理器

- Java内部类

	- 静态内部类
	- 成员内部类
	- 局部内部类(定义在方法中的类)
	- 匿名内部类(要继承一个父类或者实现一个接口、直接使用 new 来生成一个对象的引用)

- Java序列化

	- 保存(持久化)对象及其状态到内存或者磁盘
	- 序列化对象以字节数组保持-静态成员不保存
	- 序列化用户远程对象传输
	- Serializable 实现序列化
	- ObjectOutputStream 和 ObjectInputStream 对对象进行序列化及反序列化
	- writeObject 和 readObject 自定义序列化策略
	- 序列化 ID
	- 序列化并不保存静态变量
	- 序列化子父类说明
	- Transient 关键字阻止该变量被序列化到文件中

- Java复制

	- 直接赋值复制
	- 浅复制(复制引用但不复制引用的对象)
	- 深复制(复制对象和其应用对象)
	- 序列化(深 clone 一中实现)

- Java IO/NIO

	- 阻塞 I/O
	- 非阻塞 I/O
	- I/O 多路复用
	- 信号驱动 I/O
	- 异步 I/O

### Java 集合

- Collection

  Collection 是集合 List、Set、Queue 的最基本的接口。

	- List

	  List 是有序的 Collection。Java List 一共三个实现类: 分别是 ArrayList、Vector 和 LinkedList。

		- ArrayList

		  当数组大小不满足时需要增加存储能力，就要将已经有数 组的数据复制到新的存储空间中。当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进 行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。

		- Vector

		  Vector 与 ArrayList 一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一 个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此， 访问它比访问 ArrayList 慢。

		- LinkedList

		  LinkedList 是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较 慢。另外，他还提供了 List 接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆 栈、队列和双向队列使用。

	- Set

	  Set 注重独一无二的性质,该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，值不能重复。对象的相等性本质是对象 hashCode 值(java 是依据对象的内存地址计算出的此序号)判断 的，如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方 法。

		- HashSet

		  HashSet 通过 hashCode 值来确定元素在内存中的位置。一个 hashCode 位置上可以存放多个元素。

		- TreeSet

		  TreeSet()是使用二叉树的原理对新 add()的对象按照指定的顺序排序(升序、降序)，每增 加一个对象都会进行排序，将对象插入的二叉树指定的位置。

		- LinkedHashSet

		  继承与 HashSet、又基于 LinkedHashMap 来实现的。 
		  LinkedHashSet 底层使用 LinkedHashMap 来保存所有元素，它继承与 HashSet，其所有的方法 操作上又与 HashSet 相同，因此 LinkedHashSet 的实现上非常简单，只提供了四个构造方法，并 通过传递一个标识参数，调用父类的构造器，底层构造一个 LinkedHashMap 来实现，在相关操 作上与父类 HashSet 的操作相同，直接调用父类 HashSet 的方法即可。

	- Queue

- Map

	- HashMap

		- 数组+链表+红黑树

	- ConcurrentHashMap

		- Segment(ReentrantLock)

	- HashTable

	  Hashtable 是遗留类，很多映射的常用功能与 HashMap 类似，不同的是它承自 Dictionary 类， 并且是线程安全的，任一时间只有一个线程能写 Hashtable，并发性不如 ConcurrentHashMap， 因为 ConcurrentHashMap 引入了分段锁。 
	  
	  Hashtable 不建议在新代码中使用，不需要线程安全 的场合可以用 HashMap 替换，需要线程安全的场合可以用 ConcurrentHashMap 替换

	- TreeMap

	  TreeMap 实现 SortedMap 接口，能够把它保存的记录根据键排序，默认是按键值的升序排序， 也可以指定排序的比较器，当用 Iterator 遍历 TreeMap 时，得到的记录是排过序的。 
	  如果使用排序的映射，建议使用TreeMap。

### Java 并发

- 内存模型

	- 物理计算机

		- 内存模型

			- 一级缓存
			- 二级缓存
			- 三级缓存
			- 主内存

		- 数据缓存不一致

			- 通过总线加LOCK#锁的方式
			- 通过缓存一致性协议

		- CPU(处理器)的乱序执行(out-of-orderexecution)，重排序

	- Java的内存模型

		- 内存模型

			- 工作内存(线程，虚拟机栈)
			- 主内存(堆)

		- 八种内存之间交互
		- 八种原子操作规则

	- 重排序

		- 数据依赖
		- 重排序规则(as-if-serial)

			- 单线程（程序）执行结果不能被改变

	- Java内存模型的需要解决的问题

		- 工作内存的可见性问题

			- volatile

		- 重排序带来的问题

			- happens-before原则
			- synchronized
			- Lock

	- Happens-Before 原则

		- 程序次序规则

			- 在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作

		- 锁定规则

			- 对一个unlock操作先行发生于后面对同一个锁的lock操作。

		- volatlie变量规则

			- 对一个volatile变量的写操作先行发生于后面这个变量的读操作。

		- 线程启动规则

			- Thread对象的start()方法先行发生于此线程的每个动作。

		- 线程终止规则

			- 线程中的所有操作都先行发生于对此线程的终止检测。

		- 线程中断规则

			- 对线程interrupt()方法的调用先与被中断线程的代码检查到中断事件的发生。

		- 对象终结规则

			- 一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。

		- 传递性规则

			- 如果操作A先行与发生于操作B，操作B先行发生于操作C，那么就可以得出A先行发生于操作C的结论。

- 多线程

	- 线程和进程
	- 线程创建

		- 继承 Thread 类

		  启动线程的唯一方 法就是通过 Thread 类的 start()实例方法。start()方法是一个 native 方法，它将启动一个新线程，并执行 run()方法。

		- 实现 Runnable 接口

		  如果自己的类已经 extends 另一个类，就无法直接 extends Thread，此时，可以实现一个 Runnable 接口。

		- ExecutorService、Callable<Class>、Future 有返回值线程

		  有返回值的任务必须实现 Callable 接口，类似的，无返回值的任务必须 Runnable 接口。执行 Callable 任务后，可以获取一个 Future 的对象，在该对象上调用 get 就可以获取到 Callable 任务返回的 Object 了，再结合线程池接口 ExecutorService 就可以实现传说中有返回结果的多线程 了。

	- 线程生命周期

	  当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。 在线程的生命周期中，它要经过新建(New)、就绪(Runnable)、运行(Running)、阻塞 (Blocked)和死亡(Dead)5 种状态。尤其是当线程启动以后，它不可能一直"霸占"着 CPU 独自 运行，所以 CPU 需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换。

		- 新建状态(NEW)

		  当程序使用 new 关键字创建了一个线程之后，该线程就处于新建状态，此时仅由 JVM 为其分配 内存，并初始化其成员变量的值。

		- 就绪状态(RUNNABLE)

		  当线程对象调用了 start()方法之后，该线程处于就绪状态。Java 虚拟机会为其创建方法调用栈和程序计数器，等待调度运行。

		- 运行状态(RUNNING)

		  如果处于就绪状态的线程获得了 CPU，开始执行 run()方法的线程执行体，则该线程处于运行状态。

		- 阻塞状态(BLOCKED)

		  阻塞状态是指线程因为某种原因放弃了 cpu 使用权，也即让出了 cpu timeslice，暂时停止运行。 直到线程进入可运行(runnable)状态，才有机会再次获得 cpu timeslice 转到运行(running)状态。

			-  等待阻塞(o.wait->等待对列)
			-  同步阻塞(lock->锁池)
			- 其他阻塞(sleep/join)

		- 线程死亡(DEAD)

		  线程会以下面三种方式结束，结束后就是死亡状态。

			-  正常结束

			  run()或 call()方法执行完成，线程正常结束。

			-  异常结束

			  线程抛出一个未捕获的 Exception 或 Error。

			-  调用 stop

			  直接调用该线程的 stop()方法来结束该线程—该方法通常容易导致死锁，不推荐使用。

	- 中断线程 

		- 正常运行结束

		  程序运行结束，线程自动结束。

		- 使用退出标志退出线程

		  一般 run()方法执行完，线程就会正常结束，然而，常常有些线程是伺服线程。它们需要长时间的运行，只有在外部某些条件满足的情况下，才能关闭这些线程。

		- Interrupt 方法结束线程

		  使用 interrupt()方法来中断线程有两种情况: 
		  线程处于阻塞状态:如使用了 sleep,同步锁的 wait,socket 中的 receiver,accept 等方法时， 会使线程处于阻塞状态。当调用线程的 interrupt()方法时，会抛出 InterruptException 异常。 阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后 break 跳出循环状态，从而让我们有机会结束这个线程的执行。通常很多人认为只要调用 interrupt 方法线程就会结束，实际上是错的， 一定要先捕获 InterruptedException 异常之后通过 break 来跳出循环，才能正常结束 run 方法。 
		  线程未处于阻塞状态:使用 isInterrupted()判断线程的中断标志来退出循环。当使用interrupt()方法时，中断标志就会置 true，和使用自定义的标志来控制循环是一样的道理。

		- stop 方法终止线程(线程不安全)

		  程序中可以直接使用 thread.stop()来强行终止线程，但是 stop 方法是很危险的 。不安全主要是: 
		  thread.stop()调用之后，创建子线程的线程就会抛出 ThreadDeatherror 的错误，并且会释放子线程所持有的所有锁。

	- 线程基本方法

		- 线程等待(wait)

		  调用该方法的线程进入 WAITING 状态，只有等待另外线程的通知或被中断才会返回，需要注意的是调用 wait()方法后，会释放对象的锁。因此，wait 方法一般用在同步方法或同步代码块中。

		- 线程睡眠(sleep)

		  sleep 导致当前线程休眠，与 wait 方法不同的是 sleep 不会释放当前占有的锁,sleep(long)会导致线程进入 TIMED-WATING 状态，而 wait()方法会导致当前线程进入 WATING 状态。

		- 线程让步(yield)

		  yield 会使当前线程让出 CPU 执行时间片，与其他线程一起重新竞争 CPU 时间片。一般情况下，优先级高的线程有更大的可能性成功竞争得到 CPU 时间片，但这又不是绝对的，有的操作系统对线程优先级并不敏感。

		- 线程中断(interrupt)

		  中断一个线程，其本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标识位。这个线程本身并不会因此而改变状态(如阻塞，终止等)。

		- Join 等待其他线程终止

		  join() 方法，等待其他线程终止，在当前线程中调用一个线程的 join() 方法，则当前线程转为阻塞状态，回到另一个线程结束，当前线程再由阻塞状态变为就绪状态，等待 cpu 的宠幸。

		- 为什么要用 join()方法?

		  很多情况下，主线程生成并启动了子线程，需要用到子线程返回的结果，也就是需要主线程需要在子线程结束后再结束，这时候就要用到 join() 方法。

		- 线程唤醒(notify)

		  Object 类中的 notify() 方法，唤醒在此对象监视器上等待的单个线程，如果所有线程都在此对象上等待，则会选择唤醒其中一个线程，选择是任意的，并在对实现做出决定时发生，线程通过调 
		  用其中一个 wait() 方法，在对象的监视器上等待，直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程，被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞 
		  争。类似的方法还有 notifyAll() ，唤醒再次监视器上等待的所有线程。

		- 其他方法

		  sleep():强迫一个线程睡眠N毫秒。 
		  isAlive(): 判断一个线程是否存活。 
		  join(): 等待线程终止。 
		  activeCount(): 程序中活跃的线程数。 
		  enumerate(): 枚举程序中的线程。 
		  currentThread(): 得到当前线程。 
		  isDaemon(): 一个线程是否为守护线程。 
		  setDaemon(): 设置一个线程为守护线程。(用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束) 
		  setName(): 为线程设置一个名称。 
		  wait(): 强迫一个线程等待。 
		  notify(): 通知一个线程继续运行。 
		  setPriority(): 设置一个线程的优先级。 
		  getPriority():获得一个线程的优先级

	- 守护线程
	- 死锁

- Java 锁

	- 乐观锁 VS 悲观锁
	- 自旋锁 VS 适应性自旋锁
	- 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁
	- 公平锁 VS 非公平锁
	- 可重入锁 VS 非可重入锁
	- 独享锁 VS 共享锁

- Java 线程池

  线程过多会带来额外的开销，其中包括创建销毁线程的开销、调度线程的开销等等，同时也降低了计算机的整体性能。线程池维护多个线程，等待监督管理者分配可并发执行的任务。这种做法，一方面避免了处理任务时创建销毁线程开销的代价，另一方面避免了线程数量膨胀导致的过分调度问题，保证了对内核的充分利用。

	- 线程池是什么

		- 降低资源消耗
		- 提高响应速度
		- 提高线程的可管理性
		- 提供更多更强大的功能

	- 解决什么问题

		- 频繁申请/销毁资源和调度资源
		- 对资源无限申请缺少抑制手段
		- 系统无法合理管理内部的资源分布

	- 总体设计

		- Executor

		  将任务提交和任务执行进行解耦。用户无需关注如何创建线程，如何调度线程来执行任务，用户只需提供Runnable对象，将任务的运行逻辑提交到执行器(Executor)中，由Executor框架完成线程的调配和任务的执行部分。

		- ExecutorService

		  扩充执行任务的能力，补充可以为一个或一批异步任务生成Future的方法。
		  提供了管控线程池的方法，比如停止线程池的运行。

		- AbstractExecutorService
		- ThreadPoolExecutor

	- 运行机制
	- 生命周期管理

		- 运行状态

			- RUNNING：能够接受新提交的任务，并且也能处理阻塞队列中的任务。
			- SHUTDOWN：关闭状态，不能接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务。
			- STOP：不能接受新任务，也不能处理队列中的任务，会中断正在处理任务的线程。
			- TIDYING：所有任务都已终止，workCount(有效线程数)为0。
			- TERMINATED：在terminated()方法执行完后进入该状态。

		- 有效线程数量workCount

	- 任务执行机制

		- 任务调度：execute()

		  检查现在线程池的运行状态、运行线程数、运行策略，决定接下来执行的流程，是直接申请线程执行，或是缓冲到队列中执行，亦或是直接拒绝该任务。

			- 1. 首先检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
			- 2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
			- 3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
			- 4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
			- 5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

		- 任务缓冲：阻塞队列

		  线程池中是以生产者消费者模式，通过一个阻塞队列来实现的。阻塞队列缓存任务，工作线程从阻塞队列中获取任务。

			- ArrayBlockingQueue：数组实现的有界队列。
			- LinkedBlockingQueue：链表组成的有界队列
			- PriorityBlockingQueue：支持优先级的无界队列。
			- DelayQueue：实现PriorityBlockingQueue的延迟获取的无界队列。
			- SynchronousQueue：不存储元素的阻塞队列。

		- 任务申请：getTask()
		- 任务拒绝

		  当线程池的任务缓存队列已满，并且线程池中的线程数目达到maximumPoolSize时，就需要拒绝掉该任务，采取任务拒绝策略，保护线程池。

			- 【默认】ThreadPoolExecutor.AbortPolicy：丢弃任务并抛出RejectedExecutionException异常。
			- ThreadPoolExecutor.DiscardPolicy：丢弃异常，但是不抛出异常。
			- ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
			- ThreadPoolExecutor.CallerRunsPolicy：由调用线程(提交任务的线程)处理该任务。

	- work线程管理

		- 任务模型

			- Worker是通过继承AQS，使用AQS来实现独占锁这个功能。

		- work线程增加
		- work线程回收

		  线程池需要管理线程的生命周期，需要在线程长时间不运行的时候进行回收。

		- work线程执行任务：runWorker

	- 四种线程池

		- newFixedThreadPool

		  corePoolSize核心线程数量和maximumPoolSize最大线程数量是一致的，并且keepAliveTime为0。workQueue是LinkedBlockingQueue，这是一个链表阻塞队列。
		  
		  该线程池是一个固定数量的线程池，并且有一个无界的等待队列。

		- newCachedThreadPool

		  corePoolSize核心线程池为0，并且maximumPoolSize是int最大值，keepAliveTime为60，代表空闲60秒回收线程，workQueue是SynchronousQueue，该同步队列是一个没有容量队列，即一个任务到来后，要等待线程来消费，才能再继续添加任务。我们推导出该线程池适合处理平时没什么任务量，但有时任务量瞬间剧增的场景。

		- newScheduledThreadPool

		  该队列是一个按延迟时间从小到大排序的堆。并且当队列头节点的延迟时间小于0的时候返回该节点。所以该线程池可以指定一个时间进行定时任务。也可以通过添加任务时递增延迟时间，来进行周期任务。

		- newSingleThreadExecutor

		  该线程池的corePoolSize核心线程数量和maximumPoolSize最大线程数量都是1，代表该线程有且只有一个固定的线程，既然是单线程，所以该线程池实现的是串行操作，没有并发效果。workQueue是LinkedBlockingQueue，这是一个链表阻塞队列。所以该线程池适合执行串行执行队列中的任务。

- volatile

  Java 语言提供了一种稍弱的同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他 线程。volatile 变量具备两种特性，volatile 变量不会被缓存在寄存器或者对其他处理器不可见的 地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

	- 变量可见性

	  其一是保证该变量对所有线程可见，这里的可见性指的是当一个线程修改了变量的值，那么新的 值对于其他线程是可以立即获取的。

		- 工作内存
		- 主内存

	- 禁止重排序

	  volatile 禁止了指令重排。

		- 内存屏障

- ThreadLocal

  ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储，ThreadLocal 的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。

	- ThreadLocal原理

		- Thread维护ThreadLocal与实例的映射

	- ThreadLocal 在 JDK 8 中的实现
	- 适用场景

		- 每个线程需要有自己单独的实例
		- 例需要在多个方法中共享，但不希望被多线程共享

	- 内存泄漏

		- 强引用&弱引用
		- replaceStaleEntry

- Synchronized

	- 三种使用方式

		- 修饰普通的实例方法
		- 修饰静态方法
		- 修饰代码块

	- Synchronized的原理

		- Java对象的内存布局

			- 对象头(Header)

				- Mark Word

					- 哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向锁ID、偏向锁时间戳

				- 类型指针

					- 对象指向它的类元数据的指针

				- 记录数组长度的数据(可选)

			- 实例数据(Instance Data)
			- 对其填充(Padding)

	- synchronized锁优化

		- 无锁状态
		- 偏向锁状态
		- 轻量级锁状态
		- 重量级锁状态

	- synchronized代码块底层原理

		- monitorenter
		- monitorexit

	- Synchronized VS ReentrantLock

		- 两者的共同点

			- 都是用来协调多线程对共享对象、变量的访问
			-  都是可重入锁，同一线程可以多次获得同一个锁
			-  都保证了可见性和互斥性

		- 两者的不同点

			- ReentrantLock 显示的获得、释放锁，synchronized 隐式获得释放锁
			- ReentrantLock 可响应中断、可轮回，synchronized 是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性
			- ReentrantLock 是 API 级别的，synchronized 是 JVM 级别的
			- ReentrantLock 可以实现公平锁
			- ReentrantLock 通过 Condition 可以绑定多个条件
			- 底层实现不一样， synchronized 是同步阻塞，使用的是悲观并发策略，lock 是同步非阻塞，采用的是乐观并发策略
			- Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内置的语言实现。
			- synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生;而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁。
			- Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断。
			- 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
			- Lock 可以提高多个线程进行读操作的效率，既就是实现读写锁等。

- Lock

	- Segment
	- ReentrantLock

		- 可重入
		- 公平锁(FairSync) vs 非公平锁(NonfairSync)
		- ReentrantLock vs Synchronized

	- ReentrantReadWriteLock

		- 基本结构

			- ReadWriteLock

		- 实现原理

			- 整体原理
			- 读写状态设计

	- AQS(抽象队列同步器)

		- AQS类方法

			- 修改同步状态方法
			- 子类中可以重写的方法
			- 获取同步状态与释放同步状态方法

		- AQS具体实现及内部原理

			- AQS中FIFO队列
			- AQS同步队列具体实现结构
			- 独占式同步状态获取与释放
			- 共享式同步状态获取与释放
			- 独占式与共享式超时获取同步状态

		- AQS应用

			- ReentrantLock
			- JUC中的应用场景

				- CountDownLatch(线程计数器 )

				  CountDownLatch 类位于 java.util.concurrent 包下，利用它可以实现类似计数器的功能。比如有一个任务 A，它要等待其他 4 个任务执行完毕之后才能执行，此时就可以利用 CountDownLatch来实现这种功能了。

				- CyclicBarrier(回环栅栏-等待至 barrier 状态再全部同时执行)

				  通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环，是因为当所有等待线程都被释放以后，CyclicBarrier 可以被重用。我们暂且把这个状态就叫做 barrier，当调用 await()方法之后，线程就处于 barrier 了

				- Semaphore(信号量-控制同时访问的线程个数)

				  Semaphore 翻译成字面意思为 信号量，Semaphore 可以控制同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

			- 自定义同步工具

	- Condition

		- ConditionObject

			- 等待队列
			- 同步队列与等待队列的对应关系
			- Condition的基本使用

		- 阻塞实现 await()
		- 唤醒实现 signal()

- 并发容器

	- ConcurrentHashMap

		- Segment(ReentrantLock)

	- CopyOnWriteArrayList
	- CopyOnWriteArraySet
	- ConcurrentLinkedQueue
	- ConcurrentLinkedDeque
	- ConcurrentSkipListMap
	- ConcurrentSkipListSet
	- ArrayBlockingQueue
	- LinkedBlockingQueue
	- LinkedBlockingDeque
	- PriorityBlockingQueue
	- SynchronousQueue
	- LinkedTransferQueue
	- DelayQueue

- 原子类(CAS)

	- 基本数据类型原子类

		- AtomicBoolen
		- AtomicInteger
		- AtomicLong

	- 数组类型原子类

		- AtomicIntegerArray
		- AtomicLongArray
		- AtomicReferenceArray

	- 引用类型原子类

		- AtomicReference
		- AtomicReferenceFieldUpdater
		- AtomicMarkableReference
		- AtomicStampedReference

	- 字段类型原子类

		- AtomicIntegerFieldUpdater
		- AtomicLongFieldUpdater
		- AtomicReferenceFieldUpdater

- CAS(比较并交换)

  CAS操作内部实现原理是缓存锁，在其操作期间，会修改对应操作对象的内存地址。同时其会保证各个处理器的缓存是一致的，如果处理器发现自己的数据对应的内存地址被修改，就会将当前缓存的数据处理为无效，同时该处理器会重新从系统内存中把数据处理到缓存中。

	- 缓存一致性

	  在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理通过嗅探在总线上传播的数据来检查自己的缓存的值是不是过期了，当处理器发现自己缓存的数据对应的内存地址被修改，就会将当前处理器缓存的数据处理为无效，当处理器对这个数据进行修改的操作的时候，会重新从系统内存中把数据读到处理器缓存中。

	- 乐观锁
	- 三大问题

		- ABA 问题

			- AtomicStampedReference

		- 循环时间开销太大
		- 只能保证一个共享变量的原子操作

			- AtomicReference
			- 把多个共享变量合并成一个共享变量来操作

	- 实现

		- 原子类

- 其他组件

	- FutureTask
	- BlockingQueue
	- ForkJoin

### Java 设计模式

- 创建型

	- 工厂模式

		- 简单工厂模式
		- 工厂模式
		- 抽象工厂模式

	- 单例模式

		- 懒汉式(线程不安全)
		- 懒汉式(线程安全)
		- 双重检验锁
		- 饿汉式
		- 静态内部类
		- 枚举 Enum

	- 建造者模式
	- 原型模式

- 结构型

	- 代理模式
	- 适配器模式
	- 桥梁模式
	- 装饰模式
	- 门面模式
	- 组合模式
	- 享元模式

- 行为型

	- 策略模式
	- 观察者模式
	- 责任链模式
	- 模板方法模式
	- 状态模式

### Java 虚拟机(JVM)

- JVM运行时数据区域

  https://crowhawk.github.io/2017/08/09/jvm_1/

	- 线程私有

		- 程序计数器

			- 当前线程锁执行的字节码的行号指示器

			  字节码解释器工作时就是通过改变计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

		- Java 虚拟机栈

			- Java 方法执行的内存模型

			  每一个方法从调用直至执行完成 的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。 
			  每个方法在执行的同时都会创建一个栈帧（Stack Frame），栈帧中存储着局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，会对应一个栈帧在虚拟机栈中入栈到出栈的过程。
			  局部变量表：基本数据类型，对象引用，returnAddress类型。

		- 本地方法栈

		  与Java虚拟机栈作用很相似，它们的区别在于虚拟机栈为虚拟机执行Java方法（即字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

	- 线程共享

		- 方法区

		  Object Class Data(类定义数据)是存储在方法区的，此外，常量、静态变量、JIT编译后的代码也存储在方法区。

			- 运行时常量池（Runtime Constant Pool）

		- Java堆

		  唯一的目的是存放对象实例。

			- 新生代（Young）

				- Eden空间
				- From Survivor空间
				- To Survivor空间

			- 老年代（Tenured/Old）
			- 永久代（Perm）

			  永久代存储类信息、常量、静态变量、即时编译器编译后的代码等数据。
			  HotSpot VM 把 GC 分代收集扩展至方法区, 即使用 Java 堆的永久代来实现方法区 。

	- 直接内存（Direct Memory）

	  以NIO（New Input/Output）类为例，NIO引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

	- 元空间（JDK 1.8）

	  在 Java8 中，永久代已经被移除，被一个称为“元数据区”(元空间)的区域所取代。 元空间 的本质和永久代类似，元空间与永久代之间最大的区别在于:元空间并不在虚拟机中，而是使用 本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入 java 堆中，这样可以加载多少类的元数据就不再由 MaxPermSize 控制, 而由系统的实际可用空间来控制。

- 垃圾回收和算法

  https://crowhawk.github.io/2017/08/10/jvm_2/

	- 对象存活判定算法

		- 引用计数算法

		  给对象中添加一个引用计数器，每当有一个地方引用它时，计数器加1；当引用失效时，计数器减1；任何时刻计数器为0的对象就是不可能再被使用的。
		  引用计数算法的实现简单，判定效率也很高，大部分情况下是一个不错的算法。它没有被JVM采用的原因是它很难解决对象之间循环引用的问题。

		- 可达性分析算法

		  通过一系列的称为“GC Roots”的对象作为起点，从这些节点向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连（用图论的话来说，就是GC Roots 到这个对象不可达）时，则证明此对象时不可用的。
		  只有引用类型的变量才被认为是Roots，值类型的变量永远不被认为是Roots。

		- 两次标记与 finalize()方法

		  如果对象在进行可达性分析后发现没有与 GC Roots相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finaliza()方法。当对象没有覆盖finaliza()方法，或者finaliza()方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

		- 回收方法区

	- 垃圾收集算法

		- 标记－清除（Mark-Sweep）算法

		  首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象，标记过程在前一节讲述对象标记判定时已经讲过了。
		  
		  空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不触发另一次垃圾收集动作。
		  效率问题，因为内存碎片的存在，操作会变得更加费时，因为查找下一个可用空闲块已不再是一个简单操作。

		- 复制（Copying）算法

		  它将可用内存按容量分成大小相等的两块，每次只使用其中的一块。当这一块内存用完，就将还存活着的对象复制到另一块上面，然后再把已使用过的内存空间一次清理掉。
		  这样做使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，代价可能过高了。

			- Minor GC与复制算法

			  现在的商业虚拟机都使用复制算法来回收新生代。新生代的GC又叫“Minor GC”，IBM公司的专门研究表明：新生代中的对象98%是“朝生夕死”的，所以Minor GC非常频繁，一般回收速度也比较快，同时“朝生夕死”的特性也使得Minor GC使用复制算法时不需要按照1:1的比例来划分新生代内存空间。
			  
			  新生代将内存分为一块较大的Eden空间和两块较小的Survivor空间（From Survivor和To Survivor），每次Minor GC都使用Eden和From Survivor，当回收时，将Eden和From Survivor中还存活着的对象都一次性地复制到另外一块To Survivor空间上，最后清理掉Eden和刚使用的Survivor空间。一次Minor GC结束的时候，Eden空间和From Survivor空间都是空的，而To Survivor空间里面存储着存活的对象。在下次MinorGC的时候，两个Survivor空间交换他们的标签，现在是空的“From” Survivor标记成为“To”，“To” Survivor标记为“From”。因此，在MinorGC结束的时候，Eden空间是空的，两个Survivor空间中的一个是空的，而另一个存储着存活的对象。

		- 标记－整理（Mark-Compact）算法

		  老年代一般不能直接选用复制算法。
		  根据老年代的特点，标记－整理（Mark-Compact）算法被提出来，主要思想为：此算法的标记过程与标记－清除算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存。

		- 分代收集（Generational Collection）算法

		  根据对象存活周期的不同将内存划分为几块，一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适合的收集算法：
		  新生代 在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
		  老年代 在老年代中，因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清除”或“标记-整理”算法来进行回收。

- Java 四中引用类型

	- 强引用

	  在 Java 中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引 用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即 使该对象以后永远都不会被用到 JVM 也不会回收。因此强引用是造成 Java 内存泄漏的主要原因之 一。

	- 软引用

	  软引用需要用 SoftReference 类来实现，对于只有软引用的对象来说，当系统内存足够时它 
	  不会被回收，当系统内存空间不足时它会被回收。

	- 弱引用

	  弱引用需要用 WeakReference 类来实现，它比软引用的生存期更短，对于只有弱引用的对象 
	  来说，只要垃圾回收机制一运行，不管 JVM 的内存空间是否足够，总会回收该对象占用的内存。

	- 虚引用

	  虚引用需要 PhantomReference 类来实现，它不能单独使用，必须和引用队列联合使用。虚引用的主要作用是跟踪对象被垃圾回收的状态。

- GC垃圾收集器

  https://crowhawk.github.io/2017/08/15/jvm_3/

	- 新生代

		- Serial 垃圾收集器(单线程、复制算法)

		  它在进行垃圾收集时，必须暂停其他所有的工作线程，直至Serial收集器收集结束为止（“Stop The World”）
		  HotSpot虚拟机运行在Client模式下的默认的新生代收集器。它也有着优于其他收集器的地方：简单而高效（与其他收集器的单线程相比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得更高的单线程收集效率。

		- ParNew垃圾收集器(Serial+多线程)
		- Parallel Scavenge 收集器(多线程复制算法、高吞吐)

		  Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标是达到一个可控制的吞吐量（Throughput）。

	- 老年代

		- SerialOld收集器(单线程标记整理算法)

		  Serial Old 是 Serial 垃圾收集器年老代版本，它同样是个单线程的收集器，使用标记-整理算法， 这个收集器也主要是运行在 Client 默认的 java 虚拟机默认的年老代垃圾收集器。

		- ParallelOld收集器(多线程标记整理算法)

		  Parallel Old 收集器是 Parallel Scavenge 的年老代版本，使用多线程的标记-整理算法，在 JDK1.6 才开始提供。
		  Parallel Old 正是为了在年老代同样提供吞 吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，可以优先考虑新生代 Parallel Scavenge 和年老代 Parallel Old 收集器的搭配策略。

		- CMS收集器(多线程标记清除算法)

		  Concurrent mark sweep(CMS)收集器是一种年老代垃圾收集器，其最主要目标是获取最短垃圾回收停顿时间，和其他年老代使用标记-整理算法不同，它使用多线程的标记-清除算法。 最短的垃圾收集停顿时间可以为交互比较高的程序提高用户体验。

		- G1收集器(多线程标记整理算法)

		  Garbage first 垃圾收集器是目前垃圾收集器理论发展的最前沿成果，相比与 CMS 收集器，G1 收集器两个最突出的改进是:
		  
		  1. 基于标记-整理算法，不产生内存碎片。
		  2. 可以非常精确控制停顿时间，在不牺牲吞吐量前提下，实现低停顿垃圾回收。 
		  
		  G1 收集器避免全区域垃圾收集，它把堆内存划分为大小固定的几个独立区域，并且跟踪这些区域 的垃圾收集进度，同时在后台维护一个优先级列表，每次根据所允许的收集时间，优先回收垃圾 最多的区域。区域划分和优先级区域回收机制，确保 G1 收集器可以在有限时间获得最高的垃圾收 集效率。

- JVM类加载机制

	- 加载

	  这个阶段会在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据的入口。

	- 验证

	  确保 Class 文件的字节流中包含的信息是否符合当前虚拟机的要求。

	- 准备

	  准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，即在方法区中分配这些变量所使 用的内存空间。

	- 解析

	  解析阶段是指虚拟机将常量池中的符号引用替换为直接引用的过程。

	- 初始化

	  初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载 器以外，其它操作都由 JVM 主导。到了初始阶段，才开始真正执行类中定义的 Java 程序代码。

- 类加载器

	- 启动类加载器(Bootstrap ClassLoader)

	  负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath 参数指定路径中的，且被虚拟机认可(按文件名识别，如 rt.jar)的类。

	- 扩展类加载器(Extension ClassLoader)

	  负责加载 JAVA_HOME\lib\ext 目录中的，或通过 java.ext.dirs 系统变量指定路径中的类库。

	- 应用程序类加载器(Application ClassLoader)

	  负责加载用户路径(classpath)上的类库。 
	  JVM 通过双亲委派模型进行类的加载，当然我们也可以通过继承 java.lang.ClassLoader 
	  实现自定义的类加载器。

	- 双亲委派

	  当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父 类去完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载其中， 
	  只有当父类加载器反馈自己无法完成这个请求的时候(在它的加载路径下没有找到所需加载的 
	  Class)，子类加载器才会尝试自己去加载。 
	  保证了使用不同的类加载 器最终得到的都是同样一个 Object 对象。

	- SPI

	  按照SPI的要求，在jar包中进行适当的配置，jvm就会在运行时通过懒加载，帮我们找到所需的服务并加载。

### Java 框架

- Mybatis

	- Mybatis的解析和运行原理

		- Executor执行器

			- SimpleExecutor
			- ReuseExecutor
			- BatchExecutor

	- Hibernate 和 MyBatis 的区别

		- 半自动 vs 全自动
		- SQL优化和移植性
		- 开发难易程度和学习成本

	- 缓存

		- 一级缓存：Session
		- 二级缓存：Mapper

	- 动态SQL

- Mybatis Plus
- Spring

	- Spring特点

		- 轻量级
		- 控制反转
		- 面向切面
		- 容器
		- 框架集合

	- Spring核心组件
	- Spring常用模块
	- Spring主要包
	- Spring常用注解
	- Spring第三方结合
	- Spring IOC 原理

		- 概念
		- Spring 容器高层视图
		- IOC 容器实现

	- Spring AOP 原理

		- AOP 核心概念
		- AOP 两种代理方式

	- Spring MVC 原理

- Spring MVC
- Spring Boot

	- Spring Boot 原理

- Spring Cloud
- Netty

	- 零拷贝

## 数据库

### MySQL

- 数据库基础

	- 三大范式

- MySQL基础

	- 数据类型
	- SQL
	- 日志

		- bin log：二进制日志

			- 用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步
			- 用于数据库的基于时间点的还原

		- 【InnoDB 特有】redo log：重做日志

			- 确保事务的持久性
			- 防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。
			- crash-safe

		- undo log：回滚日志

			- 保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

- 架构

	- MySQL逻辑架构

		- Server层

			- 连接器
			- 分析器
			- 优化器
			- 执行器

		- 存储引擎层

			- InnoDB
			- MyISAM

	- 事务

		- 隔离级别：transaction-isolation

			- 读未提交
			- 读提交：Oracle默认隔离级别
			- 可重复读：MySQL默认隔离级别
			- 串行化

		- 四大特性

			- ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）

		- 并发问题

			- 脏读
			- 不可重复读
			- 幻读

		- 事务隔离的实现

			- 数据库的多版本并发控制（MVCC）

		- 事务的启动方式

			- 显式启动事务语句

				- begin 或 start transaction
				- commit
				- rollback

			- set autocommit=0

		- 死锁

			- 直接进入等待，直到超时：innodb_lock_wait_timeout
			- 死锁检测：innodb_deadlock_detect=on

	- 并发控制

		- 锁机制

			- 锁粒度

				- 全局锁

					- Flush tables with read lock (FTWRL)
					- 全库逻辑备份

						- mysqldump

				- 表级锁

					- 表锁

						- lock tables … read/write

					- 元数据锁（meta data lock，MDL)

				- 行锁

					- 各个引擎自己实现
					- 死锁和死锁检测

			- 锁模式

				- 记录锁
				- gap锁
				- next-key锁
				- 意向锁
				- 插入意向锁

			- 兼容性

				- 共享锁
				- 排他锁

			- 加锁机制

				- 乐观锁
				- 悲观锁

	- 多版本并发控制：MVCC

	  通过保存数据在某个时间点的快照。

		- 乐观并发控制
		- 悲观并发控制
		- InnoDB实现

			- 在每行记录后面保存两个隐藏的列实现的。(系统版本号)

				- 行的创建时间
				- 行的过期、删除时间

	- 存储引擎

		- InnoDB

			- 事务
			- 自动崩溃恢复
			- 性能

		- MyISAM

			- 不支持事务和行级锁
			- 全文索引，压缩，空间函数
			- 崩溃后无法安全恢复

- 索引

  索引的出现其实就是为了提高数据查询的效率，就像书的目录一样。

	- 索引的常见模型

		- 哈希表

			- 适用于只有等值查询的场景

		- 有序数组

			- 在等值查询和范围查询场景中的性能就都非常优秀。
			- 只适用于静态存储引擎

		- 搜索树

			- N 叉树

	- InnoDB 的索引模型

		- B+ 树索引模型
		- 聚簇索引(主键索引) vs 二级索引(非主键索引)

			- 回表：回到主键索引树搜索的过程

	- 索引维护

		- 页分裂
		- 页合并

	- 索引策略

		- 覆盖索引

			- 联合索引，用于减少回表操作。

		- 最左匹配索引

			- 建立联合索引原则

		- 索引下推

			- 在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数

- 查询性能(sql)优化

	- 优化数据访问

		- 扫描额外的记录

			- 使用 Explain 进行分析

		- 请求了不需要的数据

	- 重构查询方式

		- 一个复杂查询 vs 多个简单查询
		- 切分查询
		- 分解关联查询

- 复制

	- 复制的原理

		- 基于语句的复制
		- 基于行的复制

	- 主从复制

- 高可用

	- 分库分表
	- 读写分离

- 备份与恢复

	- binlog
	- relay log

### Redis

http://www.redis.cn/

- 单线程模型

	- 可维护性
	- 并发处理：IO多路复用
	- 性能瓶颈不是CPU
	- 引入多线程

		- 删除操作

- 数据结构

	- String
	- List
	- Set
	- Hash
	- Zset

- 回收策略

	- noeviction

	  当内存不足以容纳新写入数据时，新写入操作会报错。

	- allkeys-lru

	  当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。

	- allkeys-random

	  当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。

	- volatile-lru

	  当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。

	- volatile-random

	  当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

	- volatile-ttl

	  当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

- Redis 持久化

  http://www.redis.cn/topics/persistence.html

	- RDB

	  RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能。

		- 备份

	- AOF

		- AOF重写

	- RDB 和 AOF 的混合持久化

	  在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。

- 事务（transactions）

	- MULTI：标记一个事务块的开始。
	- EXEC：执行所有事务块内的命令。
	- DISCARD：取消事务，放弃执行事务块内的所有命令。
	- WATCH：监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
	- UNWATCH：取消 WATCH 命令对所有 key 的监视

- 复制（replication）

	- RDB

- 高可用（high availability）

	- Redis哨兵（Sentinel）

	  http://www.redis.cn/topics/sentinel.html

		- 监控（Monitoring）
		- 提醒（Notification）
		- 自动故障迁移（Automatic failover）

	- Redis 集群

	  http://www.redis.cn/topics/cluster-tutorial.html

		- 数据分片（哈希槽）
		- 主从复制

			- 不能保证强一致性

- 解决方案

	- 分布式锁

	  http://www.redis.cn/topics/distlock.html

		- 单个实例：SET resource_name my_random_value NX PX 30000
		- 集群：Redlock

- 客户端

	- Jedis
	- Redisson
	- Lettuce

- 常见问题

	- 缓存穿透

		- 设置null值
		- 布隆过滤器

	- 缓存击穿

		- 分布式锁

	- 缓存雪崩

		- 高可用
		- Hystrix限流、降级

	- 热点数据集中失效

		- 设置不同失效时间
		- 分布式锁
		- 永不失效

## 微服务

### 服务注册与发现

- Spring Cloud Zookeeper：CP
- Spring Cloud Consul：CP

	- Key/Value 存储
	- 健康检查

- Eureka（Spring Cloud Netflix）：AP
- Nacos

	- 动态配置服务

### 负载均衡的服务调用

- Ribbon（Spring Cloud Netflix）

	- 轮询策略
	- 随机选择
	- 最大可用策略
	- 带有加权的轮询策略(响应时间)
	- 可用过滤策略
	- 区域感知策略
	- 自定义负载均衡算法：AbstractLoadBalancerRule

### 服务容错保护

- Hystrix（Spring Cloud Netflix）

	- 线程隔离

		- 线程池
		- 信号量

	- 熔断器：Circuit Breaker
	- 降级机制

		- 熔断器降级
		- 限流降级
		- 超时降级
		- 异常降级
		- 平均响应时间降级

	- 熔断机制

		- 基本模式
		- 扩展模式

	- 熔断策略

		- 平均响应时间
		- 异常比例
		- 异常数

	- 降级回退方式

		- Fail Fast 快速失败
		- Fail Silent 无声失败
		- Fallback: Static 返回默认值
		- Fallback: Stubbed 自己组装一个值返回
		- Fallback: Cache via Network 利用远程缓存
		- Primary + Secondary with Fallback 主次方式回退（主要和次要）

### 基于Ribbon和Hystrix的声明式服务调用

- Spring Cloud OpenFeign

### API网关服务

- Zuul（Spring Cloud Netflix）
- Spring Cloud Gateway

### 外部集中化配置管理

- Spring Cloud Config

### 消息总线

- Spring Cloud Bus

### 分布式请求链路跟踪

- Spring Cloud Sleuth

### 微服务应用监控

- Spring Boot Admin

### 分布式系统的流量防卫兵

- Sentinel

## 系统架构

### 分布式

- 一致性协议和算法

	- CAP定理

	  不能同时保证一致性、可用性和分区容错性，每一个系统只能在这三种特性中选择两种。

		- 一致性

			- 在分布式环境下，一致性是指数据在多个副本之间能否保持一致的特性。

		- 可用性

			- 系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

		- 分区容错性

			- 分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

	- BASE理论

		- Basically Available（基本可用）
		- Soft state（软状态）
		- Eventually consistent（最终一致性）

	- 拜占庭将军问题

		- 有一组将军分别指挥一部分军队，每一个将军都不知道其它将军是否是可靠的，也不知道其他将军传递的信息是否可靠，但是它们需要通过投票选择是否要进攻或者撤退。

	- 2PC

		- 协调者
		- 参与者

	- 3PC

		- 超时机制

	- 共识算法

		- Paxos算法

			- 三个角色

				- Proposer提案者
				- Acceptor表决者
				- Learner学习者

			- 两个阶段

				- prepare 阶段
				- accept 阶段

			- 死循环问题：允许只有一个提案者

		- Raft算法

			- 只有最新的服务器才能成为领导者

- Zookeeper：分布式协调服务框架

	- ZAB协议：ZooKeeper Automic Broadcast

		- 消息广播（事务操作）

			- Zab 协议中 Leader 等待半数以上的Follower成功反馈即可，不需要收到全部Follower反馈

		- 崩溃恢复（选主+数据同步）

			- 数据同步
			- 选举：广播优先级消息

	- ZooKeeper基础

		- 数据模型

			- 持久节点
			- 持久顺序节点
			- 临时节点
			- 临时顺序节点

		- Watcher机制
		- A(可用性)、P(分区容错性)

	- Zookeeper 工作流程

		- 如果客户端想要读取特定的znode
		- 如果客户端想要将数据存储在ZooKeeper集合中

	- 集群

		- 半数以上存活
		- Leader
		- Follower

	- 应用场景

		- 领导人选举
		- 分布式锁
		- 命名服务
		- 集群管理和注册中心

- 分布式锁

	- 数据库（MySQL）

		- 唯一索引或者主键
		- 排他锁

	- Redis

		- 单个实例：SET resource_name my_random_value NX PX 30000
		- 集群：Redlock(超时时间)

	- Zookeeper

		- 临时顺序节点
		- watcher监听器

- 分布式事务

	- 分布式事务协议

		- 两阶段提交协议 2PC
		- 三阶段提交协议 3PC

	- 解决方案

		- 2PC/XA方案

			- 数据库

		- TCC强一致性方案
		- 可靠消息最终一致性方案

			- 支持事务的消息队列：RabbitMQ

			  txSelect(), txCommit()以及txRollback()。

		- 最大努力通知方案

			- 不支持事务的消息队列

	- 框架

		- Seata

		  Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。
		  https://seata.io/zh-cn/index.html

			- Seata AT 模式

				- 两阶段：支持本地ACID 事务的关系型数据库

			- Seata TCC 模式

				- 两阶段：不依赖于底层数据资源的事务支持

			- Seata Saga 模式

			  Saga模式是SEATA提供的长事务解决方案，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。

				- 长事务解决方案

			- Seata XA 模式

				- 两阶段协议，MySQL

		- LCN

			- LCN事务模式

				- AT模式

			- TCC事务模式
			- TXC事务模式

- 分布式搜索引擎

	- ElasticSearch

		- 核心概念

			- 索引 index
			- 类型 type
			- 映射 mapping
			- 文档 document
			- 字段 field
			- 集群 cluster
			- 节点 node
			- 分片和副本 node
			- 核心数据类型

				- 字符串
				- 数值型
				- 数值型 boolean
				- 二进制
				- 范围类型
				- 日期

		- 基本操作

			- 创建索引 PUT请求
			- 查看索引 GET请求
			- 删除索引 DELETE请求
			- 批量获取索引 GET请求
			- 获取全部索引 GET请求
			- 使用_cat获取全部索引 GET请求
			- 判断索引是否存在 HEAD请求
			- 关闭索引 不删除 POST请求
			- 打开索引 POST请求

		- 映射的介绍与使用

			- 创建Mapping PUT请求
			- 查看Mapping信息 GET请求
			- 批量获取Mapping信息 GET请求
			- 获取所有Mapping信息第一种方式 GET请求
			- 获取所有Mapping信息第二种方式 GET请求
			- 增加Mapping字段 POST请求

		- 文档的增删改查

			- 新增文档 指定ID PUT/POST请求
			- 新增文档 自动生成ID POST请求
			- 自动创建索引 POST请求
			- 查看自动创建的索引 GET请求
			- 指定操作类型
			- 查看指定ID文档 GET请求
			- 查看多条文档 第一种方式 GET/POST请求
			- 查看多条文档 第二种方式 GET/POST请求
			- 查看多条文档 第三种方式 GET/POST请求
			- 查看多条文档 第四种方式 GET/POST请求
			- 修改文档 POST请求
			- 向_source增加字段 POST请求
			- 向_source删除字段 POST请求
			- 更新指定文档的字段 POST请求
			- upsert介绍 
			- 删除文档 DELETE请求

		- 文档的搜索

			- 单条trem查询 GET/POST请求
			- 多条trem查询 GET/POST请求
			- match_all查询 GET/POST请求
			- match查询 GET/POST请求
			- multi_match 多个查询 GET/POST请求
			- match_phrase 多个查询 GET/POST请求
			- match_phrase_profix 多个查询 GET/POST请求

		- ElasticSerach 搜索

			- 批量导入数据
			- trem多种查询
			- 范围查询 
			- 布尔查询
			- 排序查询
			- 聚合查询指标聚合
			- 聚合查询桶聚合
			- query_string 查询

		- ElasticSerach 高级搜索

			- 索引别名的使用
			- 如何重建索引
			- refresh操作
			- 高亮查询
			- 查询建议

		- 技术架构
		- 原理
		- 集群架构

### 中间件

- 消息队列

	- 使用场景

		- 解耦
		- 异步
		- 削峰

	- 高级特性

		- 可靠传递性

			- 重复消息

				- 数据库id
				- 版本号
				- 状态机

			- 消息丢失

				- RabbitMQ

					- 生产者弄丢了数据

						- 事务机制：同步
						- confirm机制：异步

					- RabbitMQ弄丢了数据

						- 持久化

					- 消费端弄丢了数据

						- ack机制

				- Kafka

					- 消费端弄丢了数据

						- offset

					- Kafka弄丢了数据

						- 每个partition必须有至少2个副本

					- 生产者会不会弄丢数据

						- ack=all

						  每条数据，必须是写入所有replica之后，才能认为是写成功了。

			- 消息顺序性

				- RabbitMQ

					- Queue

				- Kafka

					- 内存队列

			- 高可用

		- 事务

			- RabbitMQ

		- 协议

			- AMQP 协议
			- MQTT 协议

	- RabbitMQ

		- RabbitMQ入门

			- RabbitMQ 特点

				- 可靠性
				- 灵活的路由
				- 扩展性
				- 高可用性
				- 多种协议
				- 多语言客户端
				- 管理界面
				- 插件机制

			- RabbitMQ的安装
			- RabbitMQ 基本概念

				- Exchange
				- Binding
				- Queue

			- Exchange 类型

				- direct
				- fanout
				- topic

					- *
					- #

				- headers

			- AMQP协议

				- AMOP协议的erlang实现

			- 消费模式

				- 推(Push)模式：Basic.Consume
				- 拉(Pull)模式：Basic.Get

		- RabbitMQ进阶

			- 死信队列

				- 消息被拒绝
				- 消息过期
				- 队列达到最大长度

			- 延迟队列

				- 死信队列 TTL

			- 优先级队列

				- x-max-priority

			- RPC实现
			- 持久化

				- durable = true

			- 生产者确认

				- 事务机制：同步

					- channel.txSelect
					- channel.txCommit
					- channel.txRollBack

				- confirm机制：chanel.confirmSelect

					- 回调通知：异步

			- 消息顺序性

				- 全局有序标识

			- 消息可靠传输

				- 最多一次
				- 最少一次
				- 刚好一次

		- RabbitMQ集群

			- 元数据(队列，交换器，绑定，vhost)同步
			- 消息数据不同步

		- RabbitMQ高级

			- 镜像队列(Mirror Queue)

	- Kafka

		- 与传统消息系统区别

			- 持久化日志
			- 天生分布式系统
			- 实时流式处理
			- 高吞吐率

				- partition，顺序写磁盘

		- 应用场景

			- 消息系统
			- 应用监控
			- 网站行为追踪
			- 流处理
			- 持久性日志

		- Kafka架构

			- 基本概念

				- Topic：消息类别
				- Partition：消息水平分区
				- Broker：消息服务器
				- Replica：消息副本

			- Push vs. Pull
			- Replication & Leader election

				- Replication：default.replication.factor
				- 每个partition都有一个唯一的leader
				- 怎样在follower中选举出新的leader

					- 和 Leader 保持同步的 Follower 集合：ISR：in-sync replicas

						- 如果 Follower 长时间未向 Leader 同步数据，则该 Follower 将被踢出 ISR 集合，该时间阈值由 replica.lag.time.max.ms 参数设定。Leader 发生故障后，就会从 ISR 中选举出新的 Leader。

					- 某一个partition的所有replica都挂了

						- 等待ISR中的任一个replica“活”过来，并且选它作为leader
						- 选择第一个“活”过来的replica（不一定是ISR中的）作为leader

			- Consumer group

				- Kafka保证的是稳定状态下每一个consumer实例只会消费某一个或多个特定partition的数据，而某个partition的数据只会被某一个特定的consumer实例所消费。

			- Consumer Rebalance
			- 消息可靠性传输

				- 消息丢失

					- 副本机制

				- 重复消息

					- 幂等性

				- 消息顺序性

					- Partition 级别的顺序性

	- 消息中间件对比

		- RabbitMQ：支撑高并发、高吞吐、性能很高、集群化、高可用部署架构、消息高可靠支持。
		- Kafka：专为超高吞吐量的实时日志采集、实时数据同步、实时数据计算等场景来设计。

- RPC

	- RPC原理

		- 动态代理
		- 消息编码解码
		- 序列化
		- 通信
		- 服务注册、发现

	- Dubbo
	- Thrift
	- gRPC

- 日志系统

	- ELK

- 数据库

	- Mycat

### 系统设计

- 秒杀系统

	- 前端页面

		- CDN
		- 限流

	- 限流

		- Redis
		- RateLimit

	- 查询库存

		- Redis

	- 消息队列

		- RabbitMQ

	- 数据库
	- 库存问题(超卖)

		- lock
		- 数据库乐观锁
		- 数据库悲观锁
		- 阻塞队列
		- 数据库乐观锁(可重入)

- 高并发系统

	- 流量控制

		- 限流算法

			- 固定(临界突发流量)、滑动时间窗口限流算法、多层次限流
			- 漏桶算法
			- 令牌桶算法

		- 框架

			- Sentinel
			- RateLimiter

	- 系统设计

		- 系统拆分
		- 缓存
		- MQ
		- 分库分表
		- 读写分离
		- ElasticSearch

- 抢红包系统

	- 技术难点

		- 更海量的并发请求
		- 更严格的安全级别

	- 高并发解决方案

		- 垂直切分
		- 冷热分离(天)

## 运维&测试

### 容器编排

- 容器化

	- 敏捷地创建和部署应用程序
	- 持续构建集成
	- 分离开发和运维的关注点
	- 可监控性
	- 开发、测试、生产不同阶段的环境一致性
	- 跨云服务商、跨操作系统发行版的可移植性
	- 以应用程序为中心的管理
	- 松耦合、分布式、弹性、无约束的微服务
	- 资源隔离
	- 资源利用

- Docker

	- Docker 安装
	- 镜像(image)
	- 容器(container)
	- 仓库(Registry)
	- Dockerfile
	- 数据持久化

		- 数据卷
		- 数据卷容器

	- Docker Compose

- Kubernetes(k8s)

	- K8s的功能

		- 服务发现和负载均衡
		- 存储编排
		- 自动发布和回滚
		- 自愈
		- 密钥及配置管理

	- k8s安装

		- sealyun

		  https://sealyun.com/

	- 核心组件

		- Master组件

			- kube-apiserver

				- 操作资源的入口并且提供认证、授权、权限控制、API注册和服务发现的机制。

			- etcd：配置

				- CoreOS开源的、分布式、键值对数据库，它是集群所有组件单一数据源（SSOT）的保障。

			- kube-scheduler

				- 负责资源的调度以及根据预先设定的调度策略将pod调度到合适的节点上。

			- kube-controller-manager

			  负责管理集群的状态，如异常发现、自动扩容和滚动更新等。

				- Node Controller：节点控制器
				- Replication Controller：副本控制器
				- Endpoints Controller：端点控制器
				- Service Account & Token Controllers

			- cloud-controller-manager

		- Node 组件

			- kubelet

				- Kubelet负责管理容器的生命周期、数据卷以及网络（CNI）

			- kube-proxy

				- Kube-proxy负责服务发现和集群Service的负载均衡

			- 容器引擎

		- Addons

			- DNS
			- Web UI（Dashboard）
			- Kuboard

	- 基本概念

		- Container
		- Pod
		- Node
		- Namespace
		- Service
		- Label
		- Annotations
		- Deployment：部署
		- Replication Controller：复制控制器
		- Replica Set：副本集
		- Job：任务
		- DaemonSet：后台支撑服务集
		- StatefulSet：有状态服务集

### 持续集成

- Jenkins

### 测试

### 虚拟化技术

### 云技术

### DevOps

### 服务器

- Nginx

  https://www.w3cschool.cn/nginx/

	- 特性

	  选择Nginx的核心理由还是它能在支持高并发请求的同时保持高效的服务。

		- 更快
		- 高扩展性
		- 高可靠性
		- 低内存消耗
		- 单机支持10万以上的并发连接
		- 热部署
		- 最自由的BSD许可协议

	- Nginx 架构

		- 一个 master 进程

		  master 进程主要用来管理 worker 进程，包含：接收来自外界的信号，向各 worker 进程发送信号，监控 worker 进程的运行状态，当 worker 进程退出后(异常情况下)，会自动重新启动新的 worker 进程。

		- 多个 worker 进程

		  多个 worker 进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。

	- Nginx模块

		- handler 模块
		- 过滤模块
		- upstream 模块
		- 其他模块

			- core 模块
			- event 模块

	- Nginx配置
	- 负载均衡策略

		- 轮询（默认）
		- weight（权重）
		- ip_hash（hash值分配）
		- fair（第三方）
		- url_hash（第三方）

### 负载均衡

- SLB

## 人工智能&大数据(了解)

### HDFS

### Hive

### Hadoop

### Spark

### Hbase

### Fink

