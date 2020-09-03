# Service Provider Interface\(SPI\)详解

## 1. 介绍

熟悉JDBC的同学都知道，在jdbc4.0之前，在使用`DriverManager`获取DB连接之前，我们总是需要显示的实例化DB驱动。比如，对mysql，典型的代码如下：

```java
Connection conn = null;
Statement stmt = null;
try{
    // 注册 JDBC driver
    Class.forName("com.mysql.jdbc.Driver");

    // 打开连接
    conn = DriverManagger.getConnection(DB_URL,USER,PASSWD);

    // 执行一条sql
    stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery(sql);

    // 数据解包
    while(ts.next()){
        // 根据列名获取列值
        // ...
    } catch(SQLException se) {
        // ...
    } final {
        try {
            if (stmt!=null) stmt.close();
        } catch(Exception e) {/*ignored*/}
        try {
            if (conn!=null) conn.close();
        } catch(Exception e) {/*ignored*/}
    }
}
```

JDBC的开始，总是需要通过`Class.forName`显式实例化驱动，否则将找不到对应DB的驱动。但是JDBC4.0开始，这个显式的初始化不再是必选项了，它存在的意义只是为了向上兼容。那么JDBC4.0之后，我们的应用是如何找到对应的驱动呢？

答案就是**SPI**（Service Provider Interface）。Java在语言层面为我们提供了一种方便地创建可扩展应用的途径。SPI提供了一种JVM级别的服务发现机制，我们只需要**按照SPI的要求，在jar包中进行适当的配置，jvm就会在运行时通过懒加载，帮我们找到所需的服务并加载**。如果我们一直不使用某个服务，那么它不会被加载，一定程度上避免了资源的浪费。

## 2. 一个简单的例子

我们通过一个简单的例子看看如何最小化构建一个基于SPI的服务。

### 2.1 创建一个默认的maven项目

```bash
$ mvn archetype:generate -DgroupId=cn.jinlu.spi.demo -DartifactId=simplespi -Dversion=0.1-SNAPSHOT -DpackageName=cn.jinlu.spi.demo -DarchetypeArtifactId=maven-archetype-quickstart
...
[INFO] Using property: groupId = cn.jinlu.spi.demo
[INFO] Using property: artifactId = simplespi
[INFO] Using property: version = 0.1-SNAPSHOT
[INFO] Using property: package = cn.jinlu.spi.demo
Confirm properties configuration:
groupId: cn.jinlu.spi.demo
artifactId: simplespi
version: 0.1-SNAPSHOT
package: cn.jinlu.spi.demo
 Y: :[回车]
```

### 2.2 添加一个interface或abstract class

Java SPI并没有强制必须使用interface或abstract class，完全可以将class注册为SPI注册服务，但是作为可扩展服务，使用interface或abstract class是一个好习惯。

在包 “`cn.jinlu.spi.demo`”中定义一个接口Animal：

```java
package cn.jinlu.spi.demo;

public interface Animal {
    void eat();
    void sleep();
}
```

### 2.3 提供实现类

```java
package cn.jinlu.spi.demo.impl;

import cn.jinlu.spi.demo.Animal;

public class Elephant implements Animal {
    @Override
    public void eat() {
        System.out.println("Elephant is eating");
    }

    @Override
    public void sleep() {
        System.out.println("Elephant is sleeping");
    }
}
```

### 2.4 服务注册

在main目录下创建目录 `"resources/META-INF/services"`。

```bash
mkdir -p resources/META-INF/services
```

再在该目录下创建以接口Animal全限定名为名的配置文件，文件内容为该接口的实现类的全限定名，即：

```bash
echo "cn.jinlu.spi.demo.impl.Elephant" > resources/META-INF/services/cn.jinlu.spi.demo.Animal
```

完成此步骤后，在当前maven项目的 src/main/resources/META-INF/services下有这么一个配置文件："cn.jinlu.spi.demo.Animal"，并且它的内容为"cn.jinlu.spi.demo.impl.Elephant"。

**注意本步骤的要点：**

1. 必须放在JAR包或项目的指定路径，即 META-INF/services 下。
2. 必须以服务的全限定名命名配置文件，比如本例中，配置文件必须命名为 cn.jinlu.spi.demo.Animal，java会根据此名进行服务查找。
3. 内容必须是一个实现类的全限定名，如果要注册多个实现类，按行分割。注释以\#开头。

### 2.5 增加单元测试

注意，如果找不到@Test，可能是junit版本太低，在pom.xml中将其改为 4.0 或更高版本（maven-archetype-quickstart模板默认的JUNIT目前是3.8.1版本）。

```java
package cn.jinlu.spi.demo;

import org.junit.Test;

import java.util.ServiceLoader;

public class AnimalTest {
    @Test
    public void animalTest() {
        ServiceLoader<Animal> animals = ServiceLoader.load(Animal.class);
        for(Animal animal: animals) {
            animal.eat();
            animal.sleep();
        }
    }
}
```

### 2.6 执行结果

```text
Elephant is eating
Elephant is sleeping
```

可见，虽然我们没有显式使用Animal的实现类Elephant，但是java帮我们自动加载了改实现类。

## 3. 源码分析

接下来从代码层面看看SPI都为我们做了什么。首先看看`java.util.ServiceLoader`的实现。在2.5节中，我们看到ServiceLoader使用非常简单，只需要调用一个静态方法load并以要加载的服务的父类（通常是一个interface或abstract class）作为参数，jvm就会帮我们构建好当前进程中所有注册到 `META-INF/services/[service full qualified class name]`的服务。

### 3.1 创建ServiceLoader实例

下面是构造ServiceLoader实例的相关代码。ServiceLoader必须通过静态方法load\(Class&lt;?&gt; service\)的方式加载服务，默认会使用当前线程的上下文class loader。构造完ServiceLoader后，ServiceLoader实例并不会立刻扫描当前进程中的服务实例，而是创建一个LazyIterator懒加载迭代器，在实际使用时再扫描所有jar包找到对应的服务。懒加载迭代器被保存在一个内部成员lookupIterator中。

```java
public final class ServiceLoader<S> implements Iterable<S>
{
    ...
    /**
     * 重新load指定serivice的实现。通过LazyIterator实现懒加载。
     */
    public void reload() {
        providers.clear();
        lookupIterator = new LazyIterator(service, loader);
    }
    /**
     * ServiceLoader构造函数，私有类型，必须通过ServiceLoader.load(Class<?>)静态方法来创建ServiceLoader实例
     */
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }
    /**
     * 构建ServiceLoader实例
     */
    public static <S> ServiceLoader<S> load(Class<S> service,
            ClassLoader loader)
    {
        return new ServiceLoader<>(service, loader);
    }
    /**
     * 通过service的class创建ServiceLoader实例，默认使用上下文classloader
     */
    public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
    ...
}
```

### 3.2 服务加载和遍历

上一节\(3.1\)的代码中，我们可以看到，在调用了`ServiceLoader<Animal> animals = ServiceLoader.load(Animal.class)`之后，`ServiceLoader`会返回一个`Animal.class`类型的迭代器，但此时在`ServiceLoader`内部只是创建了一个`LazyIterator`，而不会真正通过classloader在classpath中寻找相关的服务实现。

相反，`ServiceLoader`通过实现`Iterable<?>`接口（`public final class ServiceLoader<S> implements Iterable<S>`），将对服务实现的寻址延后倒了对`animals`的遍历时执行。它在`ServiceLoader`内通过`LazyIterator`实现。

```java
public final class ServiceLoader<S> implements Iterable<S>
{
    ...
    // 缓存的service provider，按照初始化顺序排列。
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();
    // 当前的LazyIterator迭代器指针，服务懒加载迭代器
    private LazyIterator lookupIterator;

    ...
    // 创建ServiceLoader迭代器，隐藏了LazyIterator的实现细节
    public Iterator<S> iterator() {
        return new Iterator<S>() {

            // 创建Iterator迭代器时的ServiceLoader.providers快照，
            // 因此在首次迭代时，iterator总是会通过LazyIterator进行懒加载
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                // 如果已经扫描过，则对providers进行迭代；
                if (knownProviders.hasNext())
                    return true;
                // 如果没有扫描过，则通过lookupIterator进行扫描和懒加载
                return lookupIterator.hasNext();
            }

            public S next() {
                // 如果已经扫描过，则对providers进行迭代；
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                // 如果没有扫描过，则通过lookupIterator进行扫描和懒加载
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }
    ...
}
```

**ServiceLoader的迭代器很简单：**

1. 未进行迭代操作时，不对jar包作任何扫描
2. 首次迭代时，因为`ServiceLoader.providers`中没有任何缓存，总是会通过LazyIterator进行懒加载，并将service实现的全限定名与加载的service实例作为key-value缓存到ServiceLoader.providers中。
3. 之后再进行迭代时，总是在ServiceLoader.providers中进行。

### 3.3 懒加载迭代器LazyIterator

懒加载迭代器LazyIterator主要实现以下功能：

1. 首次迭代时，通过ClassLoader.getResources\(String\)获得指定services文件的URL集合
2. 如果是首次遍历懒加载器，或者对上一个URL内容解析获得的service实现类集合完成了迭代，则从configs中取下一个services文件URL进行解析，按行获得具体的service实现类集合，并进行迭代。
3. 对当前URL中解析得到的实现类集合进行迭代，每次返回一个service实现类。

下面是LazyIterator的源码及注释：

```java
public final class ServiceLoader<S> implements Iterable<S>
{
    private static final String PREFIX = "META-INF/services/";
    ...

    // Private inner class implementing fully-lazy provider lookup
    private class LazyIterator implements Iterator<S>
    {

        Class<S> service;
        ClassLoader loader;
        Enumeration<URL> configs = null;
        // 当前service配置文件的内容迭代器
        // 即对services进行遍历，取出一个services配置文件，再对该文件按行解析，每行代表一个具体的service实现类，pending是某个services配置文件中service实现类的迭代器
        Iterator<String> pending = null;
        String nextName = null;

        private LazyIterator(Class<S> service, ClassLoader loader) {
            this.service = service;
            this.loader = loader;
        }

        private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            // 首次迭代时，configs为空，尝试通过classloader获取名为：
            // "META-INF/services/[服务全限定名]"的所有配置文件
            if (configs == null) {
                try {
                    // 注意fullName的定义:"META-INF/services/[服务全限定名]"
                    String fullName = PREFIX + service.getName();
                    // 通过ClassLoader.getResources()获得资源URL集合
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            // 如果pending为空，或者pending已经迭代到迭代器末尾，则尝试解析下一个services配置文件
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                pending = parse(service, configs.nextElement());
            }
            // 对当前pending内容进行遍历，每一项代表services的一个实现类
            nextName = pending.next();
            return true;
        }
    }

    ...
}
```

最后，附上parse及parseLine的代码，可以发现，parseLine中会对服务实现类进行去重，所以在一个或多个services配置文件中配置多次的服务实现类只会被处理一次。

```java
public final class ServiceLoader<S> implements Iterable<S>
{
    ...
    // 按行解析给定配置文件。如果解析出的服务实现类没有被其他已解析的配置文件配置过，则通过参数nams返回给parse方法
    //
    private int parseLine(Class<?> service, URL u, BufferedReader r, int lc, List<String> names) throws IOException, ServiceConfigurationError {
        String ln = r.readLine();
        if (ln == null) {
            return -1;
        }
        int ci = ln.indexOf('#');
        if (ci >= 0) ln = ln.substring(0, ci);
        ln = ln.trim();
        int n = ln.length();
        if (n != 0) {
            if ((ln.indexOf(' ') >= 0) || (ln.indexOf('\t') >= 0))
                fail(service, u, lc, "Illegal configuration-file syntax");
            int cp = ln.codePointAt(0);
            if (!Character.isJavaIdentifierStart(cp))
                fail(service, u, lc, "Illegal provider-class name: " + ln);
            for (int i = Character.charCount(cp); i < n; i += Character.charCount(cp)) {
                cp = ln.codePointAt(i);
                if (!Character.isJavaIdentifierPart(cp) && (cp != '.'))
                    fail(service, u, lc, "Illegal provider-class name: " + ln);
            }
            // 去重，防止重复配置服务，每个服务实现类只会被解析一次
            if (!providers.containsKey(ln) && !names.contains(ln))
                names.add(ln);
        }
        return lc + 1;
    }

    /**
     * 解析指定的作为SPI配置文件的URL的内容
     */
    private Iterator<String> parse(Class<?> service, URL u) throws ServiceConfigurationError {
        InputStream in = null;
        BufferedReader r = null;
        ArrayList<String> names = new ArrayList<>();
        try {
            in = u.openStream();
            r = new BufferedReader(new InputStreamReader(in, "utf-8"));
            int lc = 1;
            while ((lc = parseLine(service, u, r, lc, names)) >= 0);
        } catch (IOException x) {
            fail(service, "Error reading configuration file", x);
        } finally {
            try {
                if (r != null) r.close();
                if (in != null) in.close();
            } catch (IOException y) {
                fail(service, "Error closing configuration file", y);
            }
        }
        return names.iterator();
    }
    ...
}
```

## 4. JDBC中对SPI的使用

最后，以JDBC为例，看一个SPI的实际使用场景。在文章开始，我们提到过，JDBC4.0之前，我们总是需要在业务代码中显式地实例化DB驱动实现类：

```java
Class.forName("com.mysql.jdbc.Driver");
```

为什么JDBC4.0之后不需要了呢？答案就在下面的代码中。**在系统启动时，DriverManager静态初始化时会通过ServiceLoader对所有jar包中被注册为 java.sql.Driver 服务的驱动实现类进行初始化**，这样就避免了上面通过Class.forName手动初始化的繁琐工作。

```java
public class DriverManager {

    // JDBC驱动注册中心，所有加载的JDBC驱动都注册在该CopyOnWriteArrayList中
    private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<>();

    ...

    /* Prevent the DriverManager class from being instantiated. */
    private DriverManager(){}

    /**
     * Load the initial JDBC drivers by checking the System property
     * jdbc.properties and then use the {@code ServiceLoader} mechanism
     */
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

    private static void loadInitialDrivers() {
        // 如果通过jdbc.drivers配置了驱动，则在本方法最后进行实例化
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                    public String run() {
                        return System.getProperty("jdbc.drivers");
                    }
                    });
        } catch (Exception ex) {
            drivers = null;
        }
        // If the driver is packaged as a Service Provider, load it.
        // Get all the drivers through the classloader
        // exposed as a java.sql.Driver.class service.
        // ServiceLoader.load() replaces the sun.misc.Providers()
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                // 通过ServiceLoader加载所有通过SPI方式注册的"java.sql.Driver"服务
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                // 遍历ServiceLoader实例进行强制实例化，因此除了遍历不做任何其他操作
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                    // Do nothing
                }
                return null;
                }
            }
        );

        println("DriverManager.initialize: jdbc.drivers = " + drivers);

        // 强制加载"jdbc.driver"环境变量中配置的DB驱动
        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
    ...
}
```

以MySQL驱动为例看看驱动实例化时做了什么：

```java
package com.mysql.jdbc;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            // 向DriverManager注册自己
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

    /**
     * Construct a new driver and register it with DriverManager
     *
     * @throws SQLException
     *             if a database error occurs.
     */
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```

再看看mysql驱动jar包中对service的配置：

![2020-07-07-o4MaOw](https://image.ldbmcs.com/2020-07-07-o4MaOw.jpg)

因此，只要某个驱动以这种方式被引用并被上下文class loader加载，那么该驱动就会通过SPI的方式被自动发现和加载。实际使用时，`Driver.getDriver(url)`会通过DB连接url获取到正确的驱动并建立与DB的连接。

## 5. Spring Boot 中的应用

Spring Boot提供了一种快速的方式来创建可用于生产环境的基于Spring的应用程序。它基于Spring框架，更倾向于约定而不是配置，并且旨在使您尽快启动并运行。

即便没有任何配置文件，SpringBoot的Web应用都能正常运行。这种神奇的事情，SpringBoot正是依靠自动配置来完成。

说到这，我们必须关注一个东西：`SpringFactoriesLoader`，自动配置就是依靠它来加载的。

### 5.1 配置文件

SpringFactoriesLoader来负责加载配置。我们打开这个类，看到它加载文件的路径是：`META-INF/spring.factories`

![2020-07-07-IALK8S](https://image.ldbmcs.com/2020-07-07-IALK8S.jpg)

笔者在项目中搜索这个文件，发现有4个Jar包都包含它：

* `spring-boot-2.1.9.RELEASE.jar`
* `spring-beans-5.1.10.RELEASE.jar`
* `spring-boot-autoconfigure-2.1.9.RELEASE.jar`
* `mybatis-spring-boot-autoconfigure-2.1.0.jar`

那么它们里面都是些啥内容呢？其实就是一个个接口和类的映射。在这里笔者就不贴了，有兴趣的小伙伴自己去看看。

比如在SpringBoot启动的时候，要加载所有的`ApplicationContextInitializer`，那么就可以这样做：

```java
SpringFactoriesLoader.loadFactoryNames(ApplicationContextInitializer.class, classLoader)
```

### 5.2 加载文件

`loadSpringFactories`就负责读取所有的`spring.factories`文件内容。

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {

    MultiValueMap<String, String> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }
    try {
        //获取所有spring.factories文件的路径
        Enumeration<URL> urls = lassLoader.getResources("META-INF/spring.factories");
        result = new LinkedMultiValueMap<>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            //加载文件并解析文件内容
            UrlResource resource = new UrlResource(url);
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                String factoryClassName = ((String) entry.getKey()).trim();
                for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                    result.add(factoryClassName, factoryName.trim());
                }
            }
        }
        cache.put(classLoader, result);
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load factories from location [" +
            FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```

可以看到，它并没有采用JDK中的SPI机制来加载这些类，不过原理差不多。都是通过一个配置文件，加载并解析文件内容，然后通过反射创建实例。

### 5.3 参与其中

假如你希望参与到`SpringBoot`初始化的过程中，现在我们又多了一种方式。

我们也创建一个`spring.factories`文件，自定义一个初始化器。

```java
org.springframework.context.ApplicationContextInitializer=com.youyouxunyin.config.context.MyContextInitializer
```

![2020-07-07-3sVlaj](https://image.ldbmcs.com/2020-07-07-3sVlaj.jpg)

然后定义一个`MyContextInitializer`类：

```java
public class MyContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        System.out.println(configurableApplicationContext);
    }
}
```

## 6. Dubbo中的应用

我们熟悉的Dubbo也不例外，它也是通过 SPI 机制加载所有的组件。同样的，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。

关于原理，如果有小伙伴不熟悉，可以参阅笔者文章：

[Dubbo中的SPI和自适应扩展机制](https://juejin.im/post/5c909949e51d450fae18deb8)

它的使用方式同样是在`META-INF/services`创建文件并写入相关类名。

关于使用场景，可以参考: [SpringBoot+Dubbo集成ELK实战](https://juejin.im/post/5db57e856fb9a020664fc00e)

## 7. 参考

1. [Service Provider Interface详解 \(SPI\)](https://www.jianshu.com/p/27c837293aeb)
2. [SPI机制的原理和应用](https://www.lagou.com/lgeduarticle/67959.html)

