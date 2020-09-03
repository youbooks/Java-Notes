## 1. Ribbon 的负载均衡策略

- **RoundRobinRule:** 轮询策略，Ribbon以轮询的方式选择服务器，这个是默认值。
- **RandomRule:** 随机选择，也就是说Ribbon会随机从服务器列表中选择一个进行访问;
- **BestAvailableRule:** 最大可用策略，即先过滤出故障服务器后，选择一个当前并发请求数最小的;
- **WeightedResponseTimeRule:** 带有加权的轮询策略，对各个服务器响应时间进行加权处理，然后在采用轮询的方式来获取相应的服务器;
- **AvailabilityFilteringRule:** 可用过滤策略，先过滤出故障的或并发请求大于阈值一部分服务实例，然后再以线性轮询的方式从过滤后的实例清单中选出一个;
- **ZoneAvoidanceRule:** 区域感知策略，先使用主过滤条件（区域负载器，选择最优区域）对所有实例过滤并返回过滤后的实例清单，依次使用次过滤条件列表中的过滤条件对主过滤条件的结果进行过滤，判断最小过滤数（默认1）和最小过滤百分比（默认0），最后对满足条件的服务器则使用RoundRobinRule(轮询方式)选择一个服务器实例。

## 2. 配置

1. 通过配置文件配置

	```xml
	hello:
    ribbon:
     NFLoadBalancerRuleClassName:com.netflix.loadbalancer.RandomRule
	```
2. 通过java注解配置

	```java
	@Configuration
    public class RibbonConfiguration{
        @Bean
        public IRule ribbonRule(){
            //随机负载
            return new RandomRule();
       }
    }
	```
3. 通过注解 `@RibbonClient` 为特定的服务配置负载均衡策略

	```java
	@Configuration
    @RibbonClient(name="hello", configuration=RibbonConfiguration.class)
    public class TestRibbonConfiguration{
    }
	```

## 3. 自定义负载均衡算法

我们假设自己定义的策略如下：

还是按照轮询的方式来选择服务，但是每个被轮询到的服务，接下来访问4次（默认是1次），4次访问完之后，再切换到下一个服务，访问4次，以此类推。

拿到这个需求之后，我们需要改写策略了，根据官方 github 源码可以知道，类似于轮询、随机这种策略，都是继承了 `AbstractLoadBalancerRule` 类，然后重写 choose 方法。所以，自定义策略分两步走：

1. 实现 AbstractLoadBalancerRule 类
2. 重写 choose 方法

我先把代码复制一下，然后来分析一下：

```java
/**
* 自定义规则
* @author shengwu ni
*/
public class CustomRule extends AbstractLoadBalancerRule {

   /**
    * 总共被调用的次数，目前要求每台被调用4次
     */
   private int total = 0;
   /**
    * 当前提供服务列表的索引
    */
   private int currentIndex = 0;

   @Override public void initWithNiwsConfig(IClientConfig iClientConfig) {
   }

   /**
    * 在choose方法中，自定义我们自己的规则，返回的Server就是具体选择出来的服务
    * 自己的规则：按照轮询的规则，但是每个被轮询到的服务调用5次。
    * @param o
    * @return
    */
   @Override public Server choose(Object o) {
       // 获取负载均衡器lb
       ILoadBalancer lb = getLoadBalancer();
       if (lb == null) {
           return null;
       }

       Server server = null;
       while (server == null) {
           if (Thread.interrupted()) {
               return null;
           }
           // 获取可用服务列表
           List<Server> upList = lb.getReachableServers();
           // 获取所有服务列表
           List<Server> allList = lb.getAllServers();
           int serverCount = allList.size();
           if (serverCount == 0) {
               return null;
           }

           // 若调用次数小于4次，一直调用可用服务列表中索引为 currentIndex 的服务
           if(total < 4)
           {
               server = upList.get(currentIndex);
               total++;
           } else {
               // 到了4次之后，服务列表中的索引值++，表示下一个调用下一个服务
               total = 0;
               currentIndex++;
               // 当索引大于可用服务列表的size时，要重新从头开始
               currentIndex = currentIndex % upList.size();

               if (server == null) {
                   Thread.yield();
                   continue;
               }

               if (server.isAlive()) {
                   return (server);
               }

               server = null;
               Thread.yield();
           }
       }
       return server;
   }
}
```
我来简单分析一下代码：首先获取 ILoadBalancer 对象，该对象可以获取当前的服务。我们需要获取当前可用的服务列表和当前所有的服务列表。

total 表示服务被调用的次数，到4次，该服务调用停止，切换到下一个可用服务；currentIndex 表示当前可用服务列表中的索引。若调用次数小于4次，一直调用可用服务列表中索引为 currentIndex 的服务，到了4次之后，服务列表中的索引值++，表示下一个调用下一个服务。具体可以看代码中的注释。


