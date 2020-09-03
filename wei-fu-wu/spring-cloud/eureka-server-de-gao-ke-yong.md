# Eureka Server的高可用

Eureka Server进行互相注册的方式来实现高可用的部署，所以我们只需要将Eureke Server配置其他可用的serviceUrl就能实现高可用部署。

创建application-peer1.properties，作为peer1服务中心的配置，并将serviceUrl指向peer2。

```text
spring.application.name=eureka-server
server.port=1111
eureka.instance.hostname=peer1

#指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://peer2:1112/eureka/
```

创建application-peer2.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1。

```text
spring.application.name=eureka-server
server.port=1112
eureka.instance.hostname=peer2

#指向另一个注册中心
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/
```

hosts文件中添加对peer1和peer2的转换。

```text
127.0.0.1 peer1
127.0.0.1 peer2
```

通过spring.profiles.active属性来分别启动peer1和peer2。

```java
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer1
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer2
```

此时访问peer1的注册中心：[http://localhost:1111/。registered-replicas中已经有peer2节点的eureka-server了。](http://localhost:1111/。registered-replicas中已经有peer2节点的eureka-server了。)

同样地，访问peer2的注册中心：[http://localhost:1112/，registered-replicas中已经有peer1节点，并且这些节点在可用分片（available-replicase）之中。](http://localhost:1112/，registered-replicas中已经有peer1节点，并且这些节点在可用分片（available-replicase）之中。)

我们也可以尝试关闭peer1，刷新[http://localhost:1112/，peer1的节点变为了不可用分片（unavailable-replicas）。](http://localhost:1112/，peer1的节点变为了不可用分片（unavailable-replicas）。)

服务注册与发现。

```text
pring.application.name=compute-service
server.port=2222
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/,http://peer2:1112/eureka/
```

如何配置serviceUrl来让集群中的服务进行同步 Eureka Server的同步遵循着一个非常简单的原则：只要有一条边将节点连接，就可以进行信息传播与同步

**场景一** 

假设我们有3个注册中心，我们将peer1、peer2、peer3各自都将serviceUrl指向另外两个节点。换言之，peer1、peer2、peer3是两两互相注册的。启动三个服务注册中心，并将compute-service的serviceUrl指向peer1并启动，可以获得如下图所示的集群效果。

![](https://image.ldbmcs.com/2019-07-16-095019.jpg)

访问[http://localhost:1112/，可以看到3个注册中心组成了集群，compute-service服务通过peer1同步给了与之互相注册的peer2和peer3](http://localhost:1112/，可以看到3个注册中心组成了集群，compute-service服务通过peer1同步给了与之互相注册的peer2和peer3)

**总结：** 

两两注册的方式可以实现集群中节点完全对等的效果，实现最高可用性集群，任何一台注册中心故障都不会影响服务的注册与发现。

