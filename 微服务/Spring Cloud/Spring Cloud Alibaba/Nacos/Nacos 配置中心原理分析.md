# Nacos 配置中心原理分析

> 转载：[Nacos 配置中心原理分析](https://www.cnblogs.com/wuzhenzhao/p/11385079.html)

## 1. 配置类型

Spring Cloud Alibaba Nacos Config 目前提供了三种配置能力从 Nacos 拉取相关的配置。

- A: 通过 `spring.cloud.nacos.config.shared-configs[n].data-id` 支持多个共享 Data Id 的配置。
- B: 通过 `spring.cloud.nacos.config.extension-configs[n].data-id` 的方式支持多个扩展Data Id 的配置。
- C: 通过内部相关规则(应用名、应用名+ Profile )自动生成相关的 Data Id 配置。

当三种方式共同使用时，他们的一个优先级关系是: `A < B < C`。

### 1.1 基于dataid为 yaml 的文件扩展配置

spring-cloud-starter-alibaba-nacos-config 默认支持的文件格式是 properties, 如果我们想用其他格式的文件，可以只需要完成以下两步：

1. 在应用的 bootstrap.properties 配置文件中显示的声明 dataid 文件扩展名。如下所示 bootstrap.properties

   ```properties
   spring.cloud.nacos.config.file-extension=yaml
   ```

2. 在Nacos控制台，修改配置文件的类型，改成yml。

### 1.2 针对profile粒度配置

spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataid 为 `${spring.application.name}.${file-extension:properties}` 为前缀的基础配置，还加载了dataid为 `${spring.application.name}-${profile}.${file-extension:properties}` 的基础配置。在日常开发中如果遇到多套环境下的不同配置，可以通过Spring 提供的`${spring.profiles.active}` 这个配置项来配置。

1. 在bootstrap.properties中添加profile

   ```properties
   spring.profiles.active=develop
   ```

2. Nacos 上新增一个dataid为：`wuzz-nacos-dubbo-consumer-develop.yaml` 的基础配置，如下所示：

   ![2021-04-15-n9bGGP](https://image.ldbmcs.com/2021-04-15-n9bGGP.jpg)

   如果需要切换到生产环境，只需要更改 `${spring.profiles.active}` 参数配置即可。如下所示：

   ```properties
   spring.profiles.active=product
   ```

   此案例中我们通过 spring.profiles.active= 的方式写死在配置文件中，而在真正的项目实施过程中这个变量的值是需要不同环境而有不同的值。这个时候通常的做法是通过 -Dspring.profiles.active= 参数指定其配置来达到环境间灵活的切换。

## 2. Nacos 中的Namespace和Group

在nacos中提供了namespace和group命名空间和分组的机制。，它是Nacos提供的一种数据模型，也就是我们要去定位到一个配置，需要基于`namespace- > group ->dataid`来实现。

namespace可以解决多环境以及多租户数据的隔离问题。比如在多套环境下，可以根据指定环境创建不同的namespace，实现多环境隔离。或者在多租户的场景中，每个用户可以维护自己的namespace，实现每个用户的配置数据和注册数据的隔离。

group是分组机制，它的纬度是实现服务注册信息或者DataId的分组管理机制，对于group的用法，没有固定的规则，它也可以实现不同环境下的分组，也可以实现同一个应用下不同配置类型或者不同业务类型的分组。

官方建议是，namespace用来区分不同环境，group可以专注在业务层面的数据分组。实际上在使用过程中，最重要的是提前定要统一的口径和规定，避免不同的项目团队混用导致后期维护混乱的问题。

### 2.1 自定义namespace

在没有明确指定 `${spring.cloud.nacos.config.namespace`} 配置的情况下， 默认使用的是 Nacos上 Public 这个namespae。如果需要使用自定义的命名空间，可以通过以下配置来实现：

```properties
spring.cloud.nacos.config.namespace=b3404bc0-d7dc-4855-b519-570ed34b62d7
```

该配置必须放在 bootstrap.properties 文件中。此外 `spring.cloud.nacos.config.namespace` 的值是 namespace 对应的 id，id 值可以在 Nacos的控制台获取。并且在添加配置时注意不要选择其他的 namespae，否则将会导致读取不到正确的配置。

![2021-04-15-eU1nsi](https://image.ldbmcs.com/2021-04-15-eU1nsi.jpg)

### 2.2 自定义group

在没有明确指定 `${spring.cloud.nacos.config.group}` 配置的情况下， 默认使用的是`DEFAULT_GROUP` 。如果需要自定义自己的 Group，可以通过以下配置来实现：

```properties
spring.cloud.nacos.config.group=DEVELOP_GROUP
```

该配置必须放在 bootstrap.properties 文件中。并且在添加配置时 Group 的值一定要和`spring.cloud.nacos.config.group`的配置值一致。

### 2.3 自定义扩展的DataId

Spring Cloud Alibaba Nacos Config 从 0.2.1 版本后，可支持自定义 Data Id 的配置。关于这部分详细的设计可参考 [这里](https://github.com/alibaba/spring-cloud-alibaba/issues/141)。 一个完整的配置案例如下所示：

```properties
# config external configuration
# 1、Data Id 在默认的组 DEFAULT_GROUP,不支持配置的动态刷新
spring.cloud.nacos.config.extension-configs[0].data-id=ext-config-
common01.properties
# 2、Data Id 不在默认的组，不支持动态刷新
spring.cloud.nacos.config.extension-configs[1].data-id=ext-config-
common02.properties
spring.cloud.nacos.config.extension-configs[1].group=GLOBALE_GROUP
# 3、Data Id 既不在默认的组，也支持动态刷新
spring.cloud.nacos.config.extension-configs[2].data-id=ext-config-
common03.properties
spring.cloud.nacos.config.extension-configs[2].group=REFRESH_GROUP
spring.cloud.nacos.config.extension-configs[2].refresh=true
```

可以看到:

1. 通过 `spring.cloud.nacos.config.extension-configs[n].data-id` 的配置方式来支持多个Data Id 的配置。
2. 通过 `spring.cloud.nacos.config.extension-configs[n].group` 的配置方式自定义 Data Id所在的组，不明确配置的话，默认是 DEFAULT_GROUP。
3. 通过 `spring.cloud.nacos.config.extension-configs[n].refresh` 的配置方式来控制该Data Id 在配置变更时，是否支持应用中可动态刷新， 感知到最新的配置值。默认是不支持的。

多个 Data Id 同时配置时，他的优先级关系是 `spring.cloud.nacos.config.extension-configs[n].data-id` 其中 n 的值越大，优先级越高。

`spring.cloud.nacos.config.extension-configs[n].data-id` 的值必须带文件扩展名，文件扩展名既可支持 properties，又可以支持 yaml/yml。 此时`spring.cloud.nacos.config.file-extension` 的配置对自定义扩展配置的 Data Id 文件扩展名没有影响。

通过自定义扩展的 Data Id 配置，既可以解决多个应用间配置共享的问题，又可以支持一个应用有多个配置文件。

为了更加清晰的在多个应用间配置共享的 Data Id ，你可以通过以下的方式来配置：通过自定义扩展的 Data Id 配置，既可以解决多个应用间配置共享的问题，又可以支持一个应用有多个配置文件。

```properties
# 配置支持共享的 Data Id
spring.cloud.nacos.config.shared-configs[0].data-id=common.yaml
# 配置 Data Id 所在分组，缺省默认 DEFAULT_GROUP
spring.cloud.nacos.config.shared-configs[0].group=GROUP_APP1
# 配置Data Id 在配置变更时，是否动态刷新，缺省默认 false
spring.cloud.nacos.config.shared-configs[0].refresh=true
```

可以看到：

1. 通过 `spring.cloud.nacos.config.shared-configs[n].data-id` 来支持多个共享 Data Id 的配置。
2. 通过 `spring.cloud.nacos.config.shared-configs[n].group` 来配置自定义 Data Id 所在的组，不明确配置的话，默认是 DEFAULT_GROUP。
3. 通过 `spring.cloud.nacos.config.shared-configs[n].refresh` 来控制该Data Id在配置变更时，是否支持应用中动态刷新，默认false。

## 3. Nacos 源码分析

`Environment`，这个是非常重要的类，他负责管理spring的运行相关的配置信息，其中就包含`application.properties`。

而在Spring Cloud中，如果集成Nacos作为配置中心的话，那么意味着这部分配置是属于远程配置，也会作为配置源保存到`Environment`中，这样我们才能通过`@value`注解来注入配置中的属性。

`Environment`中所有外部化配置，针对不同类型的配置都会有与之对应的`PropertySource`，比如（`SystemEnvironmentPropertySource、CommandLinePropertySource`）。以及`PropertySourcesPropertyResolver`来进行解析。

那NacosClient在启动的时候，必然也会需要从远程服务器上获取配置加载到Environment中，这样才能使得应用程序通过@value进行属性的注入，而且我们一定可以猜测到的是，这块的工作一定又和spring中某个机制有关系。

在spring boot项目启动时，有一个`prepareContext`的方法，它会回调所有实现了 `ApplicationContextInitializer` 的实例，来做一些初始化工作。

```java
public ConfigurableApplicationContext run(String... args) {
 //省略代码...
    prepareContext(context, environment, listeners, applicationArguments,
printedBanner);
 //省略代码
  return context;
}
```

`PropertySourceBootstrapConfiguration` 实现了 `ApplicationContextInitialize`r 接口，其目的就是在应用程序上下文初始化的时候做一些额外的操作.，根据默认的 `AnnotationAwareOrderComparator` 排序规则对`propertySourceLocators`数组进行排序，获取运行的环境上下文`ConfigurableEnvironment`。遍历`propertySourceLocators`时：

- 调用 locate 方法，传入获取的上下文environment。
- 将source添加到PropertySource的链表中。
- 设置source是否为空的标识标量empty。source不为空的情况，才会设置到environment中

- 返回Environment的可变形式，可进行的操作如addFirst、addLast。
- 移除propertySources中的bootstrapProperties。
- 根据config server覆写的规则，设置propertySources。
- 处理多个active profiles的配置信息。

```java
public void initialize(ConfigurableApplicationContext applicationContext) {
        CompositePropertySource composite = new CompositePropertySource(
                BOOTSTRAP_PROPERTY_SOURCE_NAME);
        //对propertySourceLocators数组进行排序，根据默认的AnnotationAwareOrderComparator
        AnnotationAwareOrderComparator.sort(this.propertySourceLocators);
        boolean empty = true;
        //获取运行的环境上下文
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        for (PropertySourceLocator locator : this.propertySourceLocators) {
            //回调所有实现PropertySourceLocator接口实例的locate方法，
            PropertySource<?> source = null;
            source = locator.locate(environment);
            if (source == null) {
                continue;
            }
            logger.info("Located property source: " + source);
            composite.addPropertySource(source);//将source添加到数组
            empty = false;//表示propertysource不为空
        }
        if (!empty) {//只有propertysource不为空的情况，才会设置到environment中
            MutablePropertySources propertySources = environment.getPropertySources();
            String logConfig = environment.resolvePlaceholders("${logging.config:}");
            LogFile logFile = LogFile.get(environment);
            if (propertySources.contains(BOOTSTRAP_PROPERTY_SOURCE_NAME)) {
                propertySources.remove(BOOTSTRAP_PROPERTY_SOURCE_NAME);
            }
            insertPropertySources(propertySources, composite);
            reinitializeLoggingSystem(environment, logConfig, logFile);
            setLogLevels(applicationContext, environment);
            handleIncludedProfiles(environment);
        }
}
```

`locator.locate(environment)`：这个方法会调用子类的locate方法，来获得一个`PropertySource`，然后将PropertySource集合返回。接着它会调用 `ConfigServicePropertySourceLocator` 的locate方法。

![2021-04-15-VnluiK](https://image.ldbmcs.com/2021-04-15-VnluiK.jpg)

我们可以看到 spring-cloud-config也是实现了这个接口，Nacos 也是一样的。

`NacosPropertySourceLocator.locate`：这个就是Nacos 配置中心加载的的关键实现了，分别调用三个方法来加载配置。

Nacos配置加载顺序：`共享配置 --> 扩展配置 --> 自身配置（后面优先级高）` , 这三个配置在前面的内容中我们已经讲过了.

```java
public PropertySource<?> locate(Environment env) {
        nacosConfigProperties.setEnvironment(env);
        ConfigService configService = nacosConfigManager.getConfigService();

        if (null == configService) {
            log.warn("no instance of config service found, can't load config from nacos");
            return null;
        }
        long timeout = nacosConfigProperties.getTimeout();
        nacosPropertySourceBuilder = new NacosPropertySourceBuilder(configService,
                timeout);
        String name = nacosConfigProperties.getName();

        String dataIdPrefix = nacosConfigProperties.getPrefix();
        if (StringUtils.isEmpty(dataIdPrefix)) {
            dataIdPrefix = name;
        }

        if (StringUtils.isEmpty(dataIdPrefix)) {
            dataIdPrefix = env.getProperty("spring.application.name");
        }

        CompositePropertySource composite = new CompositePropertySource(
                NACOS_PROPERTY_SOURCE_NAME);
　　　　　//加载共享配置
        loadSharedConfiguration(composite);
　　　　 //加载扩展配置
        loadExtConfiguration(composite);
　　　　 //加载自身配置
        loadApplicationConfiguration(composite, dataIdPrefix, nacosConfigProperties, env);

        return composite;
}
```

`loadApplicationConfiguration`: 我们可以先不管加载共享配置、扩展配置的方法，最终本质上都是去远程服务上读取配置，只是传入的参数不一样。

- fileExtension，表示配置文件的扩展名
- nacosGroup表示分组
- 加载 dataid=项目名称 的配置
- 加载 dataid=项目名称+扩展名 的配置
- 遍历当前配置的激活点（profile），分别循环加载带有profile的dataid配置

```java
private void loadApplicationConfiguration(
            CompositePropertySource compositePropertySource, String dataIdPrefix,
            NacosConfigProperties properties, Environment environment) {
        //获取配置文件的扩展名
        String fileExtension = properties.getFileExtension();
        // 获取配置的分组
        String nacosGroup = properties.getGroup();
        // load directly once by default 加载存在的配置信息 可以发现这里会加载带跟不带prefix的配置
        loadNacosDataIfPresent(compositePropertySource, dataIdPrefix, nacosGroup,
                fileExtension, true);
        // load with suffix, which have a higher priority than the default
        loadNacosDataIfPresent(compositePropertySource,
                dataIdPrefix + DOT + fileExtension, nacosGroup, fileExtension, true);
        // Loaded with profile, which have a higher priority than the suffix
        for (String profile : environment.getActiveProfiles()) {
            String dataId = dataIdPrefix + SEP1 + profile + DOT + fileExtension;
            loadNacosDataIfPresent(compositePropertySource, dataId, nacosGroup,
                    fileExtension, true);
        }

}
```

`loadNacosDataIfPresent`：调用 `loadNacosPropertySource` 加载存在的配置信息。把加载之后的配置属性保存到CompositePropertySource中。

```java
private void loadNacosDataIfPresent(final CompositePropertySource composite,
            final String dataId, final String group, String fileExtension,
            boolean isRefreshable) {
        if (null == dataId || dataId.trim().length() < 1) {
            return;
        }
        if (null == group || group.trim().length() < 1) {
            return;
        }
        NacosPropertySource propertySource = this.loadNacosPropertySource(dataId, group,
                fileExtension, isRefreshable);
        this.addFirstPropertySource(composite, propertySource, false);
}
```

做一些简单的校验，然后进入 `loadNacosPropertySource`.

```java
private NacosPropertySource loadNacosPropertySource(final String dataId,
            final String group, String fileExtension, boolean isRefreshable) {
        //是否支持自动刷新，// 如果不支持自动刷新配置则自动从缓存获取返回

        if (NacosContextRefresher.getRefreshCount() != 0) {
            if (!isRefreshable) {
                return NacosPropertySourceRepository.getNacosPropertySource(dataId,
                        group);
            }
        }
        //构造器从配置中心获取数据
        return nacosPropertySourceBuilder.build(dataId, group, fileExtension,
                isRefreshable);
}
NacosPropertySource build(String dataId, String group, String fileExtension,
            boolean isRefreshable) {
　　//调用loadNacosData加载远程数据
　　Map<String, Object> p = loadNacosData(dataId, group, fileExtension);
　　NacosPropertySource nacosPropertySource = new NacosPropertySource(group, dataId,
                p, new Date(), isRefreshable);
　　NacosPropertySourceRepository.collectNacosPropertySource(nacosPropertySource);
　　return nacosPropertySource;
}
```

loadNacosData：加载Nacos的数据。

```java
private Map<String, Object> loadNacosData(String dataId, String group,
            String fileExtension) {
        String data = null;
        try {// http远程访问配置中心，获取配置数据
            data = configService.getConfig(dataId, group, timeout);
            if (StringUtils.isEmpty(data)) {//如果为空，则提示日志
                log.warn(
                        "Ignore the empty nacos configuration and get it based on dataId[{}] & group[{}]",
                        dataId, group);
                return EMPTY_MAP;
            }
            if (log.isDebugEnabled()) {
                log.debug(String.format(
                        "Loading nacos data, dataId: '%s', group: '%s', data: %s", dataId,
                        group, data));
            }//根据扩展名进行数据的解析
            Map<String, Object> dataMap = NacosDataParserHandler.getInstance()
                    .parseNacosData(data, fileExtension);
            return dataMap == null ? EMPTY_MAP : dataMap;
        }
        catch (NacosException e) {
            log.error("get data from Nacos error,dataId:{}, ", dataId, e);
        }
        catch (Exception e) {
            log.error("parse data from Nacos error,dataId:{},data:{},", dataId, data, e);
        }
        return EMPTY_MAP;
    }
```

继续往下跟踪，最终进入到getConfigInner方法，主要有几个逻辑

1. 先从本地磁盘中加载配置，因为应用在启动时，会加载远程配置缓存到本地，如果本地文件的内容不为空，直接返回。
2. 如果本地文件的内容为空，则调用worker.getServerConfig加载远程配置
3. 如果出现异常，则调用本地快照文件加载配置

```java
private String getConfigInner(String tenant, String dataId, String group, long timeoutMs) throws NacosException {
        group = null2defaultGroup(group); // 获取group
        ParamUtils.checkKeyParam(dataId, group);
        ConfigResponse cr = new ConfigResponse();

        cr.setDataId(dataId);//dataID
        cr.setTenant(tenant);
        cr.setGroup(group);

        // 优先使用本地配置
        String content = LocalConfigInfoProcessor.getFailover(agent.getName(), dataId, group, tenant);
        if (content != null) {
            LOGGER.warn("[{}] [get-config] get failover ok, dataId={}, group={}, tenant={}, config={}", agent.getName(),
                dataId, group, tenant, ContentUtils.truncateContent(content));
            cr.setContent(content);
            configFilterChainManager.doFilter(null, cr);
            content = cr.getContent();
            return content;
        }

        try {//获取远程配置
            content = worker.getServerConfig(dataId, group, tenant, timeoutMs);

            cr.setContent(content);
            configFilterChainManager.doFilter(null, cr);
            content = cr.getContent();

            return content;
        } catch (NacosException ioe) {
            if (NacosException.NO_RIGHT == ioe.getErrCode()) {
                throw ioe;
            }
            LOGGER.warn("[{}] [get-config] get from server error, dataId={}, group={}, tenant={}, msg={}",
                agent.getName(), dataId, group, tenant, ioe.toString());
        }

        LOGGER.warn("[{}] [get-config] get snapshot ok, dataId={}, group={}, tenant={}, config={}", agent.getName(),
            dataId, group, tenant, ContentUtils.truncateContent(content));
        //获取快照配置，返回兜底数据
        content = LocalConfigInfoProcessor.getSnapshot(agent.getName(), dataId, group, tenant);
        cr.setContent(content);
        configFilterChainManager.doFilter(null, cr);
        content = cr.getContent();
        return content;
    }
```

`worker.getServerConfig`：通过agent.httpGet发起http请求，获取远程服务的配置。这个agent的创建在NacosConfigService类的构造方法：

```java
public NacosConfigService(Properties properties) throws NacosException {
        String encodeTmp = properties.getProperty(PropertyKeyConst.ENCODE);
        if (StringUtils.isBlank(encodeTmp)) {
            encode = Constants.ENCODE;
        } else {
            encode = encodeTmp.trim();
        }//初始化命名空间
        initNamespace(properties);
        agent = new MetricsHttpAgent(new ServerHttpAgent(properties));
        agent.start();
        worker = new ClientWorker(agent, configFilterChainManager, properties);
}
```

### 3.1 客户端配置的动态感知

在上述 NacosConfigService的构造方法中，当这个类被实例化以后，有做一些事情初始化一个HttpAgent，这里又用到了装饰起模式，实际工作的类是ServerHttpAgent,MetricsHttpAgent内部也是调用了ServerHttpAgent的方法，增加了监控统计的信息ClientWorker， 客户端的一个工作类，agent作为参数传入到clientworker，可以基本猜测到里面会用到agent做一些远程相关的事情

这一步主要初始化了 agent 与 worker 两个实例。这里又看到熟悉的包装器模式，将ServerHttpAgent 包装成MetricsHttpAgent，这里我们需要知道，其中MetricsHttpAgent是对ServerHttpAgent功能的拓展，核心功能还是由ServerHttpAgent去实现，接下去我们来看一下 worker 的初始化，从名字上看能知道 最后真的工作的是他：

```java
public ClientWorker(final HttpAgent agent, final ConfigFilterChainManager configFilterChainManager, final Properties properties) {
        this.agent = agent;
        this.configFilterChainManager = configFilterChainManager;
        // Initialize the timeout parameter
        // 初始化一些参数
        init(properties);
        //创建了一个定时任务的线程池第一个线程池是只拥有一个线程用来执行定时任务的 executor，executor         //每隔 10ms 就会执行一次checkConfigInfo() 方法，从方法名上可以知道是每 10 ms 检查一次配置信息。
        executor = Executors.newScheduledThreadPool(1, new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setName("com.alibaba.nacos.client.Worker." + agent.getName());
                t.setDaemon(true);
                return t;
            }
        });
        //创建了一个保持长连接的线程池
        executorService = Executors.newScheduledThreadPool(Runtime.getRuntime().availableProcessors(), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setName("com.alibaba.nacos.client.Worker.longPolling." + agent.getName());
                t.setDaemon(true);
                return t;
            }
        });
        //创建了一个延迟任务线程池来每隔10ms来检查配置信息的线程池
        executor.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                try {
                    checkConfigInfo();
                } catch (Throwable e) {
                    LOGGER.error("[" + agent.getName() + "] [sub-check] rotate check error", e);
                }
            }
        }, 1L, 10L, TimeUnit.MILLISECONDS);
}
```

这一步创建了两个线程池，第一个线程池负责与配置中心进行数据的交互，并且启动后延迟1ms，之后每隔10ms对配置信息进行定时检查，第二个线程池则是负责保持一个长连接。我们再服务启动之后便会执行 checkConfigInfo()，跟进去看看：

```java
public void checkConfigInfo() {
        // 分任务（解决大数据量的传输问题）
        int listenerSize = cacheMap.get().size();
        // 向上取整为批数，分批次进行检查
        // ParamUtil.getPerTaskConfigSize() =3000
        int longingTaskCount = (int) Math.ceil(listenerSize / ParamUtil.getPerTaskConfigSize());
        // currentLongingTaskCount =0
        if (longingTaskCount > currentLongingTaskCount) {
            for (int i = (int) currentLongingTaskCount; i < longingTaskCount; i++) {
                // 要判断任务是否在执行 这块需要好好想想。 任务列表现在是无序的。变化过程可能有问题
                executorService.execute(new LongPollingRunnable(i));
            }
            currentLongingTaskCount = longingTaskCount;
        }
    }
```

这个方法主要的目的是用来检查服务端的配置信息是否发生了变化。如果有变化，则触发listener通知 `cacheMap: AtomicReference<Map<String, CacheData>> cacheMap` 用来存储监听变更的缓存集合。key是根据`dataID/group/tenant(租户)` 拼接的值。Value是对应存储在nacos服务器上的配置文件的内容。默认情况下，每个长轮训`LongPullingRunnable`任务默认处理3000个监听配置集。如果超过3000， 则需要启动多个`LongPollingRunnable`去执行。

初始化`new LongPollingRunnable()`丢给 `executorService`线程池来处理，所以我们可以找到LongPollingRunnable里面的run方法这个方法传递了一个taskid， tasked用来区分cacheMap中的任务批次, 保存到cacheDatas这个集合中。`cacheData.isUseLocalConfigInfo` 这个值的变化来自于`checkLocalConfig`这个方法

```java
public void run() {
            List<CacheData> cacheDatas = new ArrayList<CacheData>();
            List<String> inInitializingCacheList = new ArrayList<String>();
            try {
                // check failover config  tasked用来区分cacheMap中的任务批次, 保存到cacheDatas这个集合中
                for (CacheData cacheData : cacheMap.get().values()) {
                    if (cacheData.getTaskId() == taskId) {
                        cacheDatas.add(cacheData);
                        try {
                            //检查本地配置,通过本地文件中缓存的数据和cacheData集合中的数据进行比对，判断是否出现数据变化
                            checkLocalConfig(cacheData);
                            if (cacheData.isUseLocalConfigInfo()) {//这里表示数据有变化，需要通知监听器
　　　　　　　　　　　　　　　　　　  //检查缓存的MD5
                                cacheData.checkListenerMd5();
                            }
                        } catch (Exception e) {
                            LOGGER.error("get local config info error", e);
                        }
                    }
                }

                //检查服务端配置
                List<String> changedGroupKeys = checkUpdateDataIds(cacheDatas, inInitializingCacheList);

                for (String groupKey : changedGroupKeys) {
                    String[] key = GroupKey.parseKey(groupKey);
                    String dataId = key[0];
                    String group = key[1];
                    String tenant = null;
                    if (key.length == 3) {
                        tenant = key[2];
                    }
                    try {
                        String content = getServerConfig(dataId, group, tenant, 3000L);
                        //将配置设置进缓存
                        CacheData cache = cacheMap.get().get(GroupKey.getKeyTenant(dataId, group, tenant));
                        cache.setContent(content);
                        LOGGER.info("[{}] [data-received] dataId={}, group={}, tenant={}, md5={}, content={}",
                            agent.getName(), dataId, group, tenant, cache.getMd5(),
                            ContentUtils.truncateContent(content));
                    } catch (NacosException ioe) {
                        String message = String.format(
                            "[%s] [get-update] get changed config exception. dataId=%s, group=%s, tenant=%s",
                            agent.getName(), dataId, group, tenant);
                        LOGGER.error(message, ioe);
                    }
                }
                for (CacheData cacheData : cacheDatas) {
                    if (!cacheData.isInitializing() || inInitializingCacheList
                        .contains(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant))) {
                        cacheData.checkListenerMd5();
                        cacheData.setInitializing(false);
                    }
                }
                inInitializingCacheList.clear();

                executorService.execute(this);

            } catch (Throwable e) {

                // If the rotation training task is abnormal, the next execution time of the task will be punished
                LOGGER.error("longPolling error : ", e);
                executorService.schedule(this, taskPenaltyTime, TimeUnit.MILLISECONDS);
            }
        }
```

总的来说，该方法主要流程是先检查本地缓存，再检查服务端的配置，由改变最后再回写到本地及加载到缓存。

checkLocalConfig检查本地配置，这里面有三种情况：

1. 如果`isUseLocalConfigInfo`为false，但是本地缓存路径的文件是存在的，那么把isUseLocalConfigInfo设置为true，并且更新cacheData的内容以及文件的更新时间
2. 如果`isUseLocalCOnfigInfo`为true，但是本地缓存文件不存在，则设置为false，不通知监听器
3. `isUseLocalConfigInfo`为true，并且本地缓存文件也存在，但是缓存的的时间和文件的更新时间不一致，则更新cacheData中的内容，并且isUseLocalConfigInfo设置为true

```java
private void checkLocalConfig(CacheData cacheData) {
        final String dataId = cacheData.dataId;
        final String group = cacheData.group;
        final String tenant = cacheData.tenant;　　　　 //本地文件缓存
        File path = LocalConfigInfoProcessor.getFailoverFile(agent.getName(), dataId, group, tenant);
        // 没有 -> 有
        //不使用本地配置，但是持久化文件存在，需要读取文件加载至内存
        if (!cacheData.isUseLocalConfigInfo() && path.exists()) {
            String content = LocalConfigInfoProcessor.getFailover(agent.getName(), dataId, group, tenant);
            String md5 = MD5.getInstance().getMD5String(content);
            cacheData.setUseLocalConfigInfo(true);
            cacheData.setLocalConfigInfoVersion(path.lastModified());
            cacheData.setContent(content);

            LOGGER.warn("[{}] [failover-change] failover file created. dataId={}, group={}, tenant={}, md5={}, content={}",
                agent.getName(), dataId, group, tenant, md5, ContentUtils.truncateContent(content));
            return;
        }

        // 有 -> 没有。不通知业务监听器，从server拿到配置后通知。
        //使用本地配置，但是持久化文件不存在 
        if (cacheData.isUseLocalConfigInfo() && !path.exists()) {
            cacheData.setUseLocalConfigInfo(false);
            LOGGER.warn("[{}] [failover-change] failover file deleted. dataId={}, group={}, tenant={}", agent.getName(),
                dataId, group, tenant);
            return;
        }

        // 有变更
        //使用本地配置，持久化文件存在，缓存跟文件最后修改时间不一致
        if (cacheData.isUseLocalConfigInfo() && path.exists()
            && cacheData.getLocalConfigInfoVersion() != path.lastModified()) {
            String content = LocalConfigInfoProcessor.getFailover(agent.getName(), dataId, group, tenant);
            String md5 = MD5.getInstance().getMD5String(content);
            cacheData.setUseLocalConfigInfo(true);
            cacheData.setLocalConfigInfoVersion(path.lastModified());
            cacheData.setContent(content);
            LOGGER.warn("[{}] [failover-change] failover file changed. dataId={}, group={}, tenant={}, md5={}, content={}",
                agent.getName(), dataId, group, tenant, md5, ContentUtils.truncateContent(content));
        }
    }
```

本地检查主要是通过是否使用本地配置，继而寻找持久化缓存文件，再通过判断文件的最后修改事件与本地缓存的版本是否一致来判断是否由变更。本地检查完毕，如果使用本地配置会进入下列代码：

```java
if (cacheData.isUseLocalConfigInfo()) {
　　  //检查缓存的MD5
       cacheData.checkListenerMd5();
}
void checkListenerMd5() {
    for (ManagerListenerWrap wrap : listeners) {
　　　　  //MD5由变更，说明数据变更
        if (!md5.equals(wrap.lastCallMd5)) {
　　　　　　　 //执行回调
            safeNotifyListener(dataId, group, content, md5, wrap);
        }
    }
}
```

### 3.2 检查服务端配置

在LongPollingRunnable.run中，先通过本地配置的读取和检查来判断数据是否发生变化从而实现变化的通知。

接着，当前的线程还需要去远程服务器上获得最新的数据，检查哪些数据发生了变化：

- 通过checkUpdateDataIds获取远程服务器上数据变更的dataid
- 遍历这些变化的集合，然后调用getServerConfig从远程服务器获得对应的内容
- 更新本地的cache，设置为服务器端返回的内容
- 最后遍历cacheDatas，找到变化的数据进行通知

```java
//检查服务端配置
List<String> changedGroupKeys = checkUpdateDataIds(cacheDatas, inInitializingCacheList);
```

这里会去获取一个发生变化的GroupKeys 集合：

```java
/**
     * 从Server获取值变化了的DataID列表。返回的对象里只有dataId和group是有效的。 保证不返回NULL。
     */
    List<String> checkUpdateDataIds(List<CacheData> cacheDatas, List<String> inInitializingCacheList) throws IOException {
        StringBuilder sb = new StringBuilder();
        for (CacheData cacheData : cacheDatas) {
            if (!cacheData.isUseLocalConfigInfo()) {
                sb.append(cacheData.dataId).append(WORD_SEPARATOR);
                sb.append(cacheData.group).append(WORD_SEPARATOR);
                if (StringUtils.isBlank(cacheData.tenant)) {
                    sb.append(cacheData.getMd5()).append(LINE_SEPARATOR);
                } else {
                    sb.append(cacheData.getMd5()).append(WORD_SEPARATOR);
                    sb.append(cacheData.getTenant()).append(LINE_SEPARATOR);
                }
                if (cacheData.isInitializing()) {
                    // cacheData 首次出现在cacheMap中&首次check更新
                    inInitializingCacheList
                        .add(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant));
                }
            }
        }
        boolean isInitializingCacheList = !inInitializingCacheList.isEmpty();
        return checkUpdateConfigStr(sb.toString(), isInitializingCacheList);
    }
```

通过长轮训的方式，从远程服务器获得变化的数据进行返回。这里将可能发生变化的配置信息封装成一个 StringBuilder ，继而调用 `checkUpdateConfigStr`：

```java
/**
     * 从Server获取值变化了的DataID列表。返回的对象里只有dataId和group是有效的。 保证不返回NULL。
     */
    List<String> checkUpdateConfigStr(String probeUpdateString, boolean isInitializingCacheList) throws IOException {

        List<String> params = Arrays.asList(Constants.PROBE_MODIFY_REQUEST, probeUpdateString);

        List<String> headers = new ArrayList<String>(2);
        headers.add("Long-Pulling-Timeout");
        headers.add("" + timeout);

        // told server do not hang me up if new initializing cacheData added in
        if (isInitializingCacheList) {
            headers.add("Long-Pulling-Timeout-No-Hangup");
            headers.add("true");
        }

        if (StringUtils.isBlank(probeUpdateString)) {
            return Collections.emptyList();
        }

        try {//发起一个Post请求
            HttpResult result = agent.httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params,
                agent.getEncode(), timeout);

            if (HttpURLConnection.HTTP_OK == result.code) {
                setHealthServer(true);
                return parseUpdateDataIdResponse(result.content);
            } else {
                setHealthServer(false);
                LOGGER.error("[{}] [check-update] get changed dataId error, code: {}", agent.getName(), result.code);
            }
        } catch (IOException e) {
            setHealthServer(false);
            LOGGER.error("[" + agent.getName() + "] [check-update] get changed dataId exception", e);
            throw e;
        }
        return Collections.emptyList();
    }
```

就这样从Server获取值变化了的DataID列表。返回的对象里只有dataId和group是有效的。 保证不返回NULL。获取到这个列表以后就便利这个列表，去服务器端获取对应变更后的配置：

```java
for (String groupKey : changedGroupKeys) {
   String[] key = GroupKey.parseKey(groupKey);
   String dataId = key[0];
   String group = key[1];
   String tenant = null;
   if (key.length == 3) {
         tenant = key[2];
   }
   try {//遍历有变换的groupkey，发起远程请求获得指定groupkey的内容
      String content = getServerConfig(dataId, group, tenant, 3000L);
      //将配置设置进缓存
      CacheData cache = cacheMap.get().get(GroupKey.getKeyTenant(dataId, group, tenant));
      cache.setContent(content);
      LOGGER.info("[{}] [data-received] dataId={}, group={}, tenant={}, md5={}, content={}",
         agent.getName(), dataId, group, tenant, cache.getMd5(),
         ContentUtils.truncateContent(content));
   } catch (NacosException ioe) {
     String message = String.format(
        "[%s] [get-update] get changed config exception. dataId=%s, group=%s, tenant=%s",
        agent.getName(), dataId, group, tenant);
        LOGGER.error(message, ioe);
   }
}
```

这里会发起请求从服务器端获取配置：`getServerConfig`

```java
public String getServerConfig(String dataId, String group, String tenant, long readTimeout)
        throws NacosException {
        if (StringUtils.isBlank(group)) {
            group = Constants.DEFAULT_GROUP;
        }

        HttpResult result = null;
        try {
            List<String> params = null;
            if (StringUtils.isBlank(tenant)) {
                params = Arrays.asList("dataId", dataId, "group", group);
            } else {
                params = Arrays.asList("dataId", dataId, "group", group, "tenant", tenant);
            }
            result = agent.httpGet(Constants.CONFIG_CONTROLLER_PATH, null, params, agent.getEncode(), readTimeout);
        } catch (IOException e) {
            String message = String.format(
                "[%s] [sub-server] get server config exception, dataId=%s, group=%s, tenant=%s", agent.getName(),
                dataId, group, tenant);
            LOGGER.error(message, e);
            throw new NacosException(NacosException.SERVER_ERROR, e);
        }

        switch (result.code) {
            case HttpURLConnection.HTTP_OK:
                LocalConfigInfoProcessor.saveSnapshot(agent.getName(), dataId, group, tenant, result.content);
                return result.content;
            case HttpURLConnection.HTTP_NOT_FOUND:
                LocalConfigInfoProcessor.saveSnapshot(agent.getName(), dataId, group, tenant, null);
                return null;
            case HttpURLConnection.HTTP_CONFLICT: {
                LOGGER.error(
                    "[{}] [sub-server-error] get server config being modified concurrently, dataId={}, group={}, "
                        + "tenant={}", agent.getName(), dataId, group, tenant);
                throw new NacosException(NacosException.CONFLICT,
                    "data being modified, dataId=" + dataId + ",group=" + group + ",tenant=" + tenant);
            }
            case HttpURLConnection.HTTP_FORBIDDEN: {
                LOGGER.error("[{}] [sub-server-error] no right, dataId={}, group={}, tenant={}", agent.getName(), dataId,
                    group, tenant);
                throw new NacosException(result.code, result.content);
            }
            default: {
                LOGGER.error("[{}] [sub-server-error]  dataId={}, group={}, tenant={}, code={}", agent.getName(), dataId,
                    group, tenant, result.code);
                throw new NacosException(result.code,
                    "http error, code=" + result.code + ",dataId=" + dataId + ",group=" + group + ",tenant=" + tenant);
            }
        }
    }
```

通过初始化时候的 agent.httpGet 去发起一个Get请求，就这样变更本例的配置，当从远程服务器获取玩配置以后还有一个循环：

```java
for (CacheData cacheData : cacheDatas) {//再遍历CacheData这个集合，找到发生变化的数据进行通知
  if (!cacheData.isInitializing() || inInitializingCacheList
          .contains(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant))) {
      cacheData.checkListenerMd5();
      cacheData.setInitializing(false);
  }
}
```

这个循环主要是对有变化的配置进行监听回调。整个流程就差不都完成了，最后来一张流程图：

![2021-04-15-OQ7MRN](https://image.ldbmcs.com/2021-04-15-OQ7MRN.jpg)

### 3.3 长轮训的时间间隔

我们知道客户端会有一个长轮训的任务去检查服务器端的配置是否发生了变化，如果发生了变更，那么客户端会拿到变更的 groupKey 再根据 groupKey 去获取配置项的最新值更新到本地的缓存以及文件中，那么这种每次都靠客户端去请求，那请求的时间间隔设置多少合适呢？

如果间隔时间设置的太长的话有可能无法及时获取服务端的变更，如果间隔时间设置的太短的话，那么频繁的请求对于服务端来说无疑也是一种负担，所以最好的方式是客户端每隔一段长度适中的时间去服务端请求，而在这期间如果配置发生变更，服务端能够主动将变更后的结果推送给客户端，这样既能保证客户端能够实时感知到配置的变化，也降低了服务端的压力。 我们来看看nacos设置的间隔时间是多久。

#### 3.3.1 长轮训的概念

客户端发起一个请求到服务端，服务端收到客户端的请求后，并不会立刻响应给客户端，而是先把这个请求hold住，然后服务端会在hold住的这段时间检查数据是否有更新，如果有，则响应给客户端，如果一直没有数据变更，则达到一定的时间（长轮训时间间隔）才返回。

长轮训典型的场景有： 扫码登录、扫码支付。

![2021-04-15-q1e3ii](https://image.ldbmcs.com/2021-04-15-q1e3ii.jpg)

#### 3.3.2 客户端长轮训

在ClientWorker这个类里面，找到 checkUpdateConfigStr 这个方法，这里面就是去服务器端查询发生变化的groupKey。

```java
/**
* 从Server获取值变化了的DataID列表。返回的对象里只有dataId和group是有效的。 保证不返回NULL。
*/
List<String> checkUpdateConfigStr(String probeUpdateString, boolean isInitializingCacheList) throws IOException {

        List<String> params = Arrays.asList(Constants.PROBE_MODIFY_REQUEST, probeUpdateString);

        List<String> headers = new ArrayList<String>(2);
        headers.add("Long-Pulling-Timeout");
        headers.add("" + timeout);

        // told server do not hang me up if new initializing cacheData added in
        if (isInitializingCacheList) {
            headers.add("Long-Pulling-Timeout-No-Hangup");
            headers.add("true");
        }

        if (StringUtils.isBlank(probeUpdateString)) {
            return Collections.emptyList();
        }

        try {//客户端发送的请求地址是: /v1/cs/configs/listener
            HttpResult result = agent.httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params,
                agent.getEncode(), timeout);

            if (HttpURLConnection.HTTP_OK == result.code) {
                setHealthServer(true);
                return parseUpdateDataIdResponse(result.content);
            } else {
                setHealthServer(false);
                LOGGER.error("[{}] [check-update] get changed dataId error, code: {}", agent.getName(), result.code);
            }
        } catch (IOException e) {
            setHealthServer(false);
            LOGGER.error("[" + agent.getName() + "] [check-update] get changed dataId exception", e);
            throw e;
        }
        return Collections.emptyList();
    }
```

这个方法最终会发起http请求，注意这里面有一个 timeout 的属性，

```java
HttpResult result = agent.httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params,
                agent.getEncode(), timeout);
```

timeout是在init这个方法中赋值的，默认情况下是30秒，可以通过`configLongPollTimeout`进行修改。

```java
private void init(Properties properties) {
        // 默认长轮询的事件就是30S
        timeout = Math.max(NumberUtils.toInt(properties.getProperty(PropertyKeyConst.CONFIG_LONG_POLL_TIMEOUT),
            //public static final int CONFIG_LONG_POLL_TIMEOUT = 30000;
            //public static final int MIN_CONFIG_LONG_POLL_TIMEOUT = 10000;
            Constants.CONFIG_LONG_POLL_TIMEOUT), Constants.MIN_CONFIG_LONG_POLL_TIMEOUT);

        taskPenaltyTime = NumberUtils.toInt(properties.getProperty(PropertyKeyConst.CONFIG_RETRY_TIME), Constants.CONFIG_RETRY_TIME);

        enableRemoteSyncConfig = Boolean.parseBoolean(properties.getProperty(PropertyKeyConst.ENABLE_REMOTE_SYNC_CONFIG));
    }
```

所以从这里得出的一个基本结论是：客户端发起一个轮询请求，超时时间是30s。 那么客户端为什么要等待30s才超时呢？不是越快越好吗？ 我们可以在nacos的日志目录下 `$NACOS_HOME/nacos/logs/config-client-request.log` 文件.

![2021-04-15-nVu1rA](https://image.ldbmcs.com/2021-04-15-nVu1rA.jpg)

可以看到一个现象，在配置没有发生变化的情况下，客户端会等29.5s以上，才请求到服务器端的结果。然后客户端拿到服务器端的结果之后，在做后续的操作。当服务器端频繁的修改，那么服务器端频繁客户端进行推送.

### 3.4 服务端的处理

服务端是如何处理客户端的请求的？那么同样，我们需要思考几个问题：

- 客户端的长轮训响应时间受到哪些因素的影响
- 客户端的超时时间为什么要设置30s
- 客户端发送的请求地址是: /v1/cs/configs/listener 找到服务端对应的方法

nacos是使用spring mvc提供的rest api，其中有个类是 ConfigController ，我们在其中找到了Post 请求的  listener 路径的接口方法：

```java
/**
     * 比较MD5
     */
    @RequestMapping(value = "/listener", method = RequestMethod.POST)
    public void listener(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        request.setAttribute("org.apache.catalina.ASYNC_SUPPORTED", true);
        String probeModify = request.getParameter("Listening-Configs");
        if (StringUtils.isBlank(probeModify)) {
            throw new IllegalArgumentException("invalid probeModify");
        }

        probeModify = URLDecoder.decode(probeModify, Constants.ENCODE);

        Map<String, String> clientMd5Map;
        try {
            clientMd5Map = MD5Util.getClientMd5Map(probeModify);
        } catch (Throwable e) {
            throw new IllegalArgumentException("invalid probeModify");
        }

        // do long-polling
        inner.doPollingConfig(request, response, clientMd5Map, probeModify.length());
    }
```

先是获取了客户端的MD5集合，这里面会调用inner.doPollingConfig进行处理,这个方法中，兼容了长轮训和短轮询的逻辑，我们只需要关注长轮训的部分:

```java
/**
     * 轮询接口
     */
    public String doPollingConfig(HttpServletRequest request, HttpServletResponse response,
                                  Map<String, String> clientMd5Map, int probeRequestSize)
        throws IOException, ServletException {

        // 长轮询
        if (LongPollingService.isSupportLongPolling(request)) {
            longPollingService.addLongPollingClient(request, response, clientMd5Map, probeRequestSize);
            return HttpServletResponse.SC_OK + "";
        }　　　　......//省略代码　　}
```

这里我们进入长轮询的代码块：

```java
public void addLongPollingClient(HttpServletRequest req, HttpServletResponse rsp, Map<String, String> clientMd5Map,
                                     int probeRequestSize) {
　　　　  //超时时间
        String str = req.getHeader(LongPollingService.LONG_POLLING_HEADER);
        String noHangUpFlag = req.getHeader(LongPollingService.LONG_POLLING_NO_HANG_UP_HEADER);
        String appName = req.getHeader(RequestUtil.CLIENT_APPNAME_HEADER);
        String tag = req.getHeader("Vipserver-Tag");
        int delayTime = SwitchService.getSwitchInteger(SwitchService.FIXED_DELAY_TIME, 500);
        /**
         * 提前500ms返回响应，为避免客户端超时 @qiaoyi.dingqy 2013.10.22改动  add delay time for LoadBalance
         */
        long timeout = Math.max(10000, Long.parseLong(str) - delayTime);
        if (isFixedPolling()) {
            timeout = Math.max(10000, getFixedPollingInterval());
            // do nothing but set fix polling timeout
        } else {
            long start = System.currentTimeMillis();
            List<String> changedGroups = MD5Util.compareMd5(req, rsp, clientMd5Map);
            if (changedGroups.size() > 0) {
                generateResponse(req, rsp, changedGroups);
                LogUtil.clientLog.info("{}|{}|{}|{}|{}|{}|{}",
                    System.currentTimeMillis() - start, "instant", RequestUtil.getRemoteIp(req), "polling",
                    clientMd5Map.size(), probeRequestSize, changedGroups.size());
                return;
            } else if (noHangUpFlag != null && noHangUpFlag.equalsIgnoreCase(TRUE_STR)) {
                LogUtil.clientLog.info("{}|{}|{}|{}|{}|{}|{}", System.currentTimeMillis() - start, "nohangup",
                    RequestUtil.getRemoteIp(req), "polling", clientMd5Map.size(), probeRequestSize,
                    changedGroups.size());
                return;
            }
        }
        String ip = RequestUtil.getRemoteIp(req);
        // 一定要由HTTP线程调用，否则离开后容器会立即发送响应
        final AsyncContext asyncContext = req.startAsync();
        // AsyncContext.setTimeout()的超时时间不准，所以只能自己控制
        asyncContext.setTimeout(0L);

        scheduler.execute(
            new ClientLongPolling(asyncContext, clientMd5Map, ip, probeRequestSize, timeout, appName, tag));
    }
```

这个方法是把客户端的长轮训请求添加到任务中去。

1. 获得客户端传递过来的超时时间，并且进行本地计算，提前500ms返回响应，这就能解释为什么客户端响应超时时间是29.5+了。当然如果 isFixedPolling=true 的情况下，不会提前返回响应
2. 根据客户端请求过来的md5和服务器端对应的group下对应内容的md5进行比较，如果不一致，则通过 generateResponse 将结果返回
3. 如果配置文件没有发生变化，则通过 scheduler.execute 启动了一个定时任务，将客户端的长轮询请求封装成一个叫 ClientLongPolling 的任务，交给 scheduler 去执行

那么接下去一定会进入ClientLongPolling 的Run 方法:

```java
public void run() {
            asyncTimeoutFuture = scheduler.schedule(new Runnable() {
                @Override
                public void run() {
                    try {
                        getRetainIps().put(ClientLongPolling.this.ip, System.currentTimeMillis());
                        /**
                         * 删除订阅关系
                         */
                        allSubs.remove(ClientLongPolling.this);

                        if (isFixedPolling()) {
                            LogUtil.clientLog.info("{}|{}|{}|{}|{}|{}",
                                (System.currentTimeMillis() - createTime),
                                "fix", RequestUtil.getRemoteIp((HttpServletRequest)asyncContext.getRequest()),
                                "polling",
                                clientMd5Map.size(), probeRequestSize);
                            List<String> changedGroups = MD5Util.compareMd5(
                                (HttpServletRequest)asyncContext.getRequest(),
                                (HttpServletResponse)asyncContext.getResponse(), clientMd5Map);　　　　　　　　　　　　　　　　　//有变化立即执行返回
                            if (changedGroups.size() > 0) {
                                sendResponse(changedGroups);
                            } else {
                                sendResponse(null);
                            }
                        } else {
                            LogUtil.clientLog.info("{}|{}|{}|{}|{}|{}",
                                (System.currentTimeMillis() - createTime),
                                "timeout", RequestUtil.getRemoteIp((HttpServletRequest)asyncContext.getRequest()),
                                "polling",
                                clientMd5Map.size(), probeRequestSize);
                            sendResponse(null);
                        }
                    } catch (Throwable t) {
                        LogUtil.defaultLog.error("long polling error:" + t.getMessage(), t.getCause());
                    }

                }
　　　　　　　　　//延迟29.5秒后执行
            }, timeoutTime, TimeUnit.MILLISECONDS);

            allSubs.add(this);
        }
```

这个任务要阻塞29.5s才能执行，因为立马执行没有任何意义，毕竟前面已经执行过一次了如果在29.5s+之内，数据发生变化，需要提前通知。需要有一种监控机制

在run方法中，通过scheduler.schedule实现了一个定时任务，它的delay时间正好是前面计算的29.5s。在这个任务中，会通过MD5Util.compareMd5来进行计算那另外一个，当数据发生变化以后，肯定不能等到29.5s之后才通知呀，那怎么办呢？我们发现有一个allSubs 的东西，它似乎和发布订阅有关系。那是不是有可能当前的clientLongPolling订阅了数据变化的事件呢？allSubs是一个队列，队列里面放了ClientLongPolling这个对象。这个队列似乎和配置变更有某种关联关系：

```java
/**
 * 长轮询订阅关系
 */
final Queue<ClientLongPolling> allSubs;
```

注释里写明了他是和长轮询订阅相关的，接着我们先来看一下他所归属的类的类图：

![2021-04-15-eMhe6L](https://image.ldbmcs.com/2021-04-15-eMhe6L.jpg)

那么这里必须要实现的是，当用户在nacos 控制台修改了配置之后，必须要从这个订阅关系中取出关注的客户端长连接，然后把变更的结果返回。于是我们去看LongPollingService的构造方法查找订阅关系发现`LongPollingService`集成了`AbstractEventListener`，事件监听.

`AbstractEventListener`:

```java
static public abstract class AbstractEventListener {

        public AbstractEventListener() {
            /**
             * automatic register
             */
            EventDispatcher.addEventListener(this);
        }

        /**
         * 感兴趣的事件列表
         *
         * @return event list
         */
        abstract public List<Class<? extends Event>> interest();

        /**
         * 处理事件
         *
         * @param event event
         */
        abstract public void onEvent(Event event);
    }
```

这里面有一个抽象的onEvent方法，明显是用来处理事件的方法，而抽象方法必须由子类实现，所以意味着LongPollingService里面肯定实现了onEvent方法，我们可以在其构造方法中发现。

```java
public void onEvent(Event event) {
        if (isFixedPolling()) {
            // ignore
        } else {
            if (event instanceof LocalDataChangeEvent) {
                LocalDataChangeEvent evt = (LocalDataChangeEvent)event;
                scheduler.execute(new DataChangeTask(evt.groupKey, evt.isBeta, evt.betaIps));
            }
        }
    }
```

所以到了这里，肯定是修改了配置之后会有一个触发点去出发该事件，当匹配上事件类型，那么就会去执行这个回调，这个事件的实现方法中判断事件类型是否为`LocalDataChangeEvent`，通过`scheduler.execute`执行`DataChangeTask`这个任务。

从名字可以看出来，这个是数据变化的任务，最让人兴奋的应该是，它里面有一个循环迭代器，从allSubs里面获得`ClientLongPolling`。

```java
public void run() {
            try {
                ConfigService.getContentBetaMd5(groupKey);
                for (Iterator<ClientLongPolling> iter = allSubs.iterator(); iter.hasNext(); ) {
                    ClientLongPolling clientSub = iter.next();
                    if (clientSub.clientMd5Map.containsKey(groupKey)) {
                        // 如果beta发布且不在beta列表直接跳过
                        if (isBeta && !betaIps.contains(clientSub.ip)) {
                            continue;
                        }

                        // 如果tag发布且不在tag列表直接跳过
                        if (StringUtils.isNotBlank(tag) && !tag.equals(clientSub.tag)) {
                            continue;
                        }

                        getRetainIps().put(clientSub.ip, System.currentTimeMillis());
                        iter.remove(); // 删除订阅关系
                        LogUtil.clientLog.info("{}|{}|{}|{}|{}|{}|{}",
                            (System.currentTimeMillis() - changeTime),
                            "in-advance",
                            RequestUtil.getRemoteIp((HttpServletRequest)clientSub.asyncContext.getRequest()),
                            "polling",
                            clientSub.clientMd5Map.size(), clientSub.probeRequestSize, groupKey);
                        clientSub.sendResponse(Arrays.asList(groupKey));
                    }
                }
            } catch (Throwable t) {
                LogUtil.defaultLog.error("data change error:" + t.getMessage(), t.getCause());
            }
        }
```

这个是数据变化的任务，最让人兴奋的应该是，它里面有一个循环迭代器，从allSubs里面获得`ClientLongPolling`最后通过`clientSub.sendResponse`把数据返回到客户端。所以，这也就能够理解为何数据变化能够实时触发更新了。

那么接下来还有一个疑问是，数据变化之后是如何触发事件的呢？ 所以我们定位到数据变化的请求类中，在ConfigController这个类中，找到POST请求的方法找到配置变更的位置：

```java
/**
     * 增加或更新非聚合数据。
     *
     * @throws NacosException
     */
    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    public Boolean publishConfig(HttpServletRequest request, HttpServletResponse response,
                                 @RequestParam("dataId") String dataId, @RequestParam("group") String group,
                                 @RequestParam(value = "tenant", required = false, defaultValue = StringUtils.EMPTY)
                                     String tenant,
                                 @RequestParam("content") String content,
                                 @RequestParam(value = "tag", required = false) String tag,
                                 @RequestParam(value = "appName", required = false) String appName,
                                 @RequestParam(value = "src_user", required = false) String srcUser,
                                 @RequestParam(value = "config_tags", required = false) String configTags,
                                 @RequestParam(value = "desc", required = false) String desc,
                                 @RequestParam(value = "use", required = false) String use,
                                 @RequestParam(value = "effect", required = false) String effect,
                                 @RequestParam(value = "type", required = false) String type,
                                 @RequestParam(value = "schema", required = false) String schema)
        throws NacosException {
        final String srcIp = RequestUtil.getRemoteIp(request);
        String requestIpApp = RequestUtil.getAppName(request);
        ParamUtils.checkParam(dataId, group, "datumId", content);
        ParamUtils.checkParam(tag);

        Map<String, Object> configAdvanceInfo = new HashMap<String, Object>(10);
　　　　　......//省略代码
        final Timestamp time = TimeUtils.getCurrentTime();
        String betaIps = request.getHeader("betaIps");
        ConfigInfo configInfo = new ConfigInfo(dataId, group, tenant, appName, content);
        if (StringUtils.isBlank(betaIps)) {
            if (StringUtils.isBlank(tag)) {
                persistService.insertOrUpdate(srcIp, srcUser, configInfo, time, configAdvanceInfo, false);
                EventDispatcher.fireEvent(new ConfigDataChangeEvent(false, dataId, group, tenant, time.getTime()));
            } else {
                persistService.insertOrUpdateTag(configInfo, tag, srcIp, srcUser, time, false);
                EventDispatcher.fireEvent(new ConfigDataChangeEvent(false, dataId, group, tenant, tag, time.getTime()));
            }
        } else { // beta publish
            persistService.insertOrUpdateBeta(configInfo, betaIps, srcIp, srcUser, time, false);
            EventDispatcher.fireEvent(new ConfigDataChangeEvent(true, dataId, group, tenant, time.getTime()));
        }
        ConfigTraceService.logPersistenceEvent(dataId, group, tenant, requestIpApp, time.getTime(),
            LOCAL_IP, ConfigTraceService.PERSISTENCE_EVENT_PUB, content);

        return true;
    }
```

发现数据持久化之后，会通过`EventDispatcher`进行事件发布`EventDispatcher.fireEvent` 但是这个事件似乎不是我们所关心的时间，原因是这里发布的事件是`ConfigDataChangeEvent` , 而LongPollingService感兴趣的事件是` LocalDataChangeEvent`。

在Nacos中有一个DumpService，它会定时把变更后的数据dump到磁盘上，DumpService在spring启动之后，会调用init方法启动几个dump任务。然后在任务执行结束之后，会触发一个`LocalDataChangeEvent` 的事件：

```java
@PostConstruct
public void init() {
    LogUtil.defaultLog.warn("DumpService start");
    DumpProcessor processor = new DumpProcessor(this);
    DumpAllProcessor dumpAllProcessor = new DumpAllProcessor(this);
    DumpAllBetaProcessor dumpAllBetaProcessor = new DumpAllBetaProcessor(this);
    DumpAllTagProcessor dumpAllTagProcessor = new DumpAllTagProcessor(this);　　......//省略代码}
```

其中在 DumpProcessor的 process方法中会调用 ConfigService 的相关API对数据进行操作，其中调用 remove 后会传播这么一个事件：

```java
/**
     * 删除配置文件，删除缓存。
     */
    static public boolean remove(String dataId, String group, String tenant) {
        final String groupKey = GroupKey2.getKey(dataId, group, tenant);
        final int lockResult = tryWriteLock(groupKey);
        /**
         *  数据不存在
         */
        if (0 == lockResult) {
            dumpLog.info("[remove-ok] {} not exist.", groupKey);
            return true;
        }
        /**
         * 加锁失败
         */
        if (lockResult < 0) {
            dumpLog.warn("[remove-error] write lock failed. {}", groupKey);
            return false;
        }

        try {
            if (!STANDALONE_MODE || PropertyUtil.isStandaloneUseMysql()) {
                DiskUtil.removeConfigInfo(dataId, group, tenant);
            }
            CACHE.remove(groupKey);
            EventDispatcher.fireEvent(new LocalDataChangeEvent(groupKey));

            return true;
        } finally {
            releaseWriteLock(groupKey);
        }
    }
```

简单总结一下刚刚分析的整个过程。

- 客户端发起长轮训请求，
- 服务端收到请求以后，先比较服务端缓存中的数据是否相同，如果不通，则直接返回
- 如果相同，则通过schedule延迟29.5s之后再执行比较
- 为了保证当服务端在29.5s之内发生数据变化能够及时通知给客户端，服务端采用事件订阅的方式来监听服务端本地数据变化的事件，一旦收到事件，则触发DataChangeTask的通知，并且遍历allStubs队列中的ClientLongPolling,把结果写回到客户端，就完成了一次数据的推送
- 如果 DataChangeTask 任务完成了数据的 “推送” 之后，ClientLongPolling 中的调度任务又开始执行了怎么办呢？很简单，只要在进行 “推送” 操作之前，先将原来等待执行的调度任务取消掉就可以了，这样就防止了推送操作写完响应数据之后，调度任务又去写响应数据，这时肯定会报错的。所以，在ClientLongPolling方法中，最开始的一个步骤就是删除订阅事件

所以总的来说，Nacos采用推+拉的形式，来解决最开始关于长轮训时间间隔的问题。当然，30s这个时间是可以设置的，而之所以定30s，应该是一个经验值。