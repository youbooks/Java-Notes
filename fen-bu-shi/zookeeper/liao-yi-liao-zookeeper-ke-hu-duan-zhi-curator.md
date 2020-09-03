# 聊一聊 Zookeeper 客户端之 Curator

> 原文链接：[聊一聊 Zookeeper 客户端之 Curator](https://juejin.im/post/6844903818090512397)

ZooKeeper 是一个分布式的、开放源码的分布式应用程序协调服务，是 Google 的 Chubby 一个开源的实现。它是集群的管理者，监视着集群中各个节点的状态，根据节点提交的反馈进行下一步合理操作。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

Curator 是 Netflix 公司开源的一套 Zookeeper 客户端框架，解决了很多 Zookeeper 客户端非常底层的细节开发工作，包括连接重连、反复注册 Watcher 和 NodeExistsException 异常等等。Curator 包含了几个包：

* curator-framework：对 Zookeeper 的底层 api 的一些封装
* curator-client：提供一些客户端的操作，例如重试策略等
* curator-recipes：封装了一些高级特性，如：Cache 事件监听、选举、分布式锁、分布式计数器、分布式Barrier 等

## 1. Curator 和 zookeeper 的版本问题

目前 Curator 有 2.x.x 和 3.x.x 两个系列的版本，支持不同版本的 Zookeeper。其中Curator 2.x.x 兼容 Zookeeper的 3.4.x 和 3.5.x。而 Curator 3.x.x 只兼容 Zookeeper 3.5.x，并且提供了一些诸如动态重新配置、watch删除等新特性。

```text
Curator 2.x.x - compatible with both ZooKeeper 3.4.x and ZooKeeper 3.5.x
Curator 3.x.x - compatible only with ZooKeeper 3.5.x and includes support for new
```

如果跨版本会有兼容性问题，很有可能导致节点操作失败，当时在使用的时候就踩了这个坑，抛了如下的异常：

```text
KeeperErrorCode = Unimplemented for /***
```

## 2. Curator API

这里就不对比与原生 API 的区别了，Curator 的 API 直接通过 org.apache.curator.framework.CuratorFramework 接口来看，并结合相应的案例进行使用，以备后用。

> 为了可以直观的看到 Zookeeper 的节点信息，可以考虑弄一个 zk 的管控界面，常见的有 zkui 和 zkweb。
>
> zkui：[github.com/DeemOpen/zk…](https://github.com/DeemOpen/zkui)
>
> zkweb：[github.com/zhitom/zkwe…](https://github.com/zhitom/zkweb)
>
> 我用的 zkweb ，虽然界面上看起来没有 zkui 精简，但是在层次展示和一些细节上感觉比 zkui 好一点

## 3. 环境准备

之前写的一个在 [Linux 上安装部署 Zookeeper](http://www.glmapper.com/2019/03/04/zk-on-linux) 的笔记，其他操作系统请自行谷歌教程吧。

本文案例工程已经同步到了 github，[传送门](https://github.com/glmapper/glmapper-blog-samples)。

> PS : 目前还没有看过Curator的具体源码，所以不会涉及到任何源码解析、实现原理的东西；本篇主要是实际使用时的一些记录，以备后用。如果文中错误之处，希望各位指出。

## 4. Curator 客户端的初始化和初始化时机

在实际的工程中，Zookeeper 客户端的初始化会在程序启动期间完成。

### 4.1 初始化时机

在 Spring 或者 SpringBoot 工程中最常见的就是绑定到容器启动的生命周期或者应用启动的生命周期中：

* 监听 ContextRefreshedEvent 事件，在容器刷新完成之后初始化 Zookeeper
* 监听 ApplicationReadyEvent/ApplicationStartedEvent 事件，初始化 Zookeeper 客户端

除了上面的方式之外，还有一种常见的是绑定到 bean 的生命周期中

* 实现 InitializingBean 接口 ，在 afterPropertiesSet 中完成 Zookeeper 客户端初始化

> 关于 SpringBoot中的事件机制可以参考之前写过的一篇文章：[SpringBoot-SpringBoot中的事件机制](https://juejin.im/post/6844903750335725576#heading-8)。

### 4.2 Curator 初始化

这里使用 InitializingBean 的这种方式，代码如下：

```java
public class ZookeeperCuratorClient implements InitializingBean {
    private CuratorFramework curatorClient;
    @Value("${glmapper.zookeeper.address:localhost:2181}")
    private String           connectString;
    @Value("${glmapper.zookeeper.baseSleepTimeMs:1000}")
    private int              baseSleepTimeMs;
    @Value("${glmapper.zookeeper.maxRetries:3}")
    private int              maxRetries;
    @Value("${glmapper.zookeeper.sessionTimeoutMs:6000}")
    private int              sessionTimeoutMs;
    @Value("${glmapper.zookeeper.connectionTimeoutMs:6000}")
    private int              connectionTimeoutMs;

    @Override
    public void afterPropertiesSet() throws Exception {
        // custom policy
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(baseSleepTimeMs, maxRetries);
        // to build curatorClient
        curatorClient = CuratorFrameworkFactory.builder().connectString(connectString)
                .sessionTimeoutMs(sessionTimeoutMs).connectionTimeoutMs(connectionTimeoutMs)
                .retryPolicy(retryPolicy).build();
        curatorClient.start();
    }

    public CuratorFramework getCuratorClient() {
        return curatorClient;
    }
}
```

glmapper.zookeeper.xxx 是本例中需要在配置文件中配置的 zookeeper 的一些参数，参数解释如下：

* baseSleepTimeMs：重试之间等待的初始时间
* maxRetries：最大重试次数
* connectString：要连接的服务器列表
* sessionTimeoutMs：session 超时时间
* connectionTimeoutMs：连接超时时间

另外，Curator 客户端初始化时还需要指定重试策略，RetryPolicy 接口是 Curator 中重试连接\(当zookeeper失去连接时使用\)策略的顶级接口，其类继承体系如下图所示：

![2020-07-31-LnFaGx](https://image.ldbmcs.com/2020-07-31-LnFaGx.jpg)

* RetryOneTime：只重连一次
* RetryNTime：指定重连的次数N
* RetryUtilElapsed：指定最大重连超时时间和重连时间间隔，间歇性重连直到超时或者链接成功
* ExponentialBackoffRetry：基于 "backoff"方式重连，和 RetryUtilElapsed 的区别是重连的时间间隔是动态的。
* BoundedExponentialBackoffRetry： 同 ExponentialBackoffRetry的区别是增加了最大重试次数的控制

除上述之外，在一些场景中，需要对不同的业务进行隔离，这种情况下，可以通过设置 namespace 来解决，namespace 实际上就是指定zookeeper的根路径，设置之后，后面的所有操作都会基于该根目录。

## 5. Curator 基础 API 使用

### 5.1 检查节点是否存在

checkExists 方法返回的是一个 ExistsBuilder 构造器，这个构建器将返回一个 Stat 对象，就像调用了 org.apache.zookeeper.ZooKeeper.exists\(\)一样。null 表示它不存在，而实际的 Stat 对象表示存在。

```java
public void checkNodeExist(String path) throws Exception {
    Stat stat = curatorClient.checkExists().forPath(path);
    if (stat != null){
        throw new RuntimeException("path = "+path +" has bean exist.");
    }
}
```

建议在实际的应用中，操作节点时对所需操作的节点进行 checkExists。

### 5.2 新增节点

* 非递归方式创建节点

  ```java
  curatorClient.create().forPath("/glmapper");
  curatorClient.create().forPath("/glmapper/test");
  ```

  先创建/glmapper，然后再在/glmapper 下面创建 /test ，如果直接使用 /glmapper/test 没有先创建 /glmapper 时，会抛出异常：

  ```java
  org.apache.zookeeper.KeeperException$NoNodeException: KeeperErrorCode = NoNode for /glmapper/test
  ```

  如果需要在创建节点时指定节点中数据，则可以这样：

  ```java
  curatorClient.create().forPath("/glmapper","data".getBytes());
  ```

  指定节点类型\(EPHEMERAL 临时节点\)

  ```java
  curatorClient.create().withMode(CreateMode.EPHEMERAL).forPath("/glmapper","data".getBytes());
  ```

* 递归方式创建节点 递归方式创建节点有两个方法，creatingParentsIfNeeded 和 creatingParentContainersIfNeeded。在新版本的 zookeeper 这两个递归创建方法会有区别； creatingParentContainersIfNeeded\(\) 以容器模式递归创建节点，如果旧版本 zookeeper，此方法等于creatingParentsIfNeeded\(\)。 在非递归方式情况下，如果直接创建 /glmapper/test 会报错，那么在递归的方式下则是可以的：

  ```java
  curatorClient.create().creatingParentContainersIfNeeded().forPath("/glmapper/test");
  ```

  在递归调用中，如果不指定 CreateMode，则默认`PERSISTENT`，如果指定为临时节点，则最终节点会是临时节点，父节点仍旧是`PERSISTENT`

### 5.3 删除节点

* 非递归删除节点

  ```java
  curatorClient.delete().forPath("/glmapper/test");
  ```

  指定具体版本

  ```java
  curatorClient.delete().withVersion(-1).forPath("/glmapper/test");
  ```

  使用 guaranteed 方式删除，guaranteed 会保证在session有效的情况下，后台持续进行该节点的删除操作，直到删除掉：

  ```java
  curatorClient.delete().guaranteed().withVersion(-1).forPath("/glmapper/test");
  ```

* 递归删除当前节点及其子节点

  ```java
  curatorClient.delete().deletingChildrenIfNeeded().forPath("/glmapper/test");
  ```

### 5.4 获取节点数据

获取节点数据

```java
byte[] data = curatorClient.getData().forPath("/glmapper/test");
```

根据配置的压缩提供程序对数据进行解压缩处理：

```java
byte[] data = curatorClient.getData().decompressed().forPath("/glmapper/test");
```

读取数据并获得Stat信息：

```java
Stat stat = new Stat();
byte[] data = curatorClient.getData().storingStatIn(stat).forPath("/glmapper/test");
```

### 5.5 更新节点数据

设置指定值：

```java
curatorClient.setData().forPath("/glmapper/test","newData".getBytes());
```

设置数据并使用配置的压缩提供程序压缩数据：

```java
curatorClient.setData().compressed().forPath("/glmapper/test","newData".getBytes());
```

设置数据，并指定版本：

```java
curatorClient.setData().withVersion(-1).forPath("/glmapper/test","newData".getBytes());
```

### 5.6 获取子列表

```java
List<String> childrenList = curatorClient.getChildren().forPath("/glmapper");
```

## 6. 事件

Curator 也对 Zookeeper 典型场景之事件监听进行封装，这部分能力实在 curator-recipes 包下的。

### 6.1 事件类型

在使用不同的方法时会有不同的事件发生：

```java
public enum CuratorEventType
{
    //Corresponds to {@link CuratorFramework#create()}
    CREATE,
    //Corresponds to {@link CuratorFramework#delete()}
    DELETE,
        //Corresponds to {@link CuratorFramework#checkExists()}
    EXISTS,
        //Corresponds to {@link CuratorFramework#getData()}
    GET_DATA,
        //Corresponds to {@link CuratorFramework#setData()}
    SET_DATA,
        //Corresponds to {@link CuratorFramework#getChildren()}
    CHILDREN,
        //Corresponds to {@link CuratorFramework#sync(String, Object)}
    SYNC,
        //Corresponds to {@link CuratorFramework#getACL()}
    GET_ACL,
        //Corresponds to {@link CuratorFramework#setACL()}
    SET_ACL,
        //Corresponds to {@link Watchable#usingWatcher(Watcher)} or {@link Watchable#watched()}
    WATCHED,
        //Event sent when client is being closed
    CLOSING
}
```

### 6.2 事件监听

#### 6.2.1 一次性监听方式：Watcher

利用 Watcher 来对节点进行监听操作，可以典型业务场景需要使用可考虑，但一般情况不推荐使用。

```java
 byte[] data = curatorClient.getData().usingWatcher(new Watcher() {
     @Override
     public void process(WatchedEvent watchedEvent) {
       System.out.println("监听器 watchedEvent：" + watchedEvent);
     }
 }).forPath("/glmapper/test");
System.out.println("监听节点内容：" + new String(data));
// 第一次变更节点数据
curatorClient.setData().forPath("/glmapper/test","newData".getBytes());
// 第二次变更节点数据
curatorClient.setData().forPath("/glmapper/test","newChangedData".getBytes());
```

上面这段代码对 /glmapper/test 节点注册了一个 Watcher 监听事件，并且返回当前节点的内容。后面进行两次数据变更，实际上第二次变更时，监听已经失效，无法再次获得节点变动事件了。测试中控制台输出的信息如下：

```java
监听节点内容：data
watchedEvent：WatchedEvent state:SyncConnected type:NodeDataChanged path:/glmapper/test
```

#### 6.2.2 CuratorListener 方式

CuratorListener 监听，此监听主要针对 background 通知和错误通知。使用此监听器之后，调用inBackground 方法会异步获得监听，对于节点的创建或修改则不会触发监听事件。

```java
CuratorListener listener = new CuratorListener(){
    @Override
    public void eventReceived(CuratorFramework client, CuratorEvent event) throws Exception {
      System.out.println("event : " + event);
    }
 };
// 绑定监听器
curatorClient.getCuratorListenable().addListener(listener);
// 异步获取节点数据
curatorClient.getData().inBackground().forPath("/glmapper/test");
// 更新节点数据
curatorClient.setData().forPath("/glmapper/test","newData".getBytes());
```

测试中控制台输出的信息如下：

```text
event : CuratorEventImpl{type=GET_DATA, resultCode=0, path='/glmapper/test', name='null', children=null, context=null, stat=5867,5867,1555140974671,1555140974671,0,0,0,0,4,0,5867
, data=[100, 97, 116, 97], watchedEvent=null, aclList=null}
```

这里只触发了一次监听回调，就是 getData 。

#### 6.2.3 Curator 引入的 Cache 事件监听机制

Curator 引入了 Cache 来实现对 Zookeeper 服务端事件监听，Cache 事件监听可以理解为一个本地缓存视图与远程 Zookeeper 视图的对比过程。Cache 提供了反复注册的功能。Cache 分为两类注册类型：节点监听和子节点监听。

* NodeCache

  监听数据节点本身的变化。对节点的监听需要配合回调函数来进行处理接收到监听事件之后的业务处理。NodeCache 通过 NodeCacheListener 来完成后续处理。

  ```java
  String path = "/glmapper/test";
  final NodeCache nodeCache = new NodeCache(curatorClient,path);
  //如果设置为true则在首次启动时就会缓存节点内容到Cache中。 nodeCache.start(true);
  nodeCache.start();
  nodeCache.getListenable().addListener(new NodeCacheListener() {
  @Override
  public void nodeChanged() throws Exception {
  System.out.println("触发监听回调，当前节点数据为：" + new String(nodeCache.getCurrentData().getData()));
  }
  });
  curatorClient.setData().forPath(path,"1".getBytes());
  curatorClient.setData().forPath(path,"2".getBytes());
  curatorClient.setData().forPath(path,"3".getBytes());
  curatorClient.setData().forPath(path,"4".getBytes());
  curatorClient.setData().forPath(path,"5".getBytes());
  curatorClient.setData().forPath(path,"6".getBytes());
  ```

  注意：在测试过程中，nodeCache.start\(\)，NodeCache 在先后多次修改监听节点的内容时，出现了丢失事件现象，在用例执行的5次中，仅一次监听到了全部事件；如果 nodeCache.start\(true\)，NodeCache 在先后多次修改监听节点的内容时，不会出现丢失现象。

  > NodeCache不仅可以监听节点内容变化，还可以监听指定节点是否存在。如果原本节点不存在，那么Cache就会在节点被创建时触发监听事件，如果该节点被删除，就无法再触发监听事件。

* PathChildrenCache

  PathChildrenCache 不会对二级子节点进行监听，只会对子节点进行监听。

  ```java
  String path = "/glmapper";
  PathChildrenCache pathChildrenCache = new PathChildrenCache(curatorClient,path,true);
  // 如果设置为true则在首次启动时就会缓存节点内容到Cache中。 nodeCache.start(true);
  pathChildrenCache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
  pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
    @Override
    public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent event) throws Exception {
      System.out.println("-----------------------------");
      System.out.println("event:"  + event.getType());
      if (event.getData()!=null){
        System.out.println("path:" + event.getData().getPath());
      }
      System.out.println("-----------------------------");
    }
  });
  zookeeperCuratorClient.createNode("/glmapper/test","data".getBytes(),CreateMode.PERSISTENT);
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test","1".getBytes());
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test","2".getBytes());
  Thread.sleep(1000);
  zookeeperCuratorClient.createNode("/glmapper/test/second","data".getBytes(),CreateMode.PERSISTENT);
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test/second","1".getBytes());
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test/second","2".getBytes());
  Thread.sleep(1000);
  ```

  注意：在测试过程中发现，如果连续两个操作之间不进行一定时间的间隔，会导致无法监听到下一次事件。因此只会监听子节点，所以对二级子节点 /second 下面的操作是监听不到的。测试中控制台输出的信息如下：

  ```text
  -----------------------------
  event:CHILD_ADDED
  path:/glmapper/test
  -----------------------------
  -----------------------------
  event:INITIALIZED
  -----------------------------
  -----------------------------
  event:CHILD_UPDATED
  path:/glmapper/test
  -----------------------------
  -----------------------------
  event:CHILD_UPDATED
  path:/glmapper/test
  -----------------------------
  ```

* TreeCache

  TreeCache 使用一个内部类TreeNode来维护这个一个树结构。并将这个树结构与ZK节点进行了映射。所以TreeCache 可以监听当前节点下所有节点的事件。

  ```java
  String path = "/glmapper";
  TreeCache treeCache = new TreeCache(curatorClient,path);
  treeCache.getListenable().addListener((client,event)-> {
      System.out.println("-----------------------------");
      System.out.println("event:"  + event.getType());
      if (event.getData()!=null){
        System.out.println("path:" + event.getData().getPath());
      }
      System.out.println("-----------------------------");
  });
  treeCache.start();
  zookeeperCuratorClient.createNode("/glmapper/test","data".getBytes(),CreateMode.PERSISTENT);
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test","1".getBytes());
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test","2".getBytes());
  Thread.sleep(1000);
  zookeeperCuratorClient.createNode("/glmapper/test/second","data".getBytes(),CreateMode.PERSISTENT);
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test/second","1".getBytes());
  Thread.sleep(1000);
  curatorClient.setData().forPath("/glmapper/test/second","2".getBytes());
  Thread.sleep(1000);
  ```

  测试中控制台输出的信息如下：

  ```java
  -----------------------------
  event:NODE_ADDED
  path:/glmapper
  -----------------------------
  -----------------------------
  event:NODE_ADDED
  path:/glmapper/test
  -----------------------------
  -----------------------------
  event:NODE_UPDATED
  path:/glmapper/test
  -----------------------------
  -----------------------------
  event:NODE_UPDATED
  path:/glmapper/test
  -----------------------------
  -----------------------------
  event:NODE_ADDED
  path:/glmapper/test/second
  -----------------------------
  -----------------------------
  event:NODE_UPDATED
  path:/glmapper/test/second
  -----------------------------
  -----------------------------
  event:NODE_UPDATED
  path:/glmapper/test/second
  -----------------------------
  ```

## 7. 事务操作

CuratorFramework 的实例包含 inTransaction\( \) 接口方法，调用此方法开启一个 ZooKeeper 事务。 可以复合create、 setData、 check、and/or delete 等操作然后调用 commit\(\) 作为一个原子操作提交。

```java
// 开启事务  
CuratorTransaction curatorTransaction = curatorClient.inTransaction();
Collection<CuratorTransactionResult> commit = 
  // 操作1 
curatorTransaction.create().withMode(CreateMode.EPHEMERAL).forPath("/glmapper/transaction")
  .and()
  // 操作2 
  .delete().forPath("/glmapper/test")
  .and()
  // 操作3
  .setData().forPath("/glmapper/transaction", "data".getBytes())
  .and()
  // 提交事务
  .commit();
Iterator<CuratorTransactionResult> iterator = commit.iterator();
while (iterator.hasNext()){
  CuratorTransactionResult next = iterator.next();
  System.out.println(next.getForPath());
  System.out.println(next.getResultPath());
  System.out.println(next.getType());
}
```

这里debug看了下Collection信息，面板如下：

![2020-07-31-mhREde](https://image.ldbmcs.com/2020-07-31-mhREde.jpg)

## 8. 异步操作

前面提到的增删改查都是同步的，但是 Curator 也提供了异步接口，引入了 BackgroundCallback 接口用于处理异步接口调用之后服务端返回的结果信息。BackgroundCallback 接口中一个重要的回调值为 CuratorEvent，里面包含事件类型、响应吗和节点的详细信息。

在使用上也是非常简单的，只需要带上 inBackground\(\) 就行，如下：

```java
 curatorClient.getData().inBackground().forPath("/glmapper/test");
```

通过查看 inBackground 方法定义可以看到，inBackground 支持自定义线程池来处理返回结果之后的业务逻辑。

```java
public T inBackground(BackgroundCallback callback, Executor executor);
```

这里就不贴代码了。

## 9. 小结

本文主要围绕 Curator 的基本 API 进行了学习记录，对于原理及源码部分没有涉及。这部分如果有时间在慢慢研究吧。另外像分布式锁、分布式自增序列等实现停留在理论阶段，没有实践，不敢妄论，用到再码吧。

## 10. 参考

* [www.cnblogs.com/felixzh/p/5…](http://www.cnblogs.com/felixzh/p/5869212.html)
* [my.oschina.net/roccn/blog/…](https://my.oschina.net/roccn/blog/918209)

