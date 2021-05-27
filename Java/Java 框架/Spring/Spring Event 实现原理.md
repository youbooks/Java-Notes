# Spring Event 实现原理

> 转载：[Spring Event 实现原理](https://www.jianshu.com/p/8953dd048c2c)

## 1. Demo

### 1.1 事件定义

> 继承 ApplicationEvent 类

```java
import lombok.Getter;
import org.springframework.context.ApplicationEvent;

@Getter
public class EventDemo extends ApplicationEvent {

    private String message;
    
    // Object source 具体事件源类
    public EventDemo(Object source, String message) {
        super(source);
        this.message = message;
    }
}
```

### 1.2 事件监听

> 实现 ApplicationListener 类，并重写 onApplicationEvent() 方法

```java
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class EventDemoListern implements ApplicationListener<EventDemo> {

    @Override
    public void onApplicationEvent(EventDemo event) {
        System.out.println("收到信息: " + event.getMessage());
    }
}
```

### 1.3 事件发布

> ApplicationEventPublisher Bean 发布事件

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Component;

@Component
public class EventDemoPublish {

    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void publish(String message) {
        EventDemo eventDemo = new EventDemo(this, message);
        applicationEventPublisher.publishEvent(eventDemo);
    }
}
```

## 2. 源码分析

### 2.1 事件是如何发送的？

> 从 demo 的代码来看 事件是通过 `ApplicationEventPublisher` 类的 `publishEvent()` 方法发送的；那么通过断点追踪此方法的行为轨迹，我们会发现其实是调用了 `AbstractApplicationContext` 类的 `publishEvent()` 方法，那么请看下面的源码。

```java
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
        Assert.notNull(event, "Event must not be null");

        ApplicationEvent applicationEvent;
        if (event instanceof ApplicationEvent) {
            //判断事件是否是 ApplicationEvent
            applicationEvent = (ApplicationEvent) event;
        }
        else {
            applicationEvent = new PayloadApplicationEvent<>(this, event);
            if (eventType == null) {
                eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();
            }
        }

        // Multicast right now if possible - or lazily once the multicaster is initialized
        if (this.earlyApplicationEvents != null) {
            this.earlyApplicationEvents.add(applicationEvent);
        }
        else {
            //发送事件的入口
            getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
        }

        // 有父类也发送事件
        if (this.parent != null) {
            if (this.parent instanceof AbstractApplicationContext) {
                ((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
            }
            else {
                this.parent.publishEvent(event);
            }
        }
    }
```

> 既然找到了发送事件的入口，那么就先追踪 `applicationEventMulticaster()` 方法

```java
    /** Helper class used in event publishing. */
    @Nullable
    private ApplicationEventMulticaster applicationEventMulticaster;

    ApplicationEventMulticaster getApplicationEventMulticaster() throws IllegalStateException {
        if (this.applicationEventMulticaster == null) {
            throw new IllegalStateException("ApplicationEventMulticaster not initialized - " +
                    "call 'refresh' before multicasting events via the context: " + this);
        }
        return this.applicationEventMulticaster;
    }
```

到这里我们只掌握了 `ApplicationEventMulticaster` 是个接口类，并不知道真正的实现类，而且也没发现关于此接口的构造函数(稍后再揭秘实现类究竟是怎么来的)；但是通过断点追踪 `multicastEvent()` 方法，不难发现其实是调用了 `SimpleApplicationEventMulticaster` 类的 `multicastEvent()`，源码如下：

```java
    public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
        Executor executor = getTaskExecutor();
// getApplicationListeners() 获取 event 对应的监听者
        for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
            if (executor != null) {
                executor.execute(() -> invokeListener(listener, event));
            }
            else {
// 调用实例
                invokeListener(listener, event);
            }
        }
    }
```

1. 走到这里我们会发现事件的监听者都来自 `getApplicationListeners(event, type)；`
2. 通知监听者的其实是 `invokeListener(listener, event)；`

### 2.2 揭秘如何获取事件对应的监听者

```java
    protected Collection<ApplicationListener<?>> getApplicationListeners(
            ApplicationEvent event, ResolvableType eventType) {

        Object source = event.getSource();
        Class<?> sourceType = (source != null ? source.getClass() : null);
        ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

        // Quick check for existing entry on ConcurrentHashMap...
        // 内部维护的缓存 (ConcurrentHashMap) 方便下次获取对应的监听者
        ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
        if (retriever != null) {
            return retriever.getApplicationListeners();
        }

        if (this.beanClassLoader == null ||
                (ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
                        (sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
            // Fully synchronized building and caching of a ListenerRetriever
            synchronized (this.retrievalMutex) {
                retriever = this.retrieverCache.get(cacheKey);
                if (retriever != null) {
                    return retriever.getApplicationListeners();
                }
                retriever = new ListenerRetriever(true);
                //第一次获取事件对应的监听者并放置缓存
                Collection<ApplicationListener<?>> listeners =
                        retrieveApplicationListeners(eventType, sourceType, retriever);
                this.retrieverCache.put(cacheKey, retriever);
                return listeners;
            }
        }
        else {
            // No ListenerRetriever caching -> no synchronization necessary
            return retrieveApplicationListeners(eventType, sourceType, null);
        }
    }

    // 只截取了核心代码
    private Collection<ApplicationListener<?>> retrieveApplicationListeners(
            ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable ListenerRetriever retriever) {

        List<ApplicationListener<?>> allListeners = new ArrayList<>();
        Set<ApplicationListener<?>> listeners;
        Set<String> listenerBeans;
        synchronized (this.retrievalMutex) {
  // 获取所有 ApplicationListener 的实现类
 // 注： listeners 和 listenerBeans 均是取内部类 ListenerRetriever 维护的 LinkedHashSet
            listeners = new LinkedHashSet<>(this.defaultRetriever.applicationListeners);
            listenerBeans = new LinkedHashSet<>(this.defaultRetriever.applicationListenerBeans);
        }
        for (ApplicationListener<?> listener : listeners) {
  // 判断 listener 的 event 是否一致
            if (supportsEvent(listener, eventType, sourceType)) {
                if (retriever != null) {
                    retriever.applicationListeners.add(listener);
                }
                allListeners.add(listener);
            }
        }
        ...
        return allListeners;
    }

// 内部类
    private class ListenerRetriever {

        public final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();

        public final Set<String> applicationListenerBeans = new LinkedHashSet<>();

        private final boolean preFiltered;

        public ListenerRetriever(boolean preFiltered) {
            this.preFiltered = preFiltered;
        }

        public Collection<ApplicationListener<?>> getApplicationListeners() {
            ...
        }
    }
```

这里同样也会产生一个问题。我们知道 listeners 取自内部类 ListenerRetriever 维护的 applicationListeners 对象，那么 applicationListeners 何时 add 了呢？稍后再揭秘。

> 总结
> 获取所有 ApplicationListener 的实现类后，依次判断是否支持给定的事件，也就是事件的类型是否一致；最终通过特定的 cachKey 放置缓存。

### 2.3 揭秘如何通知事件的监听者

```java
protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
        ErrorHandler errorHandler = getErrorHandler();
        if (errorHandler != null) {
            try {
                doInvokeListener(listener, event);
            }
            catch (Throwable err) {
                errorHandler.handleError(err);
            }
        }
        else {
            doInvokeListener(listener, event);
        }
    }

    @SuppressWarnings({"rawtypes", "unchecked"})
    private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
        try {
// 直接调用 onApplicationEvent
            listener.onApplicationEvent(event);
        }
        catch (ClassCastException ex) {
            ...
        }
    }
```

核心代码：`listener.onApplicationEvent(event);`
拿到监听者的实例，直接调用 `onApplicationEvent()` 方法

为了揭秘上面两个问题，我们继续观察 `AbstractApplicationContext` 类，这个类里有我们想要的答案；

```java
    @Override
    public void addApplicationListener(ApplicationListener<?> listener) {
        Assert.notNull(listener, "ApplicationListener must not be null");
        if (this.applicationEventMulticaster != null) {
            this.applicationEventMulticaster.addApplicationListener(listener);
        }
        this.applicationListeners.add(listener);
    }

    /**
     * Return the list of statically specified ApplicationListeners.
     */
    public Collection<ApplicationListener<?>> getApplicationListeners() {
        return this.applicationListeners;
    }

    @Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            ...
            try {
                ...

                // Initialize event multicaster for this context.
// 初始化 ApplicationEventMulticaster
// Initialize the ApplicationEventMulticaster.
// Uses SimpleApplicationEventMulticaster if none defined in the context.
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                onRefresh();

                // Check for listener beans and register them.
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons.
// 自己实现的 ApplicationListener 从这里实例化，经过复杂的 bean 实例过程，
// 最终由 ApplicationListenerDetector 类调用 addApplicationListener();
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                finishRefresh();
            }
            ...
    }
```

也就是说在容器启动过程中会装载ApplicationEventMulticaster，以及存放applicationListeners到容器中。

