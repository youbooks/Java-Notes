# TDD 实践 - FizzFuzzWhizz（二）

> 转载：[TDD 实践 - FizzFuzzWhizz（二）](https://juejin.cn/post/6844903783248445453)

> 说明：该 TDD 系列案例主要是为了巩固和记录自己 TDD 实践过程中的思考与总结。个人认为 TDD 本身并不难，难的大部分是编程之外的技能，比如分析能力、设计能力、表达能力和沟通能力；所以在 TDD 的过程中，个人认为 TDD 可以锻炼一个人事先思考、化繁为简、制定计划、精益求精的习惯和品质。本文的源码放在[个人的 Github](https://github.com/lynings/tdd-kata) 上，案例需求来自于网上。

## 1. 目标收益

1. 熟悉掌握 TDD 整体流程。
2. 识别代码坏味道 Deplicated Code 以及重构手法。
3. 了解 java8 特性 lambda 和部分函数式接口的使用。
4. 得到满意的测试覆盖率。
5. 提高对代码的自信和重构的勇气。

## 2. 任务回顾

- 学生报数。
  1. 如果是第一个特殊数字的倍数，就报 Fizz。
  2. 如果是第二个特殊数字的倍数，就报 Buzz。
  3. 如果是第三个特殊数字的倍数，就报 Whizz **（当前任务）**。
  4. 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
  5. 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
  6. 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。

## 3. 代码回顾

```java
public class Student {

    public static String countOff(Integer position, List<GameRule> gameRules) {
        if (position % gameRules.get(0).getNumber() == 0) {
            return gameRules.get(0).getTerm();
        } else if (position % gameRules.get(1).getNumber() == 0) {
            return gameRules.get(1).getTerm();
        } else if (position % gameRules.get(2).getNumber() == 0) {
            return gameRules.get(2).getTerm();
        }
        return position.toString();
    }
}
```

## 4. 测试驱动开发

![2021-03-21-s81SyC](https://image.ldbmcs.com/2021-03-21-s81SyC.jpg)

**如果有任何重复的逻辑或无法解释的代码，重构可以消除重复并提高表达能力（减少耦合，增加内聚力）。**

继[上一篇文章](https://mp.weixin.qq.com/s/5aM-FrUN3dtcet_XfA8xQw)的内容，此时我们需要解决代码中的坏味道——Duplicated Code。分析发现，代码之间只是类似，并非完全相同，而且代码表达的意图很不清晰，可以使用 Extract Method 重构手法来解决这个问题，通过抽出 `isMultiple` 方法用于判学生的序号是否是特殊数的倍数，使代码意图清晰一些，很快我就完成了初步的重构：

```java
public class Student {

    public static String countOff(Integer position, List<GameRule> gameRules) {
        if (isMultiple(position, gameRules.get(0).getNumber())) {
            return gameRules.get(0).getTerm();
        } else if (isMultiple(position, gameRules.get(1).getNumber())) {
            return gameRules.get(1).getTerm();
        } else if (isMultiple(position, gameRules.get(2).getNumber())) {
            return gameRules.get(2).getTerm();
        }
        return position.toString();
    }

    private static boolean isMultiple(Integer divisor, Integer dividend) {
        return divisor % dividend == 0;
    }
}
```

运行自动化测试全部通过，不过取值的方式还是有点笨，然后我把上面那种取值方式改成通过循环自动取值以降低错误率，此时代码变得更加简洁，表达的意图也更加清晰：

```java
public static String countOff(Integer position, List<GameRule> gameRules) {
    for (GameRule gameRule : gameRules) {
        if (isMultiple(position, gameRule.getNumber())) {
            return gameRule.getTerm();
        }
    }
    return position.toString();
}

private static boolean isMultiple(Integer divisor, Integer dividend) {
    return divisor % dividend == 0;
}
```

**再次运行测试验证重构是否引入新的错误。如果没有通过，很可能是在重构时犯了一些错误，需要立即修复并重新运行，直到所有测试通过。**

![2021-03-21-IKw9iK](https://image.ldbmcs.com/2021-03-21-IKw9iK.jpg)

经过自动化测试的检验，测试全部通过，此时可以放心开始下一个子任务。



------

如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。

从描述中可以看出第 4 个子任务也非常简单，很快我就写好了对应的单元测试，并驱动出了具体实现：

```java
public class StudentTest {
    private final List<GameRule> gameRules = Lists.list(
            new GameRule(3, "Fizz"),
            new GameRule(5, "Buzz"),
            new GameRule(7, "Whizz")
    );
    ...
    @Test
    public void should_return_fizzbuzz_when_just_a_multiple_of_the_first_number_and_second_number() {
        assertThat(Student.countOff(15, gameRules)).isEqualTo("FizzBuzz");
        assertThat(Student.countOff(45, gameRules)).isEqualTo("FizzBuzz");
    }

    @Test
    public void should_return_fizzwhizz_when_just_a_multiple_of_the_first_number_and_third_number() {
        assertThat(Student.countOff(21, gameRules)).isEqualTo("FizzWhizz");
        assertThat(Student.countOff(42, gameRules)).isEqualTo("FizzWhizz");
        assertThat(Student.countOff(63, gameRules)).isEqualTo("FizzWhizz");
    }

    @Test
    public void should_return_buzzwhizz_when_just_a_multiple_of_the_second_number_and_third_number() {
        assertThat(Student.countOff(35, gameRules)).isEqualTo("BuzzWhizz");
        assertThat(Student.countOff(70, gameRules)).isEqualTo("BuzzWhizz");
    }

    @Test
    public void should_return_fizzbuzzwhizz_when_at_the_same_time_is_a_multiple_of_the_three_number() {
        List<GameRule> gameRules = Lists.list(
                new GameRule(2, "Fizz"),
                new GameRule(3, "Buzz"),
                new GameRule(4, "Whizz")
        );
        assertThat(Student.countOff(24, gameRules)).isEqualTo("FizzBuzzWhizz");
        assertThat(Student.countOff(48, gameRules)).isEqualTo("FizzBuzzWhizz");
        assertThat(Student.countOff(96, gameRules)).isEqualTo("FizzBuzzWhizz");
    }
}

public class Student {
    public static String countOff(Integer position, List<GameRule> gameRules) {
    
        if (isMultiple(position, gameRules.get(0).getNumber()) 
                && isMultiple(position, gameRules.get(1).getNumber()) 
                && isMultiple(position, gameRules.get(2).getNumber())) {
            return gameRules.get(0).getTerm() + gameRules.get(1).getTerm() + gameRules.get(2).getTerm();
        } else if (isMultiple(position, gameRules.get(0).getNumber()) 
                && isMultiple(position, gameRules.get(1).getNumber())) {
            return gameRules.get(0).getTerm() + gameRules.get(1).getTerm();
        } else if (isMultiple(position, gameRules.get(0).getNumber()) 
                && isMultiple(position, gameRules.get(2).getNumber())) {
            return gameRules.get(0).getTerm() + gameRules.get(2).getTerm();
        } else if (isMultiple(position, gameRules.get(1).getNumber()) 
                && isMultiple(position, gameRules.get(2).getNumber())) {
            return gameRules.get(1).getTerm() + gameRules.get(2).getTerm();
        }
    
        for (GameRule gameRule : gameRules) {
            if (isMultiple(position, gameRule.getNumber())) {
                return gameRule.getTerm();
            }
        }
        return position.toString();
    }
}
```

此时我遇到了两个问题，一个是第四个子任务的描述缺了 FizzWhizz 这种可能，所以我先完善了任务清单；第二个是我又从代码中闻到熟悉的坏味道，因此在自动化测试的支撑下，我开始建立起自信，并解决了 if else 过于冗长的问题：

```java
public static String countOff(Integer position, List<GameRule> gameRules) {
    String terms = gameRules
            .stream()
            .filter(rule -> isMultiple(position, rule.getNumber()))
            .map(rule -> rule.getTerm())
            .reduce((t1, t2) -> t1 + t2)
            .orElse(null);
    if (terms != null) {
        return terms;
    }

    for (GameRule gameRule : gameRules) {
        if (isMultiple(position, gameRule.getNumber())) {
            return gameRule.getTerm();
        }
    }
    return position.toString();
}
```

此时自动化测试全部通过，然后分析发现，下面的 `for` 循环已经变成冗余代码，因为它已经被合并到新写入的代码中，现在可以删除掉它了：

```java
public static String countOff(Integer position, List<GameRule> gameRules) {
    String term = gameRules
            .stream()
            .filter(rule -> isMultiple(position, rule.getNumber()))
            .map(rule -> rule.getTerm())
            .reduce((t1, t2) -> t1 + t2)
            .orElse(position.toString());
    return term;
}
```

自动化测试全部通过，这里我引入 java 8 的特性 lambel 和函数式接口，函数式编程在代码实现层面增强了代码的语义，也使得代码更加精练，如今总算得到一份满意的代码，可以开始“学生报数”的最后一个子任务。

**学生报数**

1. 如果是第一个特殊数字的倍数，就报 Fizz。
2. 如果是第二个特殊数字的倍数，就报 Buzz。
3. 如果是第三个特殊数字的倍数，就报 Whizz **（当前任务）**。
4. 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、FizzWhizz、BuzzWhizz、FizzBuzzWhizz。
5. 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
6. 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。

看着心里乐，最后一个子任务预计 2 分钟搞定，然后就可以把“学生报数”这个核心任务划掉。于是乎我很快的编写了对应的单元测试，并驱动出对应的具体实现：

```java
public class StudentTest {
    private final List<GameRule> gameRules = Lists.list(
            new GameRule(3, "Fizz"),
            new GameRule(5, "Buzz"),
            new GameRule(7, "Whizz")
    );
    ...
    @Test
    public void should_return_fizz_when_included_the_first_number() {
        assertThat(Student.countOff(3, gameRules)).isEqualTo("Fizz");
        assertThat(Student.countOff(13, gameRules)).isEqualTo("Fizz");
        assertThat(Student.countOff(30, gameRules)).isEqualTo("Fizz");
        assertThat(Student.countOff(31, gameRules)).isEqualTo("Fizz");
    }
}

public class Student {

    public static String countOff(final Integer position, final List<GameRule> gameRules) {
        if (position.toString().contains(gameRules.get(0).getNumber().toString())) {
            return gameRules.get(0).getTerm();
        }
        String term = gameRules
                .stream()
                .filter(rule -> isMultiple(position, rule.getNumber()))
                .map(rule -> rule.getTerm())
                .reduce((t1, t2) -> t1 + t2)
                .orElse(position.toString());
        return term;
    }

    private static boolean isMultiple(Integer divisor, Integer dividend) {
        return divisor % dividend == 0;
    }
}
```

运行自动化单元测试：

![2021-03-21-NfQAR1](https://image.ldbmcs.com/2021-03-21-NfQAR1.jpg)

新增的单元测试通过，但是却出现其它三个单元测试执行失败，出现这种情况我下意识觉得是新加入的代码有 BUG，因为是在我加入实现代码之后才出现测试失败的情况。经过分析，发现原来是最后一个子任务优先级最高，而刚好那些失败的单元测试的部分测试样本数据受到当前子任务的条件约束，解决起来很简单，删除对应的测试代码就好，现在所有单元测试运行通过，并且完成“学生报数”任务。

**知识：是什么让开发人员变得更有勇气去重构代码？**

这得益于 TDD 的核心思想——不可运行/可运行/重构。这样的机制可以保证拥有足够多的单元测试以便于支撑实施代码重构，在细微持续的反馈中可以非常自信的做到小步快跑，因为我们可以非常放心的把“后背”交给自动化 BUG 侦察机。

**讨论：新加入的代码是否需要再优化？**

可能有人觉得新加入的代码 `if(...)` 有点冗长，表达的含义也不是特别清晰，其实我也有很强烈的代码洁癖症（处女座一枚），不过现在的节奏我是认为很好了，如果还需要优化，我认为只需补充加上适当的注释表明代码的意图。您觉得呢？期待您的建议。

**反思：到目前为止，程序是否存在更加优秀的设计？**

我认为是的，不过目前看起来还不错，具体等到引入游戏上下文和实现其它任务时再综合思考这个问题。

## 5. TDD 成果

任务清单：

1. 发起游戏。
2. 定义游戏规则。
3. 说出 3 个不重复的个位数数字。
4. **!!!** 学生报数。
   - 如果是第一个特殊数字的倍数，就报 Fizz。
   - 如果是第二个特殊数字的倍数，就报 Buzz。
   - 如果是第三个特殊数字的倍数，就报 Whizz。
   - 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
   - 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
   - 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。
5. 验证入参。

测试报告：

![2021-03-21-F6EQfJ](https://image.ldbmcs.com/2021-03-21-F6EQfJ.jpg)

测试覆盖率：

![2021-03-21-Ag4TeG](https://image.ldbmcs.com/2021-03-21-Ag4TeG.jpg)

截止到目前一共编写了 9 个单元测试并驱动出“学生报数”功能，测试覆盖率几乎到达 100%（除了 Student 构造函数没有被覆盖），完成了案例中最核心的功能。在这个过程通过实践不断加深对 TDD 整体流程的理解，慢慢熟悉如何识别代码中的坏味道，同时也掌握一些重构手法，有趣的是我之前一直以为分析技术只会在需求分析和任务分解这两个阶段才会用到，现在看来在编程的过程中经常会使用到分析技术，收获还不错，可别忘了还有一点，在这个过程中自己变得越来越自信，越来越有勇气去写更好的代码。

## 6. 阅读系列文章

- [测试驱动开发（TDD）总结——原理篇](https://mp.weixin.qq.com/s/CYHshxaMtffmHms91LExnA)
- [TDD 实践-FizzFuzzWhizz（一）](https://mp.weixin.qq.com/s/5aM-FrUN3dtcet_XfA8xQw)

## 7. 源码

[github.com/lynings/tdd…](https://github.com/lynings/tdd-kata)

