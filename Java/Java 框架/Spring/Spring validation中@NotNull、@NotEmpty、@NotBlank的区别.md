> 转载：[@NotNull、@NotEmpty、@NotBlank的区别](https://www.jianshu.com/p/c9e71b241daa)

大致区别如下：

- @NotEmpty用在集合类上面。

- @NotBlank 用在String上面。

- @NotNull 用在基本类型上。

只有简单的结果，但是再更具体一点的内容就搜不到了，所以去看了看源码，发现了如下的注释：

## 1. @NotEmpty

```java
/** * Asserts that the annotated string,collection, map or array is not {**@code **null} or empty. 
 @author  Emmanuel Bernard  @author  Hardy Ferentschik
 ***/
@Documented
@Constraint(validatedBy = { })
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@ReportAsSingleViolation
@NotNull
@Size(min = 1)
public @interface NotEmpty {
   String message() default "{org.hibernate.validator.constraints.NotEmpty.message}";
   Class<?>[] groups() default { };
}  
```

也就是说，加了@NotEmpty的String类、Collection、Map、数组，是不能为null并且长度必须大于0的（String、Collection、Map的isEmpty()方法）。

## 2. @NotBlank

```java
/** Validate that the annotated string isnot {@code null} or empty.  The difference to {@code NotEmpty}is that trailing whitespaces are getting ignored. @author Hardy Ferentschik 
***/
@Documented
@Constraint(validatedBy = { })
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR,PARAMETER })
@Retention(RUNTIME)
@ReportAsSingleViolation
@NotNull
public @interface NotBlank {
   String message() default "{org.hibernate.validator.constraints.NotBlank.message}";
}  
```

注意：@NotBlank用于String类型。

“The difference to {@code NotEmpty} is that trailingwhitespaces are getting ignored.” –> 和{@code NotEmpty}不同的是，尾部空格被忽略，也就是说，纯空格的String也是不符合规则的。所以才会说@NotBlank用于String，只能作用在String上，不能为null，而且调用trim()后，长度必须大于0。

("test") 即：必须有实际字符。

## 3. @NotNull

```java
/***  The annotated element must not be {@code null}. Accepts any type. 
@author Emmanuel Bernard 
**/
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = { })
public @interface NotNull {
   String message()  default "{javax.validation.constraints.NotNull.message}";  
   Class<?>[] groups() default { };
}  
```

这个就很好理解了，不能为null，但可以为empty。

examples:

```java

1.String name = null;

@NotNull: false

@NotEmpty: false

@NotBlank: false

2.String name = "";

@NotNull: true

@NotEmpty: false

@NotBlank: false

3.String name = " ";

@NotNull: true

@NotEmpty: true

@NotBlank: false

4.String name = "Great answer!";

@NotNull: true

@NotEmpty:true

@NotBlank:true

```

附上一个使用例子：

```java
@NotBlank(message = "startTime must not be null")
private String startTime;
@NotBlank(message = "endTime must not be null")
private String endTime;
@NotNull(message = "areaType must not be null")
private Integer areaType;
@NotBlank(message = "userId must not be null")
private String userId;
```

