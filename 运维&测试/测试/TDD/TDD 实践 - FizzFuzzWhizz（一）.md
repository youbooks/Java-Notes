# TDD 实践 - FizzFuzzWhizz（一）

> 转载：[TDD 实践-FizzFuzzWhizz（一）](https://juejin.cn/post/6844903782938050573)

在[测试驱动开发（TDD）总结——原理篇](https://mp.weixin.qq.com/s/CYHshxaMtffmHms91LExnA)一文中已经对 TDD 做了概念性总结。而个人觉得理论知识的缺点在于它只强调外部刺激而缺乏学习者的内部心理过程，比如很难基于已有的经验对理论性知识建立映射关系，因此客观的实践才是检验真理的唯一标准。奔着这个目标，这些天花了一些时间去选择案例，因为好的故事或者好的案例既能让自己更有代入感，也能发挥 TDD 的魅力。

## 1. 前言

文章包括了案例，任务分解，报数部分的接口设计，单元测试命名规则和 TDD 案例实践，关于文章的源码已经放到[我的个人 Github](https://github.com/lynings/tdd-kata)，希望接下来的 TDD 实践系列文章对读者也有一些收获。

## 2. 范围

TDD （Test Driven Development） 在不同的圈子、不同的角色的认知中可能会有不同的理解，有人可能会理解成 ATDD（Acceptance Test Driven Development），也有人可能会理解成 UTDD（Unit Test Driven Development），**为了避免产生歧义，文章涉及到 TDD 专指 UTDD（Unit Test Driven Development），即 「单元测试驱动开发」。**

## 3. 准备

1. 理解 OOP。
2. 了解 Java 8。
3. 熟悉 Intellij IDEA。
4. 熟悉 TDD 理论性知识，可以参考 [测试驱动开发（TDD）总结——原理篇](https://mp.weixin.qq.com/s/CYHshxaMtffmHms91LExnA)。
5. 了解 Google 轻量级依赖注入框架 Guice  。
6. 熟悉测试工具 Junit 和 Mockito 的使用。
7. 熟悉搭建自动化单元测试环境。

## 4. 案例

![2021-03-21-jUIbjN](https://image.ldbmcs.com/2021-03-21-jUIbjN.jpg)

你是一名体育老师，在某次课距离下课还有五分钟时，你决定搞一个游戏。此时有 100 名学生在上课。游戏的规则是：

1. 你首先说出三个不同的特殊数，要求必须是个位数，比如 3、5、7。
2. 让所有学生排成一队，然后按顺序报数。
3. 学生报数时，如果所报数字是第一个特殊数（3）的倍数，那么不能说该数字，而要说 Fizz；如果所报数字是第二个特殊数（5）的倍数，那么要说 Buzz；如果所报数字是第三个特殊数（7）的倍数，那么要说 Whizz。
4. 学生报数时，如果所报数字同时是两个特殊数的倍数情况下，也要特殊处理，比如第一个特殊数和第二个特殊数的倍数，那么不能说该数字，而是要说 FizzBuzz, 以此类推。如果同时是三个特殊数的倍数，那么要说 FizzBuzzWhizz。
5. 学生报数时，如果所报数字包含了第一个特殊数，那么也不能说该数字，而是要说相应的单词，比如本例中第一个特殊数是 3，那么要报 13 的同学应该说 Fizz。如果数字中包含了第一个特殊数，那么忽略规则 3 和规则 4，比如要报 35 的同学只报 Fizz，不报 BuzzWhizz。

现在，我们需要你完成一个程序来模拟这个游戏，它首先接受 3 个特殊数，然后输出 100 名学生应该报数的数或单词。比如：

输入：

> 3,5,7

输出（片段）：

> 1,2,Fizz,4,Buzz,Fizz,Whizz,8,Fizz,Buzz,11,Fizz,Fizz,Whizz,FizzBuzz,16,17,Fizz,19,Buzz,…,100

## 5. 任务分解

在 TDD 之前进行需求分析可以在一开始就明确完成任务的目标是什么，以便于减少理解偏差所带来的低级错误；紧接着对需求进行任务分解，目的是得到一份可以被验证的任务清单。在实践 TDD 的过程可能还会调整任务清单，比如添加新的任务，或者删除掉冗余的任务等等。

在对需求进行分析的过程中，我会先从参与者的角度分析整个案例涉及到的角色有哪些，发现有两种角色参与到游戏中，分别是老师和学生；然后再从职责的角度分析得出老师的职责是发起游戏、定义游戏规则和说出 3 个不重复的个位数数字；学生的职责是参与游戏并根据游戏规则报数。最终我初步得出以下任务清单：

1. 发起游戏。
2. 定义游戏规则。
3. 说出 3 个不重复的个位数数字。
4. 学生报数。
5. 验证入参。

### 5.1 任务细分

我发现“学生报数”任务受到一系列游戏规则的影响，因此我会对该任务进行细化，并且寻找该任务的特殊需求，以便于对任务做一定程度的规划。在分析的过程中，对于比较特殊或者比较重要的规则我会做好标注，避免遗忘。下面是我细分后的任务清单：

1. 发起游戏。

2. 定义游戏规则。

3. 说出 3 个不重复的个位数数字。

4. !!!

    学生报数。

   - 如果是第一个特殊数字的倍数，就报 Fizz。
   - 如果是第二个特殊数字的倍数，就报 Buzz。
   - 如果是第三个特殊数字的倍数，就报 Whizz。
   - 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
   - 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
   - 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。

5. 验证入参。

## 6. 任务规划

有时候分解出来的任务没有体现出优先级和依赖关系，为了提高工作效率，需要对任务进行规划，让事情变得更加有条理，避免在一堆任务中迷失方向。

由于案例难度一般，所以这一步能发挥的空间不大，不过在这一步可以思考应该从哪个任务开始，选择的标准可以**参照**这三个：

- 任务的重要程度
- 任务的依赖关系
- 任务的难度

通过判断任务是否是主要流程来判断任务的重要程度，比如“验证入参”的重要程序相对“学生报数”较低，可以晚点做。

通过分析任务的依赖关系来识别任务的先后顺序，具体优先级因人而异，有人喜欢采用自顶向下，有人喜欢采用自底向上。好在 Mock 可以帮助开发人员隔离依赖，还可以通过 Mock 的方式驱动出类和接口而不依赖于具体实现，避免陷入寻找任务前后顺序的烦恼中。

分析任务的难度需要通过需求分析和经验得出，通过分析上面的案例可以知道难点在于学生报数的算法上。对于我个人来说，除非是核心任务，否则我不会一开始就选择难度大的非核心任务作为开始任务。

### 6.1 敲定任务

通过分析，刚好难度较大的任务“学生报数”是整个游戏的核心功能，最终我选择先做这个任务。

## 7. 测试命名规范

1. 测试类以 XXXTest 命名.
2. 测试方法命名必须采用`should_xxx_when_xxx`，例如：`should_return_false_when_1_is_greater_than_2`。
3. 测试方法的代码逻辑遵循 Given-When-Then 模式。

### 7.1 知识：Given-When-Then

在编写测试方法时，应该遵循 Given-When-Then 模式（在给定xx情况下，当做了xx操作，会得到xx反馈）这种模式可以让开发人员专注并思考以下这几件事情：

- Given：驱动我们思考这个测试是在一个怎样的上下文中，用到哪些对象，以便于思考需要创建哪些上下文和对象。
- When：驱动我们站在用户的角度去思考这个行为是什么，它有哪些输入，以便于思考方法的命名和入参。
- Then：驱动我们思考行为的反馈是什么，以便于思考方法的返回值。

### 7.2 思考：测试方法采用 should_xxx_when_xxx 的意义是什么？

得益于 BDD 思想和工具，这种命名方法是我在 BDD 的实践过程中琢磨出来的（当然不止我在用这种命名规则），它包含但不仅限于以下优点：

- 可以在把关注点放到行为上，避免陷入实现的细节中。
- 命名接近自然语言，表达意图清晰，可读性高，受益人群广。
- 很好地控制测试的范围，大到用户行为（偏 BDD），小到逻辑分支（偏 TDD）。

到现在需求已经明确，测试命名规范已拟定，任务已敲定，可以开始 TDD 了。

## 8. 常见错误

早期的开发习惯（编码-运行-观察）会导致开发人员过早陷入实现细节，这种开发习惯的缺陷之一在于反馈周期长，不利于小步快跑的节奏，所以在实践 TDD 的过程中需要时刻提醒自己 TDD 的口号和规则，培养自己养成新的思维习惯。

## 9. 测试驱动开发

![2021-03-21-8A6SuP](https://image.ldbmcs.com/2021-03-21-8A6SuP.jpg)

**敲定任务**

- 学生报数。
  - 如果是第一个特殊数字的倍数，就报 Fizz。
  - 如果是第二个特殊数字的倍数，就报 Buzz。
  - 如果是第三个特殊数字的倍数，就报 Whizz。
  - 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
  - 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
  - 如果不是特殊数字的倍数，也不包含第一个特殊数字，就报 Fizz。

------

**根据 TDD 的整体流程，此时需要想一下我要做什么，想想如何测试它，然后写一个小测试。思考所需的类、接口、输入和输出。**

根据之前的需求分析，学生需要明确自己对应的序号和游戏规则才能进行报数，因此驱动出 `Student` 类和 `String countOff(position, gameRules)` 方法，观察 `countOff` 方法发现需要用到游戏规则，所以还驱动出 `GameRule` 类。

------

**编写足够的代码使测试失败（明确失败总比模模糊糊的感觉要好）。**

```java
@Test
public void should_return_fizz_when_just_a_multiple_of_the_first_number() {
    List<GameRule> gameRules = new ArrayList<>();
    assertThat(Student.countOff(3, gameRules)).isEqualTo("Fizz");
}
```

这段代码运行的时候编译不通过，是因为缺少了必要的类和方法，所以我很快地补上了以下代码：

```java
public class Student {
    public static String countOff(Integer position, List<GameRule> gameRules) {
        return "";
    }
}

public class GameRule {
}
```

然后运行了单元测试得到以下错误消息：

![2021-03-21-M2R0xm](https://image.ldbmcs.com/2021-03-21-M2R0xm.jpg)

**编写刚刚好使测试通过的代码（保证之前编写的测试也需要通过）。**

检查完错误后，我发现加入了`GameRule`影响了我对代码明显实现的判断，所以此时我使用伪实现策略使测试尽快通过，以便于在持续细微的反馈中捕获明显实现，因此我很快键入以下伪代码：

```java
public static String countOff(Integer position, List<GameRule> gameRules) {
    return "Fizz";
}
```

谢天谢地测试通过了，非常快就得到了我想要的结果：

![2021-03-21-QR8nbJ](https://image.ldbmcs.com/2021-03-21-QR8nbJ.jpg)

我知道这段代码是有问题的，现在我在思考是继续编写`GameRule`使伪实现变成明显实现？还是挑下一个任务做并把“编写`GameRule`”记录到任务清单等之后再去做呢？这个选择的标准很简单，就是判断完成这个任务需要花多长时间，如果很快就能做完，那就继续做，如果需要花上一段时间，那就记下来跳下一个任务。通过我的分析，只需要给 `GameRule` 增加两个成员变量（数字和对应的术语）就可以达到我的目标， 然后我调整了对应的测试代码：

```java
@Test
public void should_return_fizz_when_just_a_multiple_of_the_first_umbe() {
    List<GameRule> gameRules = Lists.list(
        new GameRule(3, "Fizz"),
        new GameRule(5, "Buzz"),
        new GameRule(7, "Whizz")
    );
    assertThat(Student.countOff(3, gameRules)).isEqualTo("Fizz");
}
```

紧接着增加了以下代码：

```java
public class GameRule {
    private Integer number;
    private String term;

    public GameRule(Integer number, String term) {
        this.number = number;
        this.term = term;
    }

    ...
}

public class Student {
    public static String countOff(Integer position, List<GameRule> gameRules) {
        if (position % gameRules.get(0).getNumber() == 0) {
            return gameRules.get(0).getTerm();
        }
        return position.toString();
    }
}
```

然后运行测试：

![2021-03-21-XWfZtt](https://image.ldbmcs.com/2021-03-21-XWfZtt.jpg)

完美，很快就得到了测试通过的反馈。

因为目前测试和代码量很少，也没有明显的坏味道，所以暂时不需要重构，直接把当前子任务划掉并挑下个子任务。由于文章边幅有限，在重复多次 TDD 的整体流程后来到：

- 学生报数。
  - 如果是第一个特殊数字的倍数，就报 Fizz。
  - 如果是第二个特殊数字的倍数，就报 Buzz。
  - 如果是第三个特殊数字的倍数，就报 Whizz **（当前任务）**。
  - 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
  - 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
  - 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。

```java
public class StudentTest {

    private final List<GameRule> gameRules = Lists.list(
            new GameRule(3, "Fizz"),
            new GameRule(5, "Buzz"),
            new GameRule(7, "Whizz")
    );

    @Test
    public void should_return_1_when_mismatch_any_number() {
        assertThat(Student.countOff(1, gameRules)).isEqualTo("1");
    }

    @Test
    public void should_return_fizz_when_just_a_multiple_of_the_first_number() {
        assertThat(Student.countOff(3, gameRules)).isEqualTo("Fizz");
        assertThat(Student.countOff(6, gameRules)).isEqualTo("Fizz");
    }

    @Test
    public void should_return_buzz_when_just_a_multiple_of_the_second_number() {
        assertThat(Student.countOff(5, gameRules)).isEqualTo("Buzz");
        assertThat(Student.countOff(10, gameRules)).isEqualTo("Buzz");
    }

    @Test
    public void should_return_whizz_when_just_a_multiple_of_the_third_number() {
        assertThat(Student.countOff(7, gameRules)).isEqualTo("Whizz");
        assertThat(Student.countOff(14, gameRules)).isEqualTo("Whizz");
    }
}


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

此时代码的“坏味道”逐渐展示出来，需要引入重构阶段来消除重复设计，让小步快跑的节奏更加踏实。

## 10. 阅读系列文章：

- [测试驱动开发（TDD）总结——原理篇](https://mp.weixin.qq.com/s/CYHshxaMtffmHms91LExnA)

## 11. 源码

[github.com/lynings/tdd…](https://github.com/lynings/tdd-kata)