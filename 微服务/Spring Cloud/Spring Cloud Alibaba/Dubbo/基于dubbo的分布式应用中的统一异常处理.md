> 转载：[基于dubbo的分布式应用中的统一异常处理](http://xurui.pro/2018/05/29/%E5%9F%BA%E4%BA%8Edubbo%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E5%BA%94%E7%94%A8%E4%B8%AD%E7%9A%84%E7%BB%9F%E4%B8%80%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86/)

## 1. 背景

> 目前在做的项目使用 `dubbo` 作为分布式服务框架，新项目开发过程中遇到一个问题：`provider` 端抛出了自定义的业务异常，而 `consumer` 接收到的却是 `RpcException`，原来的业务异常（包括异常栈）被包装到了 `message` 中，导致 `consumer` 不能正确获取 `provider` 想要提供的 `message`。

出现这个问题，有几个前提：

- 自定义的异常。
- 自定义的业务异常继承 `RuntimeException`，属于运行期/非检查异常。
- 自定义的业务异常在二方包/三方包中，不在 `provider` 提供的API包中。

## 2. 先说结论

基于上面的几个前提，可以发现**自定义的异常可能会在 `consumer` 接收时反序列化失败**，因为这个异常不一定在 `consumer`中存在。而dubbo为了避免这种情况出现，使用 `ExceptionFilter` 过滤器对异常进行了包装，转换为 `RuntimeException` 提供给 `consumer` 。

最后我利用 `Spring AOP` 拦截掉`provider`所有异常，将异常包装成`Response`(一个自定义的返回值对象POJO)返回给`consumer`，规避掉了 `dubbo` 的处理。

## 3. ExceptionFilter - dubbo 的异常处理源码

```java
@Activate(group = Constants.PROVIDER)
public class ExceptionFilter implements Filter {

    private final Logger logger;

    public ExceptionFilter() {
        this(LoggerFactory.getLogger(ExceptionFilter.class));
    }

    public ExceptionFilter(Logger logger) {
        this.logger = logger;
    }

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        try {
            Result result = invoker.invoke(invocation);
            if (result.hasException() && GenericService.class != invoker.getInterface()) {
                try {
                    Throwable exception = result.getException();

                    // 检查异常，直接抛出
                    if (!(exception instanceof RuntimeException) && (exception instanceof Exception)) {
                        return result;
                    }
                    // 方法签名上有说明抛出非检查异常，直接抛出
                    try {
                        Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
                        Class<?>[] exceptionClassses = method.getExceptionTypes();
                        for (Class<?> exceptionClass : exceptionClassses) {
                            if (exception.getClass().equals(exceptionClass)) {
                                return result;
                            }
                        }
                    } catch (NoSuchMethodException e) {
                        return result;
                    }

                    // for the exception not found in method's signature, print ERROR message in server's log.
                    logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

                    // 异常类和接口类在同一jar包里，直接抛出
                    String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());
                    String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());
                    if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)) {
                        return result;
                    }
                    // JDK异常，直接抛出
                    String className = exception.getClass().getName();
                    if (className.startsWith("java.") || className.startsWith("javax.")) {
                        return result;
                    }
                    // dubbo异常，直接抛出
                    if (exception instanceof RpcException) {
                        return result;
                    }

                    // 否则，包装成RuntimeException抛给客户端
                    return new RpcResult(new RuntimeException(StringUtils.toString(exception)));
                } catch (Throwable e) {
                    logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
                    return result;
                }
            }
            return result;
        } catch (RuntimeException e) {
            logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                    + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                    + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
            throw e;
        }
    }

}
```

重点的话，就在：

```java
// 否则，包装成RuntimeException抛给客户端
return new RpcResult(new RuntimeException(StringUtils.toString(exception)));
```

为了避免反序列化失败，dubbo这样处理也是合理的，但确实和目前项目的应用情况有冲突，要如何解决这个问题？

### 3.1 常规来说有如下方法：

1. 将该异常的包名以`java.`或者`javax.` 开头
   不符合规范，排除

2. 异常继承Exception
   自定义的业务异常本身属于 `RuntimeException`, 排除

3. 异常类和接口类在同一jar包里
   较大的项目一般都会有一些common包，定义好异常类型，使用二方包的方式引用，所以也不适用，排除

4. `provider`的`api`明确写明`throws XxxException`
   作为服务端，不应显式抛出异常给客户的进行处理，排除

5. 实现 `dubbo` 的 `filter`，自定 `provider` 的异常处理逻辑
   可以在原有逻辑基础上增加:

   ```java
   if("自定义异常的包路径".equals(exceptionFile)){
       return result;
   }
   ```
   
   可行，但是逻辑中写死了异常的路径，而且也不能保证后期其他 `consumer`，会引用这个二方包。

## 4. 最终方案

结合项目的实际情况，我们项目中 `provider` 提供的接口返回值全部包装一层再提供出去，这样的好处是服务和服务之间的交互使用的数据对象都是一样的，便于使用，这个对象比较常规：

```java
public class Response<T> implements Serializable {
    private static final long serialVersionUID = -1L;
    private boolean success;
    private T result;
    private String error;
    
    ...
}
```

我觉得Response中增加 `private String code;` 用于标记错误类型，会更好一些
基于这个前提，我想利用 `Spring AOP` 拦截掉`provider`所有异常，将异常包装成`Response`对外提供，规避掉 `dubbo` 的处理。如果是业务异常利用 `error` 提供简单异常信息给外部，如果非业务异常只提示`服务调用失败`，具体错误输出到日志，再通过`kibana`等日志平台查看。

代码：

```java
/**
 * 服务层异常处理器
 * <p>
 * Created by XuRui on 2018/5/28.
 */
@Component
@Aspect
@Slf4j
public class ServiceExceptionHandle {

    /**
     * 指定返回值为Response类型的Service
     */
    @Pointcut(value = "execution(public com.xurui.youth.dto.Response com.xurui.youth.service..*Service*.*(..))")
    private void servicePointcut() {
        // Do nothing just pointcut
    }

    /**
     * 异常处理切面
     * 将异常包装为Response，避免dubbo进行包装
     *
     * @param pjp 处理点
     * @return 返回异常处理结果
     */
    @Around("servicePointcut()")
    public Object doAround(ProceedingJoinPoint pjp) {
        Object[] args = pjp.getArgs();
        try {
            return pjp.proceed();
            // 业务自定义异常
        } catch (ServiceException | JsonResponseException | ServiceResponseException e) {
            processException(pjp, args, e);
            return Response.fail(e.getMessage());
        } catch (Exception e) {
            processException(pjp, args, e);
            return Response.fail("服务调用失败");
        } catch (Throwable throwable) {
            processException(pjp, args, throwable);
            return Response.fail("系统异常");
        }
    }

    /**
     * 处理异常
     *
     * @param joinPoint 切点
     * @param args      参数
     * @param throwable 异常
     */
    private void processException(final ProceedingJoinPoint joinPoint, final Object[] args, Throwable throwable) {
        String inputParam = "";
        if (args != null && args.length > 0) {
            StringBuilder sb = new StringBuilder();
            for (Object arg : args) {
                sb.append(",");
                sb.append(arg);
            }
            inputParam = sb.toString().substring(1);
        }
        log.warn("\n 方法: {}\n 入参: {} \n 错误信息: {}", joinPoint.toLongString(), inputParam, Throwables.getStackTraceAsString(throwable));
    }
}
```

但是这个方式因为拦截掉了service的异常，所以如果多个service存在事务传递的情况，会导致事务失效，因为我们这个项目要求事务单独抽出一层manage来做，所以没有问题。

但是也可以优化：新定义一个切点，声明任何持有@Transactional注解的方法，将这部分需要事务的方法单独处理，最终代码如下：

```java
/**
 * 服务层异常处理器
 * <p>
 * Created by XuRui on 2018/5/28.
 */
@Component
@Aspect
@Slf4j
public class ServiceExceptionHandle {

    /**
     * 返回值类型为Response的Service
     */
    @Pointcut(value = "execution(public com.xurui.youth.dto.Response com.xurui.youth.service..*Service*.*(..))")
    private void servicePointcut() {
    }

    /**
     * 任何持有@Transactional注解的方法
     */
    @Pointcut(value = "@annotation(org.springframework.transaction.annotation.Transactional)")
    private void transactionalPointcut() {
    }

    /**
     * 异常处理切面
     * 将异常包装为Response，避免dubbo进行包装
     *
     * @param pjp 处理点
     * @return Object
     */
    @Around("servicePointcut() && !transactionalPointcut()")
    public Object doAround(ProceedingJoinPoint pjp) {
        Object[] args = pjp.getArgs();
        try {
            return pjp.proceed();
        } catch (ServiceException | JsonResponseException | ServiceResponseException e) { // 业务自定义异常
            processException(pjp, args, e);
            return Response.fail(e.getMessage());
        } catch (Exception e) {
            processException(pjp, args, e);
            return Response.fail("服务调用失败");
        } catch (Throwable throwable) {
            processException(pjp, args, throwable);
            return Response.fail("系统异常");
        }
    }

    /**
     * 任何持有@Transactional注解的方法异常处理切面
     * 将自定义的业务异常转为RuntimeException:
     * 1.规避dubbo的包装，让customer可以正常获取message
     * 2.抛出RuntimeException使事务可以正确回滚
     * 其他异常不处理
     *
     * @param pjp 处理点
     * @return Object
     */
    @Around("servicePointcut() && transactionalPointcut()")
    public Object doTransactionalAround(ProceedingJoinPoint pjp) throws Throwable {
        try {
            return pjp.proceed();
        } catch (ServiceException | JsonResponseException | ServiceResponseException e) {
            // dubbo会将异常捕获进行打印，这里就不打印了
            // processException(pjp, args, e);
            throw new RuntimeException(e.getMessage());
        }
    }

    /**
     * 处理异常
     *
     * @param joinPoint 切点
     * @param args      参数
     * @param throwable 异常
     */
    private void processException(final ProceedingJoinPoint joinPoint, final Object[] args, Throwable throwable) {
        String inputParam = "";
        if (args != null && args.length > 0) {
            StringBuilder sb = new StringBuilder();
            for (Object arg : args) {
                sb.append(",");
                sb.append(arg);
            }
            inputParam = sb.toString().substring(1);
        }
        log.warn("\n 方法: {}\n 入参: {} \n 错误信息: {}", joinPoint.toLongString(), inputParam, Throwables.getStackTraceAsString(throwable));
    }
}
```

