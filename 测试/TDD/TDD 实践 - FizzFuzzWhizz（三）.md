# TDD 实践 - FizzFuzzWhizz（三）

> 转载：[TDD 实践 - FizzFuzzWhizz（三）](https://juejin.cn/post/6844903798683467789)

> 说明：该 TDD 系列案例主要是为了巩固和记录自己 TDD 实践过程中的思考与总结。个人认为 TDD 本身并不难，难的大部分是编程之外的技能，比如分析能力、设计能力、表达能力和沟通能力，它可以锻炼一个人事先思考、化繁为简、制定计划、精益求精的习惯和品质。本文的源码放在个人的 Github 上，案例需求来自于网上。

在之前的实践文章中着重掌握 TDD 的口号和整体流程，用 9 个 UT 驱动出核心任务的实现代码，即完成了核心任务，也得到了将近 100% 的测试覆盖率，并且在测试的支撑下对程序进行小范围重构，从目前看来采用 TDD 的效果还是不错的。不过上一篇文章留下了一个反思一直困扰着我，并不是因为这个问题有多难解决，而是以后再面对这种类型的问题时，我可以运用何种思路去简化并解决这种类型的问题，这是写这篇文章最主要的动机，而 TDD 可以帮助到我。

## 1. 问题回顾

**到目前为止，程序是否存在更加优秀的设计？** 这是上一篇文章结尾留下的一个反思，提这个问题的同时也引发了自己对程序设计的反思，到底应该通过什么方法来降低错误设计和编程浪费（重写和太多的重构）呢？

## 2. 解题思路

**目标：** 得到简洁可用的代码和合理的程序设计。

**准备：** 简化案例为接下来的活动做好准备。

**活动：**

1. 通过面向对象分析提炼领域模型或分析模型。
2. 在领域模型的指导下实施面向对象设计得到设计模型。
3. 可视化领域模型和设计模型，以便于理解和分析。
4. 任务分解。
5. TDD。

## 3. 面向对象分析

**OOA 强调的是在问题领域内发现和描述对象（或概念）， 关注重要概念类、属性和关键关系，强调调查研究，而非解决方案， 最终提炼出领域模型**。得到领域模型是我第一阶段的目标。这里还要强调的是，**建模的目的是为了理解和沟通**，以便于确认模型的合理性，所以建模是一项重要但不应该花太多时间的工作，既可以在白板上草图绘制，也可以使用 UML 元素。

**创建领域模型的步骤可以分为四步：**

1. 确认需求范围。
2. 寻找重要概念类。
3. 可视化（草图或 UML）。
4. 添加关联关系和属性。

经分析，由于案例难度一般，所以当前需求范围涉及整个案例（或者当前迭代所涉及的需求），其中第三点需要学习相关的元素或者画草图即可，难度相对简单，而第二点和第四点是整个建模过程中的关键步骤，直接影响到整个领域模型，所以把主要精力放在这两个地方。

**如何寻找概念类：**

1. 在已有的模型上调整和修改（效率较高，推荐）。
2. 使用分类列表。
3. 确定名词短语（较为简单，需要注意自然语言的二义性）。

这里我使用"分类列表"策略来寻找重要概念类，在分析案例后我得到以下分类列表：

| 概念类类别 | 重要等级 | 示例             |
| ---------- | -------- | ---------------- |
| 游戏       | 重要     | Game             |
| 游戏规则   | 关键     | Rule             |
| 参与者     | 重要     | Teacher、Student |
| 地点       | 无关紧要 | Classroom        |

经分析，该案例中的核心在于游戏规则，也是整个案例中的实现难点，Classroom 在当前需求中缺乏业务含义，可以剔除，然后我使用 UML 工具快速绘制初步的领域模型：

![2021-03-21-vdEu7S](https://image.ldbmcs.com/2021-03-21-vdEu7S.jpg)

然后给模型加上关联关系和属性：

![2021-03-21-tXLM2T](https://image.ldbmcs.com/2021-03-21-tXLM2T.jpg)

这个模型主要是站在参与者 `Student` 的角度进行分析和建模，也是我一开始的理解，但是经过分析，我发现这个模型有点混乱，`Student` 可能同时跟 `Teacher` 和 `Game` 存在耦合关系， `start` 和 `play`也存在歧义，所以我再次站在参与者的角度对模型进行调整：

![2021-03-21-sWr4lZ](https://image.ldbmcs.com/2021-03-21-sWr4lZ.jpg)

我把 `Teacher` 从领域模型中去掉，并且将 `Student` 概念抽象为 `Player`，此时整个领域模型变得简洁多了，不过在进一步分析领域模型发现模型中的 `Player` 跟 `Game` 之间的关系非常奇怪，由 100 个玩家去玩一个游戏，导致游戏里面包含了 100 个玩家对象，而且每个玩家的唯一标识仅仅是通过序号来区分，那么 `Player` 到底是概念类还是属性呢？这两者要如何区分呢？

**如何分辨概念类和属性？**

> 如果某个概念类不是现实世界中的数字和文本，那么该概念类很可能是概念类而不是属性，反之同理。

这是在《UML与模式应用》这本书的第九章中提到的准则，也正是这句话给了我灵感，通过分析，在这个案例中并没有明确区分玩家是张三还是李四，因此“100个玩家”应该作为游戏的一个属性更加合适不过，所以我重新调整了模型：

![2021-03-21-8DE7Kr](https://image.ldbmcs.com/2021-03-21-8DE7Kr.jpg)

此时的领域模型得到了简化，领域聚焦也变得更加合理。那 `Rule` 的设计合理吗？通过分析案例发现 `Game` 跟 `Rule` 的关系存在问题，`Game` 和 `Rule` 并不是 1 : 3 的关系，而是 1 : 1 的关系，因为特殊数字只是游戏规则的一部分，这个问题在程序设计阶段可以非常明显的反映出来，此时模型调整为：

![2021-03-21-JwVGCs](https://image.ldbmcs.com/2021-03-21-JwVGCs.jpg)

我把一些需要重点关注的信息标上了红色，以便于聚焦自己的关注点。 虽然在识别概念类和建模的过程中遇到了一些小问题，但是最终还是得到一个合理的领域模型。**领域模型是对概念内的概念类或现实世界中对象的可视化表达**，它去掉了问题域中的大部分细枝末节，只保留重要的领域概念，这对于分析问题和指导程序设计提供了非常大的帮助，不过**有时候领域模型看不出来的问题在程序设计阶段可能会反映出来，所以可以先采用领域模型指导程序设计，然后反过来通过程序设计的反馈来分析领域模型的合理性**。

**思考：如何判断领域模型是否正确？**

不同的分析角度得到的领域模型可能存在异同，所以并没有所谓正确的领域模型，模型只是近似地在尝试描述一个实际领域，它可以有效地捕获理解当前问题域所需要的重要信息，帮助人们理解当前问题域中的概念、术语和关系，以便于提高开发人员和业务人员之间的理解和沟通效率。

**思考：应该花多少时间去建立领域模型？**

假设在为期 3 周的迭代中，建模时间最好不要超过一天，因为有很多因素阻碍我们建立一个"完美"的模型，例如业务需求变更、部分重要概念未被发掘等等情况，所以应该把主要精力花在核心问题域的分析和建模，然后通过程序设计阶段反过来去验证模型的合理性，在 TDD 的过程中通过重构来发现隐式概念并提炼程序设计，因此在实践过程中运用好 OOA 、 OOD 和 TDD 可以达到相辅相成的作用。

## 4. 面向对象设计

**OOD 关注软件对象的职责和协作以实现软件需求，强调得到满足当前需求的概念上的解决方案（可以被实现，但不是具体实现，设计思想不关注具体实现），而实现则表达了真实且完整的设计，通常会使用 UML 交互图和类图进行可视化**。

**动态建模和静态建模的区别**

动态模型有助于设计逻辑和设计代码行为，通常会使用如下工具进行动态建模：

- UML 交互图
- UML 顺序图
- UML 活动图
- UML 状态机图
- ...

静态建模有助于设计包、类名、属性和方法，通常会使用如下工具进行静态建模：

- UML 类图
- ...

**其中最有价值、最具挑战性、最有益和有效的设计工作基本上都发生在动态建模的过程中，在动态建模的过程中可以明确知道有哪些对象，对象之间如何进行交互，所以应该在这方面花费更多的精力**。

**思考：领域模型和设计模型（UML交互图、类图）之间是什么关系？**

设计模型并不是完完全全临摹领域模型，因为这两个阶段的目标完全不同， 领域模型描述的是真实世界领域内的概念类，设计模型描述的是软件类（可以被某种编程语言实现），因此设计阶段需要结合编程语言、编程思想和设计原则，而领域模型在这里可以给予设计师灵感并指导程序设计。

在明确设计建模的输入（领域模型）和输出（设计模型）之后，现在需要明确输入到输出的中间过程。在进行动态建模的过程中，我主要采用了**职责驱动设计和 GRASP**来帮助我实施这一过程（通常可能还会有更多的活动，比如开讨论会等）。

**知识：职责驱动设计**

职责驱动设计把软件对象想象成具有某种职责的人，这个人要与其他人协作以完成工作，这个过程中会考虑职责、角色和协作三大要素。

**知识：GRASP**

GRASP 是 GRAS Pattern 的缩写，是一系列模式的统称，它可以以一种系统的、合理的、可以被解释的方式来运用职责驱动设计，并进行设计推理，详细描述请阅读《UML与模式应用》。

### 4.1 构建 UML 交互图

> UML 交互图用于动态对象建模，它可以帮助描述对象之间的交互。

在经过上面一番对 OOD 的理解之后，这里我使用 UML 交互图中的顺序图（UML交互图还包括通信图等）来描述对象之间的交互，在领域模型的指导下我得到一个初步的 UML 交互图：

![2021-03-21-fb7l28](https://image.ldbmcs.com/2021-03-21-fb7l28.jpg)

这个图体现了 `Game` 和 `Rule` 的职责和协作，`Game` 承担玩游戏 `play(specialNumbers)` 职责，`Rule` 承担核心的规则匹配 `match(number)` 职责，`Game` 和 `Rule` 的协作体现在对象的创建和方法的调用。

到这里 UML 交互图总算大功告成，接下来进入设计类图阶段。

### 4.2 构建 UML 类图

> 类图用于静态对象建模，可以用来描述类、接口、属性及其关联关系。

在领域模型和 UML 交互图的指导下，类图已经变得非常简单，因为大部分信息都已经在 UML 交互图体现出来，所以我根据 UML 交互图快速设计了以下类图：

![2021-03-21-nxpSfV](https://image.ldbmcs.com/2021-03-21-nxpSfV.jpg)

到这里 UML 类图构建完毕，在进行任务分解后就可以开始 TDD。

## 5. 任务分解

OOD 已经让我得到了实现需求的解决方案，有趣的是设计模型很大程度上已经帮我完成了任务分解的过程，所以我根据设计模型修改了一开始的任务清单。

**旧的任务清单：**

1. 发起游戏。
2. 定义游戏规则。
3. 说出 3 个不重复的个位数数字。
4. **!!!** 学生报数。
   1. 如果是第一个特殊数字的倍数，就报 Fizz。
   2. 如果是第二个特殊数字的倍数，就报 Buzz。
   3. 如果是第三个特殊数字的倍数，就报 Whizz。
   4. 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
   5. 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
   6. 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。
5. 验证入参。

**修改后的任务清单：**

1. 生成 3 个不重复的个位数数字。

2. !!!

    定义游戏规则。

   1. 如果是第一个特殊数字的倍数，就报 Fizz。
   2. 如果是第二个特殊数字的倍数，就报 Buzz。
   3. 如果是第三个特殊数字的倍数，就报 Whizz。
   4. 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
   5. 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
   6. 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。
   7. 重写 `Student` 类，使用 `Rule` 替换 `Student`。

3. 验证入参。

## 6. 测试驱动开发

> Kent Beck：“测试驱动开发不是一种测试技术。它是一种分析技术、设计技术，更是一种组织所有开发活动的技术”。

**有效的分析和设计可以概括为：做正确的事（分析）和正确地做事（设计）**，这应该就是 TDD 中提到的分析技术和设计技术吧。

在之前的 TDD 实践文章中我驱动出了 `Student` 并为其分配了报数 `countOff()` 职责，但是这跟目前的设计模型完全不一样，好在 TDD 让程序留下了单元测试，可以在自动化测试的支撑下进行安全的重写，所以我运用了以下策略帮助我完成重写任务：

1. 在不修改原来的代码的前提下逐渐进行小范围重写和替换
   1. 将 `Student` 替换成 `Rule`，并使自动化测试通过。
   2. 引入并初始化 `List<Rule.Item> items` 成员变量，用以保存特殊数字和单词的映射关系。
   3. 创建 `match(number)` 方法。
   4. 逐渐使所有测试通过的方式驱动 `match(number)` 方法的实现。
   5. 在保证新引入的代码全部通过测试之后删除旧代码。
2. 运行自动化测试以保证没有引入新的错误。
3. 如果引入错误则马上修改使测试通过。
4. 回到第一步，直到完成重写任务。
5. 保证重构任务完成和所有测试通过的情况下，删除多余的代码。

------

遵循上面的重写策略最终我得到了以下代码：

```java
public class RuleTest {

    final Rule rule = new Rule(Arrays.asList(3, 5, 7));

    @Test
    public void should_return_1_when_mismatch_any_number() {
        assertThat(rule.match(1)).isEqualTo("1");
    }

    @Test
    public void should_return_fizz_when_just_a_multiple_of_the_first_number() {
        assertThat(rule.match(3)).isEqualTo("Fizz");
        assertThat(rule.match(6)).isEqualTo("Fizz");
    }

    @Test
    public void should_return_buzz_when_just_a_multiple_of_the_second_number() {
        assertThat(rule.match(5)).isEqualTo("Buzz");
        assertThat(rule.match(10)).isEqualTo("Buzz");
    }

    @Test
    public void should_return_whizz_when_just_a_multiple_of_the_third_number() {
        assertThat(rule.match(7)).isEqualTo("Whizz");
        assertThat(rule.match(14)).isEqualTo("Whizz");
    }

    @Test
    public void should_return_fizzbuzz_when_just_a_multiple_of_the_first_number_and_second_number() {
        assertThat(rule.match(15)).isEqualTo("FizzBuzz");
        assertThat(rule.match(45)).isEqualTo("FizzBuzz");
    }

    @Test
    public void should_return_fizzwhizz_when_just_a_multiple_of_the_first_number_and_third_number() {
        assertThat(rule.match(21)).isEqualTo("FizzWhizz");
        assertThat(rule.match(42)).isEqualTo("FizzWhizz");
    }

    @Test
    public void should_return_buzzwhizz_when_just_a_multiple_of_the_second_number_and_third_number() {
        assertThat(rule.match(70)).isEqualTo("BuzzWhizz");
    }

    @Test
    public void should_return_fizzbuzzwhizz_when_at_the_same_time_is_a_multiple_of_the_three_number() {
        Rule rule = new Rule(Arrays.asList(2, 3, 4));
        assertThat(rule.match(48)).isEqualTo("FizzBuzzWhizz");
        assertThat(rule.match(96)).isEqualTo("FizzBuzzWhizz");
    }

    @Test
    public void should_return_fizz_when_included_the_first_number() {
        assertThat(rule.match(3)).isEqualTo("Fizz");
        assertThat(rule.match(13)).isEqualTo("Fizz");
        assertThat(rule.match(30)).isEqualTo("Fizz");
        assertThat(rule.match(31)).isEqualTo("Fizz");
    }
}

public class Rule {

    private List<Item> items;

    public Rule(final List<Integer> specialNumbers) {
        this.items = new ArrayList<>(3);
        this.items.add(new Item(specialNumbers.get(0), "Fizz"));
        this.items.add(new Item(specialNumbers.get(1), "Buzz"));
        this.items.add(new Item(specialNumbers.get(2), "Whizz"));
    }

    public String match(Integer number) {
        if (number.toString().contains(items.get(0).getNumber().toString())) {
            return items.get(0).getWord();
        }
        return items
                .stream()
                .filter(item -> isMultiple(number, item.getNumber()))
                .map(item -> item.getWord())
                .reduce((w1, w2) -> w1 + w2)
                .orElse(number.toString());
    }

    private boolean isMultiple(Integer divisor, Integer dividend) {
        return divisor % dividend == 0;
    }


    private class Item {
        private Integer number;
        private String word;

        public Item(Integer number, String word) {
            this.number = number;
            this.word = word;
        }

        public Integer getNumber() {
            return number;
        }

        public String getWord() {
            return word;
        }
    }
}
```

整个过程对原始的代码改动稍微有点大，不过在自动化测试的支撑下还是非常顺利地完成了重写任务，接下来可以进入重构环节识别代码中的"坏味道"，

```java
if (number.toString().contains(items.get(0).getNumber().toString())) {
    return items.get(0).getWord();
}
```

通过分析发现上面的判断条件表达的意图不太清晰，因此我通过重构手法 Extract Method 来提高代码的表达能力：

```java
public class Rule {
    ...
    public String match(Integer number) {
        if (isContainFirstSpecialNumber(number)) {
            return items.get(0).getWord();
        }
        return items
                .stream()
                .filter(item -> isMultiple(number, item.getNumber()))
                .map(item -> item.getWord())
                .reduce((w1, w2) -> w1 + w2)
                .orElse(number.toString());
    }

    private boolean isMultiple(Integer divisor, Integer dividend) {
        return divisor % dividend == 0;
    }

    private boolean isContainFirstSpecialNumber(Integer number) {
        if (number.toString().contains(items.get(0).getNumber().toString())) {
            return true;
        }
        return false;
    }
    
    ...
}
```

**执行单元测试保证所有的测试通过**

![2021-03-21-hsyV4n](https://image.ldbmcs.com/2021-03-21-hsyV4n.jpg)

## 7. TDD 成果

**任务清单：**

1. 生成 3 个不重复的个位数数字。
2. **!!!** 定义游戏规则。
   1. 如果是第一个特殊数字的倍数，就报 Fizz。
   2. 如果是第二个特殊数字的倍数，就报 Buzz。
   3. 如果是第三个特殊数字的倍数，就报 Whizz。
   4. 如果同时是多个特殊数字的倍数，需要按特殊数字的顺序把对应的单词拼接起来再报出，比如 FizzBuzz、BuzzWhizz、FizzBuzzWhizz。
   5. 如果包含第一个特殊数字，只报 Fizz **（忽略规则 1、2、3、4）**。
   6. 如果不是特殊数字的倍数，并且不包含第一个特殊数字，就报对应的序号。
   7. 重写 `Student` 类，使用 `Rule` 替换 `Student`。
3. 验证入参。

**测试报告：**

![2021-03-21-BNjlNk](https://image.ldbmcs.com/2021-03-21-BNjlNk.jpg)

**测试覆盖率：**

![2021-03-21-5a4yIB](https://image.ldbmcs.com/2021-03-21-5a4yIB.jpg)

## 8. 总结

坏消息是一开始因缺乏分析和设计留下来的坑迟早要填，好消息是通过上面的分析，这种问题是可以得到很大程度上的控制，先通过对问题进行分析和建模，再通过领域模型指导程序设计可以有效的降低错误设计的概率，在解决复杂问题域的时候效果更加明显，不过**需要注意的是 TDD 主张简单设计，在保证代码可用的前提下追求代码简洁，在重构中消除代码坏味道，并对原有的设计模型进行微观层面的演化和提炼，这种方式可以避免不同程度的浪费（设计浪费、不必要的重写、频繁重构和纠结等）**。

## 9. 阅读系列文章

- [测试驱动开发（TDD）总结——原理篇](https://mp.weixin.qq.com/s/CYHshxaMtffmHms91LExnA)
- [TDD 实践-FizzFuzzWhizz（一）](https://mp.weixin.qq.com/s/5aM-FrUN3dtcet_XfA8xQw)
- [TDD 实践-FizzFuzzWhizz（二）](https://mp.weixin.qq.com/s/nmF73LiMgAoQc39C2rQOHQ)

## 10. 源码

[github.com/lynings/tdd…](https://github.com/lynings/tdd-kata)

