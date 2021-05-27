# 代码覆盖率-JaCoCo

> 转载：[代码覆盖率-JaCoCo](https://www.jianshu.com/p/4c728b39185e)

## 1. 代码覆盖率

在做单元测试时，代码覆盖率常常被拿来作为衡量测试好坏的指标，甚至，用代码覆盖率来考核测试任务完成情况，比如，代码覆盖率必须达到80％或 90％。

## 2. JaCoCo

官网：https://www.jacoco.org/jacoco/trunk/doc/counters.html

Jacoco从多种角度对代码进行了分析，包括指令（Instructions，C0 Coverage），分支（Branches，C1 Coverage），圈复杂度（Cyclomatic Complexity），行（Lines），方法（Methods），类（Classes）。

### 2.1 Instructions

Jacoco计算的最小单位就是字节码指令。**指令覆盖率表明了在所有的指令中，哪些被指令过以及哪些没有被执行**。这项指数完全独立于源码格式并且在任何情况下有效，不需要类文件的调试信息。

### 2.2 Branches

Jacoco对所有的if和switch指令计算了分支覆盖率。**这项指标会统计所有的分支数量，并同时支出哪些分支被执行，哪些分支没有被执行**。这项指标也在任何情况都有效。**异常处理**不考虑在分支范围内。

在有调试信息的情况下，分支点可以被映射到源码中的每一行，并且被高亮表示。

> 红色钻石：无覆盖，没有分支被执行。
> 黄色钻石：部分覆盖，部分分支被执行。
> 绿色钻石：全覆盖，所有分支被执行。

### 2.3 Cyclomatic Complexity

Jacoco为每个非抽象方法计算圈复杂度，并也会计算每个类，包，组的复杂度。根据**McCabe1996**的定义，**圈复杂度可以理解为覆盖所有的可能情况最少使用的测试用例数**。这项参数也在任何情况下有效。

### 2.4 Lines

该项指数在有调试信息的情况下计算。因为每一行代码可能会产生若干条字节码指令，所以我们用三种不同状态表示行覆盖率。

> 红色背景：无覆盖，该行的所有指令均无执行。
> 黄色背景：部分覆盖，该行部分指令被执行。
> 绿色背景：全覆盖，该行所有指令被执行。

### 2.5 Methods

每一个非抽象方法都至少有一条指令。若一个方法至少被执行了一条指令，就认为它被执行过。因为JaCoco直接对字节码进行操作，所以有些方法没有在源码显示（比如某些构造方法和由编译器自动生成的方法）也会被计入在内。

### 2.6 Classes

每个类中只要有一个方法被执行，这个类就被认定为被执行。同5一样，有些没有在源码声明的方法被执行，也认定该类被执行。

## 3. JaCoCo原理

参考网址：[http://www.open-open.com/lib/view/open1472174544246.html](https://link.jianshu.com/?t=http://www.open-open.com/lib/view/open1472174544246.html)
其中包含了注入探针以及修改字节码的相关原理。

## 4. 与maven集成

```xml
<plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.7.7.201606060606</version>
    <configuration>
        <destFile>target/coverage-reports/jacoco-unit.exec</destFile>
        <dataFile>target/coverage-reports/jacoco-unit.exec</dataFile>
    </configuration>
    <executions>
        <execution>
            <id>jacoco-initialize</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-site</id>
            <phase>package</phase>           
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 5. 实例解析

在经过与maven集成之后，生成的report文件在site/jacoco下，点开index.xml，即可查看生成报告。
包视图：

![2021-05-27-uurv2U](https://image.ldbmcs.com/2021-05-27-uurv2U.jpg)

类视图：

![2021-05-27-KXTqgF](https://image.ldbmcs.com/2021-05-27-KXTqgF.jpg)

方法视图：

![2021-05-27-WFRpMG](https://image.ldbmcs.com/2021-05-27-WFRpMG.jpg)

代码视图：

![2021-05-27-UmGuzn](https://image.ldbmcs.com/2021-05-27-UmGuzn.jpg)

以上三个表每个表都包含了五项指标数据。
 代码视图中，背景色代表的含义上文已经提到。
 宝石的颜色代表分支覆盖率，鼠标移动到黄色宝石上，将会提示如“1 of 2 branches missed”，对于“name==null”，有true和false两种分支，这说明程序只执行了一种分支。绿色宝石“All 2 branches covered”。红色宝石“All 2 branches missed”。