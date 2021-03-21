> 转载：[SpringBoot 使用ApplicationListener监听器](https://my.oschina.net/dslcode/blog/1591523)

## 1. 使用场景

在一些业务场景中，当Serverlet容器初始化完成、重启、关闭等等一系列动作之后，需要处理一些操作，比如一些数据的加载、初始化缓存、特定任务的注册等等。这个时候我们就可以使用Spring提供的`ApplicationListener`来进行操作。

## 2. 原理

ApplicationListener是一个接口，里面只有一个`onApplicationEvent`方法，方法的参数为`ApplicationEvent`，ApplicationEvent是个抽象类，顾名思义就是Spring应用的一些Event，ApplicationEvent又有一个抽象子类`ApplicationContextEvent`，这里用到的就是ApplicationContextEvent的实现类。查看ApplicationContextEvent文档的实现类，有[ContextClosedEvent](https://docs.spring.io/spring/docs/4.3.13.RELEASE/javadoc-api/org/springframework/context/event/ContextClosedEvent.html), [ContextRefreshedEvent](https://docs.spring.io/spring/docs/4.3.13.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html), [ContextStartedEvent](https://docs.spring.io/spring/docs/4.3.13.RELEASE/javadoc-api/org/springframework/context/event/ContextStartedEvent.html), [ContextStoppedEvent](https://docs.spring.io/spring/docs/4.3.13.RELEASE/javadoc-api/org/springframework/context/event/ContextStoppedEvent.html)，这四个都是spring context包中的核心实现：

![2020-09-29-ql3bKC](https://image.ldbmcs.com/2020-09-29-ql3bKC.jpg)

```java
ApplicationEvent
  |-ApplicationContextEvent
    |-ContextClosedEvent：应用关闭事件
    |-ContextRefreshedEvent：应用刷新事件
    |-ContextStartedEvent：应用开启事件
    |-ContextStoppedEvent：应用停止事件
```

另外：spring boot扩展了两个实现：

- `EmbeddedServletContainerInitializedEvent`：内嵌Servlet容器初始化事件。
- `ApplicationReadyEvent`：spring Application启动完成事件。

## 3. 用法

本文以在Spring boot下的使用为例来进行说明。首先，需要实现ApplicationListener接口并实现onApplicationEvent方法。把需要处理的操作放在onApplicationEvent中进行处理：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.embedded.EmbeddedServletContainerInitializedEvent;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextClosedEvent;
import org.springframework.context.event.ContextRefreshedEvent;
import org.springframework.context.event.ContextStartedEvent;
import org.springframework.context.event.ContextStoppedEvent;

/**
 * Created by dongsilin on 2017/12/18.
 */
@Slf4j
public class AppApplicationListener implements ApplicationListener {

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        log.info("+++++++++++++++++++++++++++++++++++++++++++++");
        if (event instanceof ContextStartedEvent){
            log.info("================:{}", "ContextStartedEvent");
        }
        if (event instanceof ContextRefreshedEvent){
            log.info("================:{}", "ContextRefreshedEvent");
        }
        if (event instanceof ContextClosedEvent){
            log.info("================:{}", "ContextClosedEvent");
        }
        if (event instanceof ContextStoppedEvent){
            log.info("================:{}", "ContextStoppedEvent");
        }
        if (event instanceof EmbeddedServletContainerInitializedEvent){
            log.info("================:{}", "EmbeddedServletContainerInitializedEvent");
        }
        if (event instanceof ApplicationReadyEvent){
            log.info("================:{}", "ApplicationReadyEvent");
        }
        log.info(">>>>>>>>>>>>>>>>:{}\n", event.getClass().getName());
    }

}
```

```java
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}


	@Bean
	public AppApplicationListener appApplicationListener(){
		return new AppApplicationListener();
	}
}
```

启动运行结果：

![2020-09-29-PNDiEc](https://image.ldbmcs.com/2020-09-29-PNDiEc.jpg)

关闭运行结果：

![2020-09-29-GhhUrF](https://image.ldbmcs.com/2020-09-29-GhhUrF.jpg)

## 4. 二次调用问题

此处使用Spring boot来进行操作，没有出现二次调用的问题。在使用传统的`applicationContext.xml`和`project-servlet.xml`配置中会出现二次调用的问题。主要原因是初始化root容器之后，会初始化project-servlet.xml对应的子容器。我们需要的是只执行一遍即可。那么上面打印父容器的代码用来进行判断排除子容器即可。在业务处理之前添加如下判断：

```java
if(contextRefreshedEvent.getApplicationContext().getParent() != null){
            return;
}
```

这样其他容器的初始化就会直接返回，而父容器（Parent为null的容器）启动时将会执行相应的业务操作。

## 5. 关联知识

在spring中InitializingBean接口也提供了类似的功能，只不过它进行操作的时机是在所有bean都被实例化之后才进行调用。根据不同的业务场景和需求，可选择不同的方案来实现。

## 6. 使用场景

```java
@Configuration
public class InitProperties implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    private static final Logger log = LoggerFactory.getLogger(InitProperties.class);
    private static final String nacos_fat = "172.27.0.4:8848";
    private static final String nacos_prod = "119.27.187.243:8848";
    private static final String nacos_dev = "118.24.108.222:8848";
    private static final String zipkin_fat = "http://118.24.108.222:9411";
    private static final String zipkin_prod = "http://119.27.187.243:9411";
    private static final Integer redis_min_idle = 0;
    private static final Integer redis_max_idle = 8;
    private static final Integer redis_max_active = 8;
    private static final Integer redis_max_wait = 1000;

    public InitProperties() {
    }

    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        ConfigurableEnvironment environment = event.getEnvironment();
        String[] actives = environment.getActiveProfiles();
        String active = "fat";
        if (actives != null && actives.length > 0) {
            active = actives[0];
        }

        Properties props = new Properties();
        props.put("nacos.config.server-addr", "172.27.0.4:8848");
        props.put("server.shutdown", "GRACEFUL");
        props.put("management.endpoints.web.exposure.include", "*");
        props.put("management.endpoint.health.show-details", "always");
        props.put("spring.jackson.time-zone", "GMT+8");
        props.put("spring.jackson.serialization.write-dates-as-timestamps", true);
        props.put("spring.cloud.nacos.discovery.heart-beat-interval", 1);
        props.put("spring.cloud.nacos.discovery.heart-beat-timeout", 1);
        props.put("spring.cloud.nacos.discovery.ip-delete-timeout", 1);
        if ("prod".equals(active)) {
            props.put("spring.cloud.nacos.discovery.server-addr", "119.27.187.243:8848");
            props.put("spring.cloud.nacos.discovery.namespace", "prod");
            props.put("spring.zipkin.baseUrl", "http://119.27.187.243:9411");
            props.put("swagger.production", true);
            props.put("spring.redis.host", "10.0.0.2");
            props.put("spring.redis.port", "6379");
            props.put("spring.redis.password", "oLM0Bh!zn450");
            props.put("spring.redis.lettuce.pool.max-active", redis_max_active);
            props.put("spring.redis.lettuce.pool.max-idle", redis_max_idle);
            props.put("spring.redis.lettuce.pool.max-wait", redis_max_wait);
            props.put("spring.redis.lettuce.pool.min-idle", redis_min_idle);
            log.info("===>配置 prod 环境默认参数");
        } else if ("dev".equals(active)) {
            props.put("nacos.config.server-addr", "139.155.27.221:8848");
            props.put("spring.cloud.nacos.discovery.server-addr", "118.24.108.222:8848");
            props.put("spring.cloud.nacos.discovery.namespace", "dev");
            props.put("spring.redis.host", "118.24.108.222");
            props.put("spring.redis.port", "6379");
            props.put("spring.redis.password", "lgl250??");
            props.put("spring.redis.lettuce.pool.max-active", redis_max_active);
            props.put("spring.redis.lettuce.pool.max-idle", redis_max_idle);
            props.put("spring.redis.lettuce.pool.max-wait", redis_max_wait);
            props.put("spring.redis.lettuce.pool.min-idle", redis_min_idle);
            props.put("spring.zipkin.baseUrl", "http://118.24.108.222:9411");
            log.info("===>配置 dev 环境默认参数");
        }

        environment.getPropertySources().addFirst(new PropertiesPropertySource("nacos-config", props));
    }
}
```

spring.factories

```
org.springframework.context.ApplicationListener = \
  com.zto.business.baseconfig.properties.InitProperties
```

