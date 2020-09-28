> 转载：[自定义 logback 日志过滤器](https://www.jianshu.com/p/a0c66ec8a5d0)

Logback 提供两种类型的过滤器，常规过滤器和turbo过滤器。本例讲述基于常规过滤器的自定义实现。

## 1. 常规过滤器

常规的logback-classic过滤器扩展了 `Filter` 抽象类，它基本上由一个`decide()`以`ILoggingEvent` 实例作为参数的方法组成 。

过滤器基于三元逻辑。`decide(ILoggingEvent event)`按顺序调用每个过滤器的方法。此方法返回的一个 `FilterReply` 枚举值，即`DENY`、 `NEUTRAL`或`ACCEPT`。

| 枚举值  | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| DENY    | 立即删除日志事件而不咨询剩余的过滤器                         |
| NEUTRAL | 查询列表中的下一个过滤器，如果没有其他过滤器可供参考，则会正常处理日志记录事件 |
| ACCEPT  | 立即处理日志事件，跳过其余过滤器的调用                       |

在 logback 中，可以将过滤器添加到 `Appender` 实例中。通过向 appender 添加一个或多个过滤器，可以按任意条件过滤事件。

## 2. 实现自己的过滤器

创建自己的过滤器很简单。所要做的就是扩展`Filter`抽象类并实现该 `decide()`方法。

下面显示的 SampleFilter 类提供了一个示例。其 `decide`方法返回 ACCEPT 以记录其消息字段中包含字符串 “sample” 的事件。对于其他事件，返回值 NEUTRAL。

```java
package chapters.filters;

import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class SampleFilter extends Filter<ILoggingEvent> {

  @Override
  public FilterReply decide(ILoggingEvent event) {    
    if (event.getMessage().contains("sample")) {
      return FilterReply.ACCEPT;
    } else {
      return FilterReply.NEUTRAL;
    }
  }
}
```

接下来显示的配置文件将过滤器 SampleFilter 附加到 ConsoleAppender。

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="chapters.filters.SampleFilter" />
    <encoder>
      <pattern>
        %-4relative [%thread] %-5level %logger - %msg%n
      </pattern>
    </encoder>
  </appender>
  <root>
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

如果想设置过滤器的属性或子组件也很容易。在过滤器类中添加相应的setter方法后，在属性命名的xml元素中指定属性的值，并将其嵌套在`<filter>` 元素中。

如将 SampleFilter 中的用于判断的字符串 "sample" 字符提取到配置文件中：

- SampleFilter 增加属性 `flag`，并提供 set 方法。

  ```java
  package chapters.filters;
  
  import ch.qos.logback.classic.spi.ILoggingEvent;
  import ch.qos.logback.core.filter.Filter;
  import ch.qos.logback.core.spi.FilterReply;
  
  public class SampleFilter extends Filter<ILoggingEvent> {
  
    private String flag;
  
    @Override
    public FilterReply decide(ILoggingEvent event) {    
      if (event.getMessage().contains(flag)) {
        return FilterReply.ACCEPT;
      } else {
        return FilterReply.NEUTRAL;
      }
    }
  
    public void setFlag(String flag) {
      this.flag = flag;
    }
  }
  ```

- 在配置文件中指定 `flag` 的值。

  ```xml
  <configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <filter class="chapters.filters.SampleFilter" >
        <flag>sample</flag>
      </filter>
      <encoder>
        <pattern>
          %-4relative [%thread] %-5level %logger - %msg%n
        </pattern>
      </encoder>
    </appender>
    <root>
      <appender-ref ref="STDOUT" />
    </root>
  </configuration>
  ```

## 3. 链接

[官方文档 - Filters](https://logback.qos.ch/manual/filters.html)