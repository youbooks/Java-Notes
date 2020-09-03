# Spring Security

[TOC]

## 1. 核心组件

### 1.1 SecurityContextHolder

SecurityContextHolder用于存储安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限…这些都被保存在SecurityContextHolder中。SecurityContextHolder默认使用ThreadLocal 策略来存储认证信息。看到ThreadLocal 也就意味着，这是一种与线程绑定的策略。

```java
// getAuthentication()返回了认证信息，再次getPrincipal()返回了身份信息，UserDetails便是Spring对身份信息封装的一个接口。
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}
```

### 1.2 Authentication

```java
public interface Authentication extends Principal, Serializable {

	// getAuthorities()，权限信息列表，默认是GrantedAuthority接口的一些实现类，通常是代表权限信息的一系列字符串。
	Collection<? extends GrantedAuthority> getAuthorities();

	// getCredentials()，密码信息，用户输入的密码字符串，在认证过后通常会被移除，用于保障安全。
	Object getCredentials();// <2>

	// getDetails()，细节信息，web应用中的实现接口通常为 WebAuthenticationDetails，它记录了访问者的ip地址和sessionId的值。
	Object getDetails();// <2>

	// getPrincipal()，敲黑板！！！最重要的身份信息，大部分情况下返回的是UserDetails接口的实现类，也是框架中的常用接口之一。UserDetails接口将会在下面的小节重点介绍。
	Object getPrincipal();// <2>

	boolean isAuthenticated();// <2>

	void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```

1. `Authentication`是spring security包中的接口，直接继承自Principal类，而Principal是位于java.security包中的。可以见得，Authentication在spring security中是最高级别的身份/认证的抽象。

2. 由这个顶级接口，我们可以得到用户拥有的权限信息列表，密码，用户细节信息，用户身份信息，认证信息。

**Spring Security是如何完成身份认证的?**

1. 用户名和密码被过滤器获取到，封装成`Authentication`,通常情况下是通过`UsernamePasswordAuthenticationToken`这个实现类。
```java
Authentication request = new UsernamePasswordAuthenticationToken(name, password);
```

2. `AuthenticationManager` 身份管理器负责验证这个`Authentication`。
```java
private static AuthenticationManager am = new SampleAuthenticationManager();
Authentication result = am.authenticate(request);
```

3. 认证成功后，`AuthenticationManager`身份管理器返回一个被填充满了信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除）`Authentication`实例。
```java
Authentication result = am.authenticate(request);
```

4. `SecurityContextHolder`安全上下文容器将第3步填充了信息的`Authentication`，通过`SecurityContextHolder.getContext().setAuthentication(…)`方法，设置到其中。
```java
SecurityContextHolder.getContext().setAuthentication(result);
```

### 1.3 AuthenticationManager

初次接触Spring Security的朋友相信会被AuthenticationManager，ProviderManager ，AuthenticationProvider …这么多相似的Spring认证类搞得晕头转向，但只要稍微梳理一下就可以理解清楚它们的联系和设计者的用意。

AuthenticationManager（接口）是认证相关的核心接口，也是发起认证的出发点，因为在实际需求中，我们可能会允许用户使用用户名+密码登录，同时允许用户使用邮箱+密码，手机号码+密码登录，甚至，可能允许用户使用指纹登录（还有这样的操作？没想到吧），所以说AuthenticationManager一般不直接认证，**AuthenticationManager接口的常用实现类ProviderManager 内部会维护一个List<AuthenticationProvider>列表，存放多种认证方式，实际上这是委托者模式的应用（Delegate）**。

也就是说，核心的认证入口始终只有一个：**AuthenticationManager**，不同的认证方式：用户名+密码（UsernamePasswordAuthenticationToken），邮箱+密码，手机号码+密码登录则对应了三个**AuthenticationProvider**。这样一来四不四就好理解多了？熟悉shiro的朋友可以把AuthenticationProvider理解成Realm。在默认策略下，只需要通过一个AuthenticationProvider的认证，即可被认为是登录成功。

ProviderManager 中的List，会依照次序去认证，认证成功则立即返回，若认证失败则返回null，下一个AuthenticationProvider会继续尝试认证，如果所有认证器都无法认证成功，则ProviderManager 会抛出一个ProviderNotFoundException异常。

### 1.4 DaoAuthenticationProvider

**AuthenticationProvider最最最常用的一个实现便是DaoAuthenticationProvider。**

在Spring Security中。提交的用户名和密码，被封装成了UsernamePasswordAuthenticationToken，而根据用户名加载用户的任务则是交给了UserDetailsService，在DaoAuthenticationProvider中，对应的方法便是retrieveUser，虽然有两个参数，但是retrieveUser只有第一个参数起主要作用，返回一个UserDetails。还需要完成UsernamePasswordAuthenticationToken和UserDetails密码的比对，这便是交给additionalAuthenticationChecks方法完成的，如果这个void方法没有抛异常，则认为比对成功。比对密码的过程，用到了PasswordEncoder和SaltSource，密码加密和盐的概念相信不用我赘述了，它们为保障安全而设计，都是比较基础的概念。

![](https://image.ldbmcs.com/2020-04-08-054216.jpg)

### 1.5 UserDetails与UserDetailsService

上面不断提到了UserDetails这个接口，它代表了最详细的用户信息，这个接口涵盖了一些必要的用户信息字段，具体的实现类对它进行了扩展。

```java
public interface UserDetails extends Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();

	String getPassword();

	String getUsername();

	boolean isAccountNonExpired();

	boolean isAccountNonLocked();

	boolean isCredentialsNonExpired();

	boolean isEnabled();
}
```

它和Authentication接口很类似，比如它们都拥有username，authorities，区分他们也是本文的重点内容之一。

**Authentication的getCredentials()与UserDetails中的getPassword()需要被区分对待，前者是用户提交的密码凭证，后者是用户正确的密码，认证器其实就是对这两者的比对。Authentication中的getAuthorities()实际是由UserDetails的getAuthorities()传递而形成的。还记得Authentication接口中的getUserDetails()方法吗？其中的UserDetails用户详细信息便是经过了AuthenticationProvider之后被填充的。**

```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

UserDetailsService和AuthenticationProvider两者的职责常常被人们搞混，关于他们的问题在文档的FAQ和issues中屡见不鲜。记住一点即可，敲黑板！！！**UserDetailsService只负责从特定的地方（通常是数据库）加载用户信息，仅此而已，记住这一点，可以避免走很多弯路。UserDetailsService常见的实现类有JdbcDaoImpl，InMemoryUserDetailsManager，前者从数据库加载用户，后者从内存中加载用户，也可以自己实现UserDetailsService，通常这更加灵活**。

![](https://image.ldbmcs.com/2020-04-08-020120.jpg)

## 2. 入门

1. maven依赖

	```xml
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
	```
2. 	配置Spring Security

	```java
	@Configuration
	@EnableWebSecurity <1>
	public class WebSecurityConfig extends WebSecurityConfigurerAdapter { <1>
		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http <2>
				.authorizeRequests()
					.antMatchers("/", "/home").permitAll()
					.anyRequest().authenticated()
					.and()
				.formLogin()
					.loginPage("/login")
					.permitAll()
					.and()
				.logout()
					.permitAll();
		}

		@Autowired
		public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
			auth <3>
				.inMemoryAuthentication()
					.withUser("admin").password("admin").roles("USER");
		}
	}
	```
	1. @EnableWebSecurity注解使得SpringMVC集成了Spring Security的web安全支持。另外，WebSecurityConfig配置类同时集成了WebSecurityConfigurerAdapter，重写了其中的特定方法，用于自定义Spring Security配置。整个Spring Security的工作量，其实都是集中在该配置类，不仅仅是这个guides，实际项目中也是如此。

	2. configure(HttpSecurity)定义了哪些URL路径应该被拦截，如字面意思所描述：”/“, “/home”允许所有人访问，”/login”作为登录入口，也被允许访问，而剩下的”/hello”则需要登陆后才可以访问。

	3. configureGlobal(AuthenticationManagerBuilder)在内存中配置一个用户，admin/admin分别是用户名和密码，这个用户拥有USER角色。

## 3. 核心配置解读

### 3.1 功能介绍

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.antMatchers("/", "/home").permitAll()
			.anyRequest().authenticated()
			.and()
		.formLogin()
			.loginPage("/login")
			.permitAll()
			.and()
		.logout()
			.permitAll();
}

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	auth
		.inMemoryAuthentication()
			.withUser("admin").password("admin").roles("USER");
}
}
```
当配置了上述的javaconfig之后，我们的应用便具备了如下的功能：

1. 除了“/”,”/home”(首页),”/login”(登录),”/logout”(注销),之外，其他路径都需要认证。
2. 指定“/login”该路径为登录页面，当未认证的用户尝试访问任何受保护的资源时，都会跳转到“/login”。
3. 默认指定“/logout”为注销页面
4. 配置一个内存中的用户认证器，使用admin/admin作为用户名和密码，具有USER角色

### 3.2 @EnableWebSecurity

与继承了WebSecurityConfigurerAdapter相比，毫无疑问@EnableWebSecurity起到决定性的配置作用，它其实是个组合注解。

```java
@Import({ WebSecurityConfiguration.class, // <2>
      SpringWebMvcImportSelector.class }) // <1>
@EnableGlobalAuthentication // <3>
@Configuration
public @interface EnableWebSecurity {
   boolean debug() default false;
}
```
@Import是springboot提供的用于引入外部的配置的注解，可以理解为：@EnableWebSecurity注解激活了@Import注解中包含的配置类。

1. SpringWebMvcImportSelector的作用是判断当前的环境是否包含springmvc，因为spring security可以在非spring环境下使用，为了避免DispatcherServlet的重复配置，所以使用了这个注解来区分。
2. WebSecurityConfiguration顾名思义，是用来配置web安全的，下面的小节会详细介绍。
3. @EnableGlobalAuthentication注解的源码如下：

	```java
	@Import(AuthenticationConfiguration.class)
	@Configuration
	public @interface EnableGlobalAuthentication {
	}
	```
	注意点同样在@Import之中，它实际上激活了AuthenticationConfiguration这样的一个配置类，用来配置认证相关的核心类。

	也就是说：@EnableWebSecurity完成的工作便是加载了**WebSecurityConfiguration，AuthenticationConfiguration**这两个核心配置类，也就此将spring security的职责划分为了配置安全信息，配置认证信息两部分。

**WebSecurityConfiguration**

```java
@Configuration
public class WebSecurityConfiguration {
	//DEFAULT_FILTER_NAME = "springSecurityFilterChain"
	@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
    public Filter springSecurityFilterChain() throws Exception {
    	...
    }
 }
```
在未使用springboot之前，大多数人都应该对“springSecurityFilterChain”这个名词不会陌生，**springSecurityFilterChain是spring security的核心过滤器，是整个认证的入口**。在曾经的XML配置中，想要启用spring security，需要在web.xml中进行如下配置：

```xml
<!-- Spring Security -->
   <filter>
       <filter-name>springSecurityFilterChain</filter-name>
       <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
   </filter>

   <filter-mapping>
       <filter-name>springSecurityFilterChain</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
```
而在springboot集成之后，这样的XML被java配置取代。**WebSecurityConfiguration中完成了声明springSecurityFilterChain的作用，并且最终交给DelegatingFilterProxy这个代理类，负责拦截请求**（注意DelegatingFilterProxy这个类不是spring security包中的，而是存在于web包中，spring使用了代理模式来实现安全过滤的解耦）。

**AuthenticationConfiguration**

```java
@Configuration
@Import(ObjectPostProcessorConfiguration.class)
public class AuthenticationConfiguration {

  	@Bean
	public AuthenticationManagerBuilder authenticationManagerBuilder(
			ObjectPostProcessor<Object> objectPostProcessor) {
		return new AuthenticationManagerBuilder(objectPostProcessor);
	}
  
  	public AuthenticationManager getAuthenticationManager() throws Exception {
    	...
    }

}
```
AuthenticationConfiguration的主要任务，便是负责生成全局的身份认证管理者AuthenticationManager，AuthenticationManager便是最核心的身份认证管理器。

### 3.3 WebSecurityConfigurerAdapter

适配器模式在spring中被广泛的使用，在配置中使用Adapter的好处便是，我们可以选择性的配置想要修改的那一部分配置，而不用覆盖其他不相关的配置。WebSecurityConfigurerAdapter中我们可以选择自己想要修改的内容，来进行重写。

#### 3.3.1 HttpSecurity常用配置

```java
@Configuration
@EnableWebSecurity
public class CustomWebSecurityConfig extends WebSecurityConfigurerAdapter {
  
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/resources/**", "/signup", "/about").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .usernameParameter("username")
                .passwordParameter("password")
                .failureForwardUrl("/login?error")
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/index")
                .permitAll()
                .and()
            .httpBasic()
                .disable();
    }
}
```
上述是一个使用Java Configuration配置HttpSecurity的典型配置，其中http作为根开始配置，每一个and()对应了一个模块的配置（等同于xml配置中的结束标签），并且and()返回了HttpSecurity本身，于是可以连续进行配置。他们配置的含义也非常容易通过变量本身来推测，

1. authorizeRequests()配置路径拦截，表明路径访问所对应的权限，角色，认证信息。
2. formLogin()对应表单认证相关的配置
3. logout()对应了注销相关的配置
4. httpBasic()可以配置basic登录

#### 3.3.2 WebSecurityBuilder

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(WebSecurity web) throws Exception {
        web
            .ignoring()
            .antMatchers("/resources/**");
    }
}
```
这个配置中并不会出现太多的配置信息。

#### 3.3.3 AuthenticationManagerBuilder

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
            .withUser("admin").password("admin").roles("USER");
    }
}
```
想要在WebSecurityConfigurerAdapter中进行认证相关的配置，可以使用configure(AuthenticationManagerBuilder auth)暴露一个AuthenticationManager的建造器：`AuthenticationManagerBuilder` 。如上所示，我们便完成了内存中用户的配置。

在前面的文章中我们配置内存中的用户时，似乎不是这么配置的，而是：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("admin").password("admin").roles("USER");
    }
}
```
如果你的应用只有唯一一个WebSecurityConfigurerAdapter，那么他们之间的差距可以被忽略，从方法名可以看出两者的区别：**使用@Autowired注入的AuthenticationManagerBuilder是全局的身份认证器，作用域可以跨越多个WebSecurityConfigurerAdapter，以及影响到基于Method的安全控制；而 @Override的方式则类似于一个匿名内部类，它的作用域局限于一个WebSecurityConfigurerAdapter内部。**

关于这一点的区别，可以参考我曾经提出的issuespring-security#issues4571。官方文档中，也给出了配置多个WebSecurityConfigurerAdapter的场景以及demo，将在该系列的后续文章中解读。

## 4. 核心过滤器源码分析

前面的部分，我们关注了Spring Security是如何完成认证工作的，但是另外一部分核心的内容：**过滤器**，一直没有提到，我们已经知道Spring Security使用了`springSecurityFillterChian`作为了安全过滤的入口，这一节主要分析一下这个过滤器链都包含了哪些关键的过滤器，并且各自的使命是什么。

### 4.1 核心过滤器概述

在之前入门指南中我们配置了一个表单登录的demo，以此为例，来看看这过程中Spring Security都帮我们自动配置了哪些过滤器。

```java
Creating filter chain: o.s.s.web.util.matcher.AnyRequestMatcher@1, 
[o.s.s.web.context.SecurityContextPersistenceFilter@8851ce1, 
o.s.s.web.header.HeaderWriterFilter@6a472566, o.s.s.web.csrf.CsrfFilter@61cd1c71, 
o.s.s.web.authentication.logout.LogoutFilter@5e1d03d7, 
o.s.s.web.authentication.UsernamePasswordAuthenticationFilter@122d6c22, 
o.s.s.web.savedrequest.RequestCacheAwareFilter@5ef6fd7f, 
o.s.s.web.servletapi.SecurityContextHolderAwareRequestFilter@4beaf6bd, 
o.s.s.web.authentication.AnonymousAuthenticationFilter@6edcad64, 
o.s.s.web.session.SessionManagementFilter@5e65afb6, 
o.s.s.web.access.ExceptionTranslationFilter@5b9396d3, 
o.s.s.web.access.intercept.FilterSecurityInterceptor@3c5dbdf8
]
```
上述的log信息是我从springboot启动的日志中CV所得，spring security的过滤器日志有一个特点：log打印顺序与实际配置顺序符合，也就意味着SecurityContextPersistenceFilter是整个过滤器链的第一个过滤器，而FilterSecurityInterceptor则是末置的过滤器。另外通过观察过滤器的名称，和所在的包名，可以大致地分析出他们各自的作用，如UsernamePasswordAuthenticationFilter明显便是与使用用户名和密码登录相关的过滤器，而FilterSecurityInterceptor我们似乎看不出它的作用，但是其位于web.access包下，大致可以分析出他与访问限制相关。第四篇文章主要就是介绍这些常用的过滤器，对其中关键的过滤器进行一些源码分析。先大致介绍下每个过滤器的作用：

1. **SecurityContextPersistenceFilter** 两个主要职责：请求来临时，创建SecurityContext安全上下文信息，请求结束时清空SecurityContextHolder。

2. HeaderWriterFilter (文档中并未介绍，非核心过滤器) 用来给http响应添加一些Header,比如X-Frame-Options, X-XSS-Protection*，X-Content-Type-Options.

3. CsrfFilter 在spring4这个版本中被默认开启的一个过滤器，用于防止csrf攻击，了解前后端分离的人一定不会对这个攻击方式感到陌生，前后端使用json交互需要注意的一个问题。

4. LogoutFilter 顾名思义，处理注销的过滤器

5. **UsernamePasswordAuthenticationFilter** 这个会重点分析，表单提交了username和password，被封装成token进行一系列的认证，便是主要通过这个过滤器完成的，在表单认证的方法中，这是最最关键的过滤器。

5. RequestCacheAwareFilter (文档中并未介绍，非核心过滤器) 内部维护了一个RequestCache，用于缓存request请求。

6. SecurityContextHolderAwareRequestFilter 此过滤器对ServletRequest进行了一次包装，使得request具有更加丰富的API。

7. **AnonymousAuthenticationFilter** 匿名身份过滤器，这个过滤器个人认为很重要，需要将它与UsernamePasswordAuthenticationFilter 放在一起比较理解，spring security为了兼容未登录的访问，也走了一套认证流程，只不过是一个匿名的身份。

8. SessionManagementFilter 和session相关的过滤器，内部维护了一个SessionAuthenticationStrategy，两者组合使用，常用来防止session-fixation protection attack，以及限制同一用户开启多个会话的数量。

9. **ExceptionTranslationFilter** 直译成异常翻译过滤器，还是比较形象的，这个过滤器本身不处理异常，而是将认证过程中出现的异常交给内部维护的一些类去处理，具体是那些类下面详细介绍。

10. **FilterSecurityInterceptor** 这个过滤器决定了访问特定路径应该具备的权限，访问的用户的角色，权限是什么？访问的路径需要什么样的角色和权限？这些判断和处理都是由该类进行的。

其中加粗的过滤器可以被认为是Spring Security的核心过滤器，将在下面，一个过滤器对应一个小节来讲解。

### 4.2 SecurityContextPersistenceFilter

试想一下，如果我们不使用Spring Security，如果保存用户信息呢，大多数情况下会考虑使用Session对吧？在Spring Security中也是如此，用户在登录过一次之后，后续的访问便是通过sessionId来识别，从而认为用户已经被认证。具体在何处存放用户信息，便是第一篇文章中提到的**SecurityContextHolder**；认证相关的信息是如何被存放到其中的，便是通过**SecurityContextPersistenceFilter**。在4.1概述中也提到了，SecurityContextPersistenceFilter的两个主要作用便是请求来临时，创建SecurityContext安全上下文信息和请求结束时清空SecurityContextHolder。

顺带提一下：微服务的一个设计理念需要实现服务通信的无状态，而http协议中的无状态意味着不允许存在session，这可以通过setAllowSessionCreation(false) 实现，这并不意味着SecurityContextPersistenceFilter变得无用，因为它还需要负责清除用户信息。在Spring Security中，虽然安全上下文信息被存储于Session中，但我们在实际使用中不应该直接操作Session，而应当使用SecurityContextHolder。

```java
// org.springframework.security.web.context.SecurityContextPersistenceFilter
public class SecurityContextPersistenceFilter extends GenericFilterBean {

   static final String FILTER_APPLIED = "__spring_security_scpf_applied";
   //安全上下文存储的仓库
   private SecurityContextRepository repo;
  
   public SecurityContextPersistenceFilter() {
      //HttpSessionSecurityContextRepository是SecurityContextRepository接口的一个实现类
      //使用HttpSession来存储SecurityContext
      this(new HttpSessionSecurityContextRepository());
   }

   public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
         throws IOException, ServletException {
      HttpServletRequest request = (HttpServletRequest) req;
      HttpServletResponse response = (HttpServletResponse) res;

      if (request.getAttribute(FILTER_APPLIED) != null) {
         // ensure that filter is only applied once per request
         chain.doFilter(request, response);
         return;
      }
      request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
      //包装request，response
      HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request,
            response);
      //从Session中获取安全上下文信息
      SecurityContext contextBeforeChainExecution = repo.loadContext(holder);
      try {
         //请求开始时，设置安全上下文信息，这样就避免了用户直接从Session中获取安全上下文信息
         SecurityContextHolder.setContext(contextBeforeChainExecution);
         chain.doFilter(holder.getRequest(), holder.getResponse());
      }
      finally {
         //请求结束后，清空安全上下文信息
         SecurityContext contextAfterChainExecution = SecurityContextHolder
               .getContext();
         SecurityContextHolder.clearContext();
         repo.saveContext(contextAfterChainExecution, holder.getRequest(),
               holder.getResponse());
         request.removeAttribute(FILTER_APPLIED);
         if (debug) {
            logger.debug("SecurityContextHolder now cleared, as request processing completed");
         }
      }
   }

}
```
过滤器一般负责核心的处理流程，而具体的业务实现，通常交给其中聚合的其他实体类，这在Filter的设计中很常见，同时也符合职责分离模式。例如存储安全上下文和读取安全上下文的工作完全委托给了HttpSessionSecurityContextRepository去处理，而这个类中也有几个方法可以稍微解读下，方便我们理解内部的工作流程。
```java
//org.springframework.security.web.context.HttpSessionSecurityContextRepository
public class HttpSessionSecurityContextRepository implements SecurityContextRepository {
   // 'SPRING_SECURITY_CONTEXT'是安全上下文默认存储在Session中的键值
   public static final String SPRING_SECURITY_CONTEXT_KEY = "SPRING_SECURITY_CONTEXT";
   ...
   private final Object contextObject = SecurityContextHolder.createEmptyContext();
   private boolean allowSessionCreation = true;
   private boolean disableUrlRewriting = false;
   private String springSecurityContextKey = SPRING_SECURITY_CONTEXT_KEY;

   private AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();

   //从当前request中取出安全上下文，如果session为空，则会返回一个新的安全上下文
   public SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder) {
      HttpServletRequest request = requestResponseHolder.getRequest();
      HttpServletResponse response = requestResponseHolder.getResponse();
      HttpSession httpSession = request.getSession(false);
      SecurityContext context = readSecurityContextFromSession(httpSession);
      if (context == null) {
         context = generateNewContext();
      }
      ...
      return context;
   }

   ...

   public boolean containsContext(HttpServletRequest request) {
      HttpSession session = request.getSession(false);
      if (session == null) {
         return false;
      }
      return session.getAttribute(springSecurityContextKey) != null;
   }

   private SecurityContext readSecurityContextFromSession(HttpSession httpSession) {
      if (httpSession == null) {
         return null;
      }
      ...
      // Session存在的情况下，尝试获取其中的SecurityContext
      Object contextFromSession = httpSession.getAttribute(springSecurityContextKey);
      if (contextFromSession == null) {
         return null;
      }
      ...
      return (SecurityContext) contextFromSession;
   }

   //初次请求时创建一个新的SecurityContext实例
   protected SecurityContext generateNewContext() {
      return SecurityContextHolder.createEmptyContext();
   }

}
```
SecurityContextPersistenceFilter和HttpSessionSecurityContextRepository配合使用，构成了Spring Security整个调用链路的入口，为什么将它放在最开始的地方也是显而易见的，后续的过滤器中大概率会依赖Session信息和安全上下文信息。

### 4.3 UsernamePasswordAuthenticationFilter

表单认证是最常用的一个认证方式，一个最直观的业务场景便是允许用户在表单中输入用户名和密码进行登录，而这背后的UsernamePasswordAuthenticationFilter，在整个Spring Security的认证体系中则扮演着至关重要的角色。

```java
// org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter#attemptAuthentication
public Authentication attemptAuthentication(HttpServletRequest request,
      HttpServletResponse response) throws AuthenticationException {
   //获取表单中的用户名和密码
   String username = obtainUsername(request);
   String password = obtainPassword(request);
   ...
   username = username.trim();
   //组装成username+password形式的token
   UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
         username, password);
   // Allow subclasses to set the "details" property
   setDetails(request, authRequest);
   //交给内部的AuthenticationManager去认证，并返回认证信息
   return this.getAuthenticationManager().authenticate(authRequest);
}
```
UsernamePasswordAuthenticationFilter本身的代码只包含了上述这么一个方法，非常简略，而在其父类`AbstractAuthenticationProcessingFilter`中包含了大量的细节，值得我们分析：

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
      implements ApplicationEventPublisherAware, MessageSourceAware {
	//包含了一个身份认证器
	private AuthenticationManager authenticationManager;
	//用于实现remeberMe
	private RememberMeServices rememberMeServices = new NullRememberMeServices();
	private RequestMatcher requiresAuthenticationRequestMatcher;
	//这两个Handler很关键，分别代表了认证成功和失败相应的处理器
	private AuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
	private AuthenticationFailureHandler failureHandler = new SimpleUrlAuthenticationFailureHandler();
	
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
		...
		Authentication authResult;
		try {
			//此处实际上就是调用UsernamePasswordAuthenticationFilter的attemptAuthentication方法
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				//子类未完成认证，立刻返回
				return;
			}
			sessionStrategy.onAuthentication(authResult, request, response);
		}
		//在认证过程中可以直接抛出异常，在过滤器中，就像此处一样，进行捕获
		catch (InternalAuthenticationServiceException failed) {
			//内部服务异常
			unsuccessfulAuthentication(request, response, failed);
			return;
		}
		catch (AuthenticationException failed) {
			//认证失败
			unsuccessfulAuthentication(request, response, failed);
			return;
		}
		//认证成功
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}
		//注意，认证成功后过滤器把authResult结果也传递给了成功处理器
		successfulAuthentication(request, response, chain, authResult);
	}
	
}
```
整个流程理解起来也并不难，主要就是内部调用了`authenticationManager`完成认证，根据认证结果执行successfulAuthentication或者unsuccessfulAuthentication，无论成功失败，一般的实现都是转发或者重定向等处理，不再细究AuthenticationSuccessHandler和AuthenticationFailureHandler，有兴趣的朋友，可以去看看两者的实现类。

### 4.4 AnonymousAuthenticationFilter

匿名认证过滤器，可能有人会想：匿名了还有身份？我自己对于Anonymous匿名身份的理解是Spirng Security为了整体逻辑的统一性，即使是未通过认证的用户，也给予了一个匿名身份。而AnonymousAuthenticationFilter该过滤器的位置也是非常的科学的，它位于常用的身份认证过滤器（如UsernamePasswordAuthenticationFilter、BasicAuthenticationFilter、RememberMeAuthenticationFilter）之后，意味着只有在上述身份过滤器执行完毕后，SecurityContext依旧没有用户信息，AnonymousAuthenticationFilter该过滤器才会有意义—-基于用户一个匿名身份。

```java
// org.springframework.security.web.authentication.AnonymousAuthenticationFilter

public class AnonymousAuthenticationFilter extends GenericFilterBean implements
      InitializingBean {

   private AuthenticationDetailsSource<HttpServletRequest, ?> authenticationDetailsSource = new WebAuthenticationDetailsSource();
   private String key;
   private Object principal;
   private List<GrantedAuthority> authorities;


   //自动创建一个"anonymousUser"的匿名用户,其具有ANONYMOUS角色
   public AnonymousAuthenticationFilter(String key) {
      this(key, "anonymousUser", AuthorityUtils.createAuthorityList("ROLE_ANONYMOUS"));
   }

   /**
    *
    * @param key key用来识别该过滤器创建的身份
    * @param principal principal代表匿名用户的身份
    * @param authorities authorities代表匿名用户的权限集合
    */
   public AnonymousAuthenticationFilter(String key, Object principal,
         List<GrantedAuthority> authorities) {
      Assert.hasLength(key, "key cannot be null or empty");
      Assert.notNull(principal, "Anonymous authentication principal must be set");
      Assert.notNull(authorities, "Anonymous authorities must be set");
      this.key = key;
      this.principal = principal;
      this.authorities = authorities;
   }

   ...

   public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
         throws IOException, ServletException {
      //过滤器链都执行到匿名认证过滤器这儿了还没有身份信息，塞一个匿名身份进去
      if (SecurityContextHolder.getContext().getAuthentication() == null) {
         SecurityContextHolder.getContext().setAuthentication(
               createAuthentication((HttpServletRequest) req));
      }
      chain.doFilter(req, res);
   }

   protected Authentication createAuthentication(HttpServletRequest request) {
     //创建一个AnonymousAuthenticationToken
      AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key,
            principal, authorities);
      auth.setDetails(authenticationDetailsSource.buildDetails(request));

      return auth;
   }
   ...
}
```
其实对比AnonymousAuthenticationFilter和UsernamePasswordAuthenticationFilter就可以发现一些门道了，UsernamePasswordAuthenticationToken对应AnonymousAuthenticationToken，他们都是Authentication的实现类，而Authentication则是被SecurityContextHolder(SecurityContext)持有的，一切都被串联在了一起。

### 4.5 ExceptionTranslationFilter

ExceptionTranslationFilter异常转换过滤器位于整个springSecurityFilterChain的后方，用来转换整个链路中出现的异常，将其转化，顾名思义，转化以意味本身并不处理。**一般其只处理两大类异常：AccessDeniedException访问异常和AuthenticationException认证异常。**

这个过滤器非常重要，因为它将Java中的异常和HTTP的响应连接在了一起，这样在处理异常时，我们不用考虑密码错误该跳到什么页面，账号锁定该如何，只需要关注自己的业务逻辑，抛出相应的异常便可。如果该过滤器检测到AuthenticationException，则将会交给内部的AuthenticationEntryPoint去处理，如果检测到AccessDeniedException，需要先判断当前用户是不是匿名用户，如果是匿名访问，则和前面一样运行AuthenticationEntryPoint，否则会委托给AccessDeniedHandler去处理，而AccessDeniedHandler的默认实现，是AccessDeniedHandlerImpl。所以ExceptionTranslationFilter内部的AuthenticationEntryPoint是至关重要的，顾名思义：认证的入口点。

```java
public class ExceptionTranslationFilter extends GenericFilterBean {
  //处理异常转换的核心方法
  private void handleSpringSecurityException(HttpServletRequest request,
        HttpServletResponse response, FilterChain chain, RuntimeException exception)
        throws IOException, ServletException {
     if (exception instanceof AuthenticationException) {
       	//重定向到登录端点
        sendStartAuthentication(request, response, chain,
              (AuthenticationException) exception);
     }
     else if (exception instanceof AccessDeniedException) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
		  //重定向到登录端点
           sendStartAuthentication(
                 request,
                 response,
                 chain,
                 new InsufficientAuthenticationException(
                       "Full authentication is required to access this resource"));
        }
        else {
           //交给accessDeniedHandler处理
           accessDeniedHandler.handle(request, response,
                 (AccessDeniedException) exception);
        }
     }
  }
}
```
简单提一下：AccessDeniedHandlerImpl这个默认实现类会根据errorPage和状态码来判断，最终决定跳转的页面。

```java
// org.springframework.security.web.access.AccessDeniedHandlerImpl#handle
public void handle(HttpServletRequest request, HttpServletResponse response,
      AccessDeniedException accessDeniedException) throws IOException,
      ServletException {
   if (!response.isCommitted()) {
      if (errorPage != null) {
         // Put exception into request scope (perhaps of use to a view)
         request.setAttribute(WebAttributes.ACCESS_DENIED_403,
               accessDeniedException);
         // Set the 403 status code.
         response.setStatus(HttpServletResponse.SC_FORBIDDEN);
         // forward to error page.
         RequestDispatcher dispatcher = request.getRequestDispatcher(errorPage);
         dispatcher.forward(request, response);
      }
      else {
         response.sendError(HttpServletResponse.SC_FORBIDDEN,
               accessDeniedException.getMessage());
      }
   }
}
```
### 4.6 FilterSecurityInterceptor

想想整个认证安全控制流程还缺了什么？我们已经有了认证，有了请求的封装，有了Session的关联…还缺一个：**由什么控制哪些资源是受限的，这些受限的资源需要什么权限，需要什么角色…这一切和访问控制相关的操作，都是由FilterSecurityInterceptor完成的**。

FilterSecurityInterceptor的工作流程用笔者的理解可以理解如下：**FilterSecurityInterceptor从SecurityContextHolder中获取Authentication对象，然后比对用户拥有的权限和资源所需的权限。前者可以通过Authentication对象直接获得，而后者则需要引入我们之前一直未提到过的两个类：SecurityMetadataSource，AccessDecisionManager**。理解清楚决策管理器的整个创建流程和SecurityMetadataSource的作用需要花很大一笔功夫，这里，暂时只介绍其大概的作用。

在JavaConfig的配置中，我们通常如下配置路径的访问控制：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.antMatchers("/resources/**", "/signup", "/about").permitAll()
            .antMatchers("/admin/**").hasRole("ADMIN")
            .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
            .anyRequest().authenticated()
			.withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
				public <O extends FilterSecurityInterceptor> O postProcess(
						O fsi) {
					fsi.setPublishAuthorizationSuccess(true);
					return fsi;
				}
			});
}
```
在ObjectPostProcessor的泛型中看到了FilterSecurityInterceptor，以笔者的经验，目前并没有太多机会需要修改FilterSecurityInterceptor的配置。

![](https://image.ldbmcs.com/2020-04-08-032414.jpg)

一个认证过程，其实就是过滤器链上的一个绿色矩形Filter所要执行的过程。

基本的认证过程有三步骤：

1. Filter拦截请求，生成一个未认证的Authentication，交由AuthenticationManager进行认证；

2. AuthenticationManager的默认实现ProviderManager会通过AuthenticationProvider对Authentication进行认证，其本身不做认证处理；

3. 如果认证通过，则创建一个认证通过的Authentication返回；否则抛出异常，以表示认证不通过。

## 5. 总结（Spring Security 认证流程）

1. 用户输入的账号密码等信息，这些东西其实就是用户的认证信息，对应到 Spring Security 中的话就是 Authentication 对象，只不过，Spring Security 中的 Authentication 对象除了保存用户的认证信息以外，还可以用来保存用户认证成功后从数据库中拿到的用户的详细信息。

   ```java
   public interface Authentication extends Principal, Serializable {
      Collection<? extends GrantedAuthority> getAuthorities();  // 用户权限
      Object getCredentials();                                  // 用户认证信息
      Object getDetails();                                      // 用户详细信息
      Object getPrincipal();                                    // 用户身份信息
      boolean isAuthenticated();                                // 当前 Authentication 是否已认证
      void setAuthenticated(boolean isAuthenticated);
   }
   ```

 2. 只凭用户提供的认证信息往往是不足以用来判断该用户是否合法的，因此，我们通常还需要某种手段来获取保存在服务端的用户信息，同时，也需要某种手段来保存用户信息，这些对应到 Spring Security 中的话就是 UserDetailsService 和 UserDetails 这两个对象。

   ```java
   public interface UserDetailsService {
      UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
   }

   public interface UserDetails extends Serializable {
      Collection<? extends GrantedAuthority> getAuthorities();
      String getPassword();
      String getUsername();
      boolean isAccountNonExpired();
      boolean isAccountNonLocked();
      boolean isCredentialsNonExpired();
      boolean isEnabled();
   }
   ```

3. 在拥有了用户提供的认证信息和保存在服务端的用户信息后，我们就需要通过某种方式来比较这两份信息，而这种来效验用户认证信息的对象对应到 Spring Security 中便是 AuthenticationManager 对象了。

   ```java
   Authentication authenticate(Authentication authentication)throws AuthenticationException;
   ```

   而鉴于各种各样的用户认证信息和层出不穷的效验方式，Spring Security 提供了更易于我们扩展的接口 AuthenticationProvider 和 ProviderManager 这个默认的 AuthenticationManager 实现。使用时，我们往往就只需要实现 AuthenticationProvider 就足够了。

   ```java
   public interface AuthenticationProvider {
      Authentication authenticate(Authentication authentication) throws AuthenticationException;
      boolean supports(Class<?> authentication);
   }
   ```
   可以看到，AuthenticationProvider 中的方法 authenticate 会返回一个 Authentication 对象，当通过认证后，这个对象往往会保存用户的详细信息。

4. 当用户的认证信息通过效验后，我们往往还需要在服务端保存通过的认证信息或生成的令牌，这个保存通过的认证信息的对象在 Spring Security 中就是 SecurityContext 和 SecurityContextHolder 这两个对象， SecurityContext 保存已通过认证的 Authentication 对象，SecurityContextHolder 保存 SecurityContext 到当前线程的上下文，方便我们的使用。

   ```java
   public interface SecurityContext extends Serializable {
      Authentication getAuthentication();
      void setAuthentication(Authentication authentication);
   }

   public class SecurityContextHolder {
      public static SecurityContext getContext();
      public void SecurityContext setContext();
   }
   ```
Spring Security 是基于 Filter 来实现的，而每个请求往往也会分配一个线程，因此，Spring Security 在请求到达具体的处理逻辑之前，就可以在 Filter 中完成用户信息的认证，生成 SecurityContext 方面后续的使用。  










