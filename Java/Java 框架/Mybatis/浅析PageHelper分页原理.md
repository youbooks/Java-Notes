> 转载：[浅析pagehelper分页原理](https://blog.csdn.net/qq_21996541/article/details/79796117)

写的第一篇文章，不足之处，请多提宝贵意见，谢谢。

之前项目一直使用的是普元框架，最近公司项目搭建了新框架，主要是由公司的大佬搭建的，以springboot为基础。为了多学习点东西，我也模仿他搭了一套自己的框架，但是在完成分页功能的时候，确遇到了问题。

框架的分页组件使用的是pagehelper，对其我也是早有耳闻，但是也是第一次接触（ps:工作1年，一直使用的是普元封装好的前端框架）。

要是用pagehelper，首先maven项目，要引入：

```xml
<dependency>
   <groupId>com.github.pagehelper</groupId>
   <artifactId>pagehelper</artifactId>
   <version>4.1.6</version>
</dependency>
```

![2020-09-08-XXSeq5](https://image.ldbmcs.com/2020-09-08-XXSeq5.jpg)

前端使用的bootstrap分页插件，这里不再赘述，直接切入正题，持久层框架使用的是mybatis，分页插件的URL，指向后台的controller，

```java
@ResponseBody
@RequestMapping("/testPage")
public String testPage(HttpServletRequest request) {
   Page<PmProduct> page = PageUtil.pageStart(request);
   List<PmProduct> list = prodMapper.selectAll();
   JSONObject rst = PageUtil.pageEnd(request, page, list);
   return rst.toString();
}
```

这是controller的代码，当时就产生一个疑问，为啥在这个pageStart后面查询就能够实现分页呢？可以查出想要的10条分页数据，而去掉则是全部查询出来的84条记录。

带着问题，我开始了我的debug之旅，首先，我跟进pageStart方法，这个PageUtil类是公司大佬封装的一个工具类，代码如下：

```java
public class PageUtil {

    public enum CNT{
        total,
        res
    }

    public static <E> Page<E> pageStart(HttpServletRequest request){
        return pageStart(request,null);
    }

    /**
     *
     * @param request
     * @param pageSize 每页显示条数
     * @param orderBy 写入 需要排序的 字段名 如： product_id desc
     * @return
     */
    public static <E> Page<E> pageStart(HttpServletRequest request,String orderBy){

        int pageon=getPageon(request);
        int pageSize=getpageSize(request);
        Page<E> page=PageHelper.startPage(pageon, pageSize);
        if(!StringUtils.isEmpty(orderBy)){
            PageHelper.orderBy(orderBy);
        }

        return page;
    }

    private static int getPageon(HttpServletRequest request){
        String pageonStr=request.getParameter("pageon");
        int pageon;
        if(StringUtils.isEmpty(pageonStr)){
            pageon=1;
        }else{
            pageon=Integer.parseInt(pageonStr);
        }
        return pageon;
    }

    private static int getpageSize(HttpServletRequest request){
        String pageSizeStr=request.getParameter("pageSize");
        int pageSize;
        if(StringUtils.isEmpty(pageSizeStr)){
            pageSize=1;
        }else{
            pageSize=Integer.parseInt(pageSizeStr);
        }
        return pageSize;
    }

    /**
     *
     * @param request
     * @param page
     * @param list
     * @param elName 页面显示所引用的变量名
     */
    public static JSONObject pageEnd(HttpServletRequest request, Page<?> page,List<?> list){
        JSONObject rstPage=new JSONObject();
        rstPage.put(CNT.total.toString(), page.getTotal());
        rstPage.put(CNT.res.toString(), list);
        return rstPage;
    }
}
```

可以看到，pageStart有两个方法重载，进入方法后，获取了前端页面传递的pageon、pageSize两个参数，分别表示当前页面和每页显示多少条，然后调用了PageHelper.startPage，接着跟进此方法，发现也是一对方法重载，没关系，往下看：

```java
/**
 * 开始分页
 *
 * @param pageNum      页码
 * @param pageSize     每页显示数量
 * @param count        是否进行count查询
 * @param reasonable   分页合理化,null时用默认配置
 * @param pageSizeZero true且pageSize=0时返回全部结果，false时分页,null时用默认配置
 */
public static <E> Page<E> startPage(int pageNum, int pageSize, boolean count, Boolean reasonable, Boolean pageSizeZero) {
    Page<E> page = new Page<E>(pageNum, pageSize, count);
    page.setReasonable(reasonable);
    page.setPageSizeZero(pageSizeZero);
    //当已经执行过orderBy的时候
    Page<E> oldPage = SqlUtil.getLocalPage();
    if (oldPage != null && oldPage.isOrderByOnly()) {
        page.setOrderBy(oldPage.getOrderBy());
    }
    SqlUtil.setLocalPage(page);
    return page;
}
```

上面的方法才是真正分页调用的地方，原来是对传入参数的赋值，赋给Page这个类，继续，发现getLocalPage和setLoaclPage这两个方法，很可疑，跟进，看看他到底做了啥，

```java
public static <T> Page<T> getLocalPage() {
    return LOCAL_PAGE.get();
}

public static void setLocalPage(Page page) {
    LOCAL_PAGE.set(page);
}
private static final ThreadLocal<Page> LOCAL_PAGE = new ThreadLocal<Page>();

```

哦，有点明白了，原来就是赋值保存到本地线程变量里面，这个ThreadLocal是何方神圣，居然有这么厉害，所以查阅了相关博客，链接：https://blog.csdn.net/u013521220/article/details/73604917，个人简单理解大概意思是这个类有4个方法，set,get,remove,initialValue,可以使每个线程独立开来，参数互不影响，里面保存当前线程的变量副本。

OK，那这个地方就是保存了当前分页线程的Page参数的变量。有赋值就有取值，那么在下面的分页过程中，肯定在哪边取到了这个threadLocal的page参数。

好，执行完startPage，下面就是执行了mybatis的SQL语句：

```xml
<select id="selectAll" resultMap="BaseResultMap">
  select SEQ_ID, PRODUCT_ID, PRODUCT_NAME, PRODUCT_DESC, CREATE_TIME, EFFECT_TIME, 
  EXPIRE_TIME, PRODUCT_STATUS, PROVINCE_CODE, REGION_CODE, CHANGE_TIME, OP_OPERATOR_ID, 
  PRODUCT_SYSTEM, PRODUCT_CODE
  from PM_PRODUCT
</select>
```

SQL语句很简单，就是简单的查询出PM_PRODUCT的全部记录，那到底是哪边做了拦截吗？带着这个疑问，我跟进了代码，

发现进入了mybatis的MapperPoxy这个代理类：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  if (Object.class.equals(method.getDeclaringClass())) {
    try {
      return method.invoke(this, args);
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }
  final MapperMethod mapperMethod = cachedMapperMethod(method);
  return mapperMethod.execute(sqlSession, args);
}
```

最后执行的execute方法，再次跟进，进入MapperMethod这个类的execute方法：

```java
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  if (SqlCommandType.INSERT == command.getType()) {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.insert(command.getName(), param));
  } else if (SqlCommandType.UPDATE == command.getType()) {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.update(command.getName(), param));
  } else if (SqlCommandType.DELETE == command.getType()) {
    Object param = method.convertArgsToSqlCommandParam(args);
    result = rowCountResult(sqlSession.delete(command.getName(), param));
  } else if (SqlCommandType.SELECT == command.getType()) {
    if (method.returnsVoid() && method.hasResultHandler()) {
      executeWithResultHandler(sqlSession, args);
      result = null;
    } else if (method.returnsMany()) {
      result = executeForMany(sqlSession, args);
    } else if (method.returnsMap()) {
      result = executeForMap(sqlSession, args);
    } else if (method.returnsCursor()) {
      result = executeForCursor(sqlSession, args);
    } else {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = sqlSession.selectOne(command.getName(), param);
    }
  } else if (SqlCommandType.FLUSH == command.getType()) {
      result = sqlSession.flushStatements();
  } else {
    throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName() 
        + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
```

由于执行的是select操作，并且查询出多条，所以就到了executeForMany这个方法中，后面继续跟进代码SqlSessionTemplate,DefaultSqlSession（不再赘述），最后可以看到代码进入了Plugin这个类的invoke方法中：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    Set<Method> methods = signatureMap.get(method.getDeclaringClass());
    if (methods != null && methods.contains(method)) {
      return interceptor.intercept(new Invocation(target, method, args));
    }
    return method.invoke(target, args);
  } catch (Exception e) {
    throw ExceptionUtil.unwrapThrowable(e);
  }
}
```

这下明白了，interceptor是mybatis的拦截器，而PageHelper这个类就实现了interceptor接口，调用其中的intercept方法：

```java
/**
 * Mybatis拦截器方法
 *
 * @param invocation 拦截器入参
 * @return 返回执行结果
 * @throws Throwable 抛出异常
 */
public Object intercept(Invocation invocation) throws Throwable {
    if (autoRuntimeDialect) {
        SqlUtil sqlUtil = getSqlUtil(invocation);
        return sqlUtil.processPage(invocation);
    } else {
        if (autoDialect) {
            initSqlUtil(invocation);
        }
        return sqlUtil.processPage(invocation);
    }
}
```

```java
/**
 * Mybatis拦截器方法
 *
 * @param invocation 拦截器入参
 * @return 返回执行结果
 * @throws Throwable 抛出异常
 */
private Object _processPage(Invocation invocation) throws Throwable {
    final Object[] args = invocation.getArgs();
    Page page = null;
    //支持方法参数时，会先尝试获取Page
    if (supportMethodsArguments) {
        page = getPage(args);
    }
    //分页信息
    RowBounds rowBounds = (RowBounds) args[2];
    //支持方法参数时，如果page == null就说明没有分页条件，不需要分页查询
    if ((supportMethodsArguments && page == null)
            //当不支持分页参数时，判断LocalPage和RowBounds判断是否需要分页
            || (!supportMethodsArguments && SqlUtil.getLocalPage() == null && rowBounds == RowBounds.DEFAULT)) {
        return invocation.proceed();
    } else {
        //不支持分页参数时，page==null，这里需要获取
        if (!supportMethodsArguments && page == null) {
            page = getPage(args);
        }
        return doProcessPage(invocation, page, args);
    }
}
```

最终我在SqlUtil中的_processPage方法中找到了，getPage这句话，getLocalPage就将保存在ThreadLocal中的Page变量取了出来，这下一切一目了然了。

![2020-09-08-LVKlso](https://image.ldbmcs.com/2020-09-08-LVKlso.jpg)

跟进代码，发现进入了doProcessPage方法，通过反射机制，首先查询出数据总数量，然后进行分页SQL的拼装，MappedStatement的getBoundSql：

```java
public BoundSql getBoundSql(Object parameterObject) {
  BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
  List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
  if (parameterMappings == null || parameterMappings.isEmpty()) {
    boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);
  }

  // check for nested result maps in parameter mappings (issue #30)
  for (ParameterMapping pm : boundSql.getParameterMappings()) {
    String rmId = pm.getResultMapId();
    if (rmId != null) {
      ResultMap rm = configuration.getResultMap(rmId);
      if (rm != null) {
        hasNestedResultMaps |= rm.hasNestedResultMaps();
      }
    }
  }

  return boundSql;
}
```

继续，跟进代码，发现，最终分页的查询，调到了PageStaticSqlSource类的getPageBoundSql中：

```java
protected BoundSql getPageBoundSql(Object parameterObject) {
    String tempSql = sql;
    String orderBy = PageHelper.getOrderBy();
    if (orderBy != null) {
        tempSql = OrderByParser.converToOrderBySql(sql, orderBy);
    }
    tempSql = localParser.get().getPageSql(tempSql);
    return new BoundSql(configuration, tempSql, localParser.get().getPageParameterMapping(configuration, original.getBoundSql(parameterObject)), parameterObject);
}
```

进入getPageSql这个方法，发现，进入了OracleParser类中（还有很多其他的Parser，适用于不同的数据库）。

```java
public String getPageSql(String sql) {
    StringBuilder sqlBuilder = new StringBuilder(sql.length() + 120);
    sqlBuilder.append("select * from ( select tmp_page.*, rownum row_id from ( ");
    sqlBuilder.append(sql);
    sqlBuilder.append(" ) tmp_page where rownum <= ? ) where row_id > ?");
    return sqlBuilder.toString();
}
```

终于，原来分页的SQL是在这里拼装起来的。

总结：PageHelper首先将前端传递的参数保存到page这个对象中，接着将page的副本存放入ThreadLoacl中，这样可以保证分页的时候，参数互不影响，接着利用了mybatis提供的拦截器，取得ThreadLocal的值，重新拼装分页SQL，完成分页。

PS：DEBUG用的好不好，对看框架源码很有很大的影响。

