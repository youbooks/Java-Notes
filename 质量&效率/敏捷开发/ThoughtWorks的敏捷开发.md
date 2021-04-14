# ThoughtWorks的敏捷开发

> 转载：[ThoughtWorks的敏捷开发](https://insights.thoughtworks.cn/agile-development-thoughtworks/)

[ThoughtWorks的敏捷](https://insights.thoughtworks.cn/category/agile-practice/)开发方法一直是一种神秘存在。在敏捷开发还没有主流化的年代，为了让外界理解ThoughtWorks全球团队怎么做敏捷，我们商定了一个“60% Scrum + 40% XP”的经典答案。当然其实[ThoughtWorks](https://www.thoughtworks.com/)的敏捷开发既不是[Scrum](https://insights.thoughtworks.cn/why-your-scrum-failed/)，也不是XP。

造成这个状态的原因，一方面是行业特点 —— 软件开发还是一个充满不确定性的手工业，方法套路当然不可避免因人而异；另一方面作为一个提倡端到端软件交付的组织，敏捷开发本身并不能解决我们所有的问题。基于这两点，大家都不是特别愿意去总结ThoughtWorks的敏捷方法，换言之“标准化”就不在我们的基因里。

## 1. 标准化的初衷

那么为什么这个时间点来谈这件事情呢？从我个人角度来说对内对外都到了必须总结的时间点。在上一篇文章《[忘记“规模化敏捷”](https://insights.thoughtworks.cn/forget-agile-at-scale/)》中，我批判了市场上的[所谓规模化敏捷框架](https://insights.thoughtworks.cn/innovation-needs-edge/)，如空中楼阁，概念打包。但这样的现象确实也表明敏捷开发已经进入大规模采用阶段，一定的标准化是必须的。

ThoughtWorks快20年的敏捷开发实战经验不总结给更广大的社区，于我自己感觉是一种不负责任。对内ThoughtWorks中国已经从我加入时，北京东西宫每天流窜两趟招呼到所有人，变成了全国六地1,200多人的数字化服务公司。每年还是很多人为着敏捷慕名而来加入ThoughtWorks，而我能给新员工分享一点敏捷开发实践的机会可能一年一次都难了。

留在ThoughtWorks的一个重要原因就是差异永远大于共识，这样的环境里想总结点啥，没实战经验保底肯定是会被鄙视的。作为一个开发加入，在近十年的ThoughtWorks生涯里，我做过敏捷开发里除了测试以外的所有角色，如果把咨询的部分经验算成敏捷教练，和最近在努力的UX方向，应该是给了我足够全面的视角来审视ThoughtWorks的敏捷开发。在中国区持续最长时间的离岸敏捷交付团队近4年的经历，和敏捷咨询8年的经历也应该让我有一定经验支撑来谈这个话题。

## 2. 标准化的范围

即使有上面的经验，也只能聚焦在“开发”，而不是“ThoughtWorks敏捷”。针对市场探索和产品创新的方法仍然存在根本性认知上的不同，数字化时代的不确定性又让此刻去更大范围标准化的想法不切实际。但我认为敏捷开发 —— 即从开发团队启动交付到持续迭代运行 —— 已经为我们应对市场不确定性，构建高响应力组织，提供了基石，让我们能够在数字化时代将整个软件开发逐渐改进为真正的价值和成效驱动，而不仅仅是快速交付了一堆不知道有用没用的特性。

在下面的总结过程中有两个“技术处理”希望大家理解。

第一，软件开发手工业的属性造成了不同的团队成熟度显然是不同的，总结的ThoughtWorks敏捷开发实践并非所有团队都能够做到，但我强调的一点是所有团队都共识这些实践是有价值的，可能出于某种外部约束做不了，比如部门墙造成业务人员无法参与。

第二，尽量不引入ThoughtWorks自己的“黑话”，跟Scrum、Kanban和XP这些经典框架相似的实践，保持命名一致，毕竟标准化的作用之一是对外推广。

换上[咨询顾问](https://insights.thoughtworks.cn/consultant-teacher-coach/)的帽子，ThoughtWorks敏捷开发方法应该是现在市面上实践过程中最接近[敏捷宣言](https://insights.thoughtworks.cn/how-many-words-in-agile-manifesto/)价值主张的实战了。当然距离理想的价值和成效驱动的精益模式仍然有相当的距离，面临的挑战和困难可能不是敏捷开发能够解决的，但这些问题现在却反过来在压迫正确的敏捷开发方法，造成不少团队越来越多的困惑。当一个有追求的开发团队需要持续去解释[TDD](https://insights.thoughtworks.cn/talk-about-tdd-again/)和[Pair](https://insights.thoughtworks.cn/pair-programming/)不会降低效率时，技术卓越的追求就在逐步被消磨了。

## 3. ThoughtWorks敏捷开发核心原则

铺垫很长，希望尽可能在讨论范围上保持客观，现在让我们一起来看看ThoughtWorks敏捷开发模式。

为了帮助大家理解，我尝试从软件开发实践和管理体系两个维度去解释。先列举一下核心实践，然后从软件开发的几条管理主线帮助大家串联一下这台看似松散，实则精密的机器。这里要再次提醒大家我们讨论的范围仅仅是开发段，所列实践也不会特别关注团队文化建设。

在展开具体实践前，首先要明确ThoughtWorks敏捷开发的核心原则：[价值驱动](https://insights.thoughtworks.cn/redefine-customer-journey/)、技术卓越。

这两个四字短语在ThoughtWorks这个开发系统里有着不可撼动的地位。毕业刚加入的热血青年质问项目经理一个Story的价值何在？[Angluar](https://insights.thoughtworks.cn/angular-guide-for-backend-programmer/)和[React](https://insights.thoughtworks.cn/react-and-unit-testing/)谁更全面？这样的讨论在内部是被鼓励的。

这两个核心原则甚至上升到了价值观的层面，于是我们认为[好的客户一定能够“耐心”跟团队辩论](https://insights.thoughtworks.cn/think-as-a-customer/)价值，而让团队“听我的”业务人员基本只能维持在一个商务上的甲方。如果开发团队某晚上努力把Angular换成React，管理者（甚至客户）也被要求在强调风险管理的基础上，肯定团队为技术卓越追求付出的努力。

这对管理人员来说近乎是残忍的，这也是为什么ThoughtWorks在2010年左右经历了一批外聘PM的离职。虽然现在我们创造出了很大PM的管理空间，但值得警惕的是如果没有那些“恼人”的价值问题，和技术上的一点偏执，ThoughtWorks敏捷开发模式很可能就不存在了。

## 4. ThoughtWorks敏捷开发核心实践

在核心原则下，ThoughtWorks不同团队实践非常多，想要找到万变不离其中的骨干其实挺困难的。记得当年一家保险公司CIO带队来参观，看完早站会后直言不讳没有看懂一个50人的团队在干啥，只看到不同人群在自由组合。于是我花了一个早晨只为解释一小时过程背后的“隐次序”。

![2021-04-12-LbBeya](https://image.ldbmcs.com/2021-04-12-LbBeya.jpg)

（图：一个现场团队的早站会）

![2021-04-12-9UsDA0](https://image.ldbmcs.com/2021-04-12-9UsDA0.jpg)

（图：这是一个离岸团队的现场）

既然是核心实践，就从一个最小集开始，如果减掉我就认为不再是ThoughtWorks的敏捷开发模式。当然由于ThoughtWorks开发团队更多做的是互联网软件的开发，这个实践集并不一定适用于类似嵌入式设备和合规系统的软件开发。以下是我认为的集合，欢迎大家一起来研讨！

### 4.1 基于统一迭代节奏的全功能团队

刚加入ThoughtWorks的时候认识到开发团队除了开发和测试（当然这里我们认为是QA）外，还有BA —— [业务分析师](https://insights.thoughtworks.cn/business-analyst-in-thoughtworks/)。10年前这个角色在和任何客户合作的时候都需要解释，当然现在大家都已经在开始谈论UX和DevOps了。[全功能团队](https://insights.thoughtworks.cn/hk-build-full-function-team-practice/)这个实践本身并非是说要固定哪些角色在开发团队，而是强调为了交付软件所需要的技能都应该在一个团队里。在不远的将来，可能我们会再讨论Data Scientist的融入问题。

这个实践还要强调“[统一迭代节奏](https://insights.thoughtworks.cn/brainwashed-by-agile？/)”，要求团队各个角色同步协作，而不是每个角色自己迭代。我看见过太多的伪全功能团队，一个迭代开发完的Story由QA下一个迭代测试。

![2021-04-12-5krSeR](https://image.ldbmcs.com/2021-04-12-5krSeR.jpg)

（图：全功能团队跨职能协作示意，一个典型团队包含了BA、Dev、QA和UX。）

### 4.2 基于Story的需求及范围实时管理

Story是开发团队的最小工作单元，由于价值驱动的原则，Story INVEST原则是各个角色广泛认可的。如果哪个角色（包含业务）看不懂一个Story，那么大家会认为Story本身有问题。

ThoughtWorks敏捷开发不对Story进行更技术的Task拆分，这样做保证了大家都关注Story承载的业务价值，当然这需要技术能力上的“全栈”文化支持，即大家以能够同时做多个技术栈为荣。

运行成熟的ThoughtWorks开发团队有[Tasking](https://insights.thoughtworks.cn/pair-programming/)这个环节，形式可能是全队在[迭代启动](https://insights.thoughtworks.cn/agile-practices-in-project-management/)会上针对复杂Story进行“实现预演”，也可能是资深开发人员在自己显示器上贴出的彩色纸条，每张纸条承载着一个技术动作。[小虫](https://insights.thoughtworks.cn/author/zhangxiaochong/)和晶晶是我见过把这种Tasking深入工作骨髓的人，有幸看到他们基于这种Tasking模式的神pair是终生难忘的！

![2021-04-12-HsZ9NK](https://image.ldbmcs.com/2021-04-12-HsZ9NK.jpg)

虽然有整体项目的Backlog，但Story一般是迭代澄清，为了保证统一迭代，BA一般只会提前一个迭代梳理下一个迭代（类似Scrum中的Sprint）的需求。非常[成熟的ThoughtWorks开发团队](https://insights.thoughtworks.cn/a-story-of-delivery/)在这个过程中能够让客户或业务负责人持续迭代参与Story澄清，并能够持续调整Story[优先级](http://null/)。

国内由于固定合同项目较多，很多[需求澄清](https://insights.thoughtworks.cn/requirement-risk-strategy/)发生更早，但实际上很多人不理解保持一定并发性，正是驾驭软件不确定性的关键。“实时性” 是精益Just In Time能够起作用的核心机制。

范围管理上由于这样的Story迭代机制，基本也需要实时，常用的工具是燃起图（Burn-Up）和累积流量图（CFD来至于[Kanban](https://en.wikipedia.org/wiki/Kanban)）。Scrum的燃尽图并不推荐，原因是很容易营造一种遵循计划的假象。对于客户/业务和项目管理者，从燃起图能够看到实时需求范围的变化，按期交付风险也能够实时推测。累计流量图在成熟团队广泛应用，它能够直观告诉开发团队瓶颈在哪里，驱动改进。能够收集累计流量图所需的数据，本身也说明团队具备了一定的成熟度。

![2021-04-12-PMaXHX](https://image.ldbmcs.com/2021-04-12-PMaXHX.jpg)

![2021-04-12-dS1xDM](https://image.ldbmcs.com/2021-04-12-dS1xDM.jpg)

（图：燃起Burn-Up图示意，上图为一个最简单的统计展现，仅包含已完成和总共的Story个数。下图是一个相对长期和复杂产品，针对Story进行了类型划分的管理。注意这里的“完成”，包含了Story的分析、开发和测试，甚至一些团队Story [DoD](https://www.agilealliance.org/glossary/definition-of-done/)中要求上线。制造过程中的Story都是没有完成。）

行业里目前很关心这方面的电子化平台，ThoughtWorks由于历史原因，用各种平台都有，目前最多的是Jira、Mingle和Rally。但实际上这些平台主要目的还是为了[离岸敏捷交付团队](https://insights.thoughtworks.cn/something-about-offshore/)，本地的交付团队很多是物理墙+Excel（或[Trello](https://trello.com/)）。Story本身不作为审计和追责记录，真正的交付是线上工作的软件。

### 4.3 基于持续集成和测试前置的质量内建

[持续集成](https://insights.thoughtworks.cn/continuous-integration/)是敏捷实践中最广泛共识的技术实践（没有之一）。ThoughtWorks对持续集成的重视可以从历史经典的开源[CrusieControl](http://cruisecontrol.sourceforge.net/)窥见一斑。由于大显示器的普及和CI展示看板的美化，现在各个团队基本都采用一个显示器展示CI的实时构建情况，但历史上还是有很多类似警报灯这样的创意出现的。

![2021-04-12-0PHUzu](https://image.ldbmcs.com/2021-04-12-0PHUzu.jpg)

（图：为一个典型的团队CI看板展示）

![2021-04-12-uo2lWD](https://image.ldbmcs.com/2021-04-12-uo2lWD.jpg)

（图：看板一般所在的团队位置。）

ThoughtWorks持续集成纪律有两条核心，**第一是必须每次提交触发构建；第二是每次提交必须基于上次的成功构建。**这两条纪律是底线。如果有人说哪个团队CI红着没有修复，基本就等于说这个团队没有干活儿，因为理论上对着失败的构建提交代码是无效的。

持续集成对代码管理的要求是主干开发，这是ThoughtWorks开发团队的默认模式。去年和[刘冉](https://insights.thoughtworks.cn/author/liuran/)、覃宇通过《[代码管理核心技术及实践](https://baike.baidu.com/item/代码管理核心技术及实践/22293610?fr=aladdin)》一书阐述了行业内流行的分支开发模式实践和工具，但在ThoughtWorks内部，即使对使用比较广泛的[GitFlow](https://nvie.com/posts/a-successful-git-branching-model/)模式，也是[持负面意见居多](https://insights.thoughtworks.cn/gitflow-consider-harmful/)。显然[主干开发](https://insights.thoughtworks.cn/three-stages-agile-transformation-team/)的代码管理成本是最低的，但同时也引入了较高的代码实践能力和协同纪律的要求。

持续集成已经是软件开发过程中质量内建的经典实践，在这个基础上ThoughtWorks开发团队有共识的是测试前置，落地过程中有两个经典方法，即开发的TDD和提前验收（[Desk Check](https://insights.thoughtworks.cn/desk-check/)或叫Shoulder Check）。

TDD不用在这里多讲了，每年总会有一两次争论，第二个“D” —— Drvien —— 驱动，永远是大家争论的焦点，但先写[单元测试](https://insights.thoughtworks.cn/improve-the-effectiveness-of-unit-testing/)是ThoughtWorks程序员基本素养。[代码走查](https://insights.thoughtworks.cn/code-review/)形式多样，但成熟团队一般都从单元测试开始，如果你跟骨灰级程序员新老头走查，他会第一句问你“给我看看你的测试”。

提前验收操作起来是比较容易的，即开发人员完成Story后，在最后提交前邀请BA和QA快速在开发机上做展示。这样做的好处是尽量避免Story被移动到后期测试或客户验收的时候，才发现需求实现有问题，返工浪费。由于这个预验收时间很快，所以有些团队说是“站肩膀后面”检查。当然这个不是机械的规定，简单Story也不一定要做提前验收。

![2021-04-12-3444EG](https://image.ldbmcs.com/2021-04-12-3444EG.jpg)

（图：提前验收现场，Story开发人员快速讲解实现，现场客户也有可能参加到Story验收交流中。）

ThoughtWorks敏捷开发对回归测试考虑不多，[质量内建](https://insights.thoughtworks.cn/foucs-on-testing/)意味着不希望最后靠测试把质量关，相比之下，大家对代码走查和Pair这些开发过程中的质量实践更看重，但这些实践的频繁运用在很多团队都受到了成本的约束，并非普遍。关于分层的[自动化测试](https://insights.thoughtworks.cn/automated-testing-seven-years/)也是一个质量保障的核心话题，但随着技术的进步，认知在逐步清晰，我个人认为还不能说是非常成熟的基础方法。

![2021-04-12-rTEWLv](https://image.ldbmcs.com/2021-04-12-rTEWLv.jpg)

（图：一团队的代码集体走查现场）

![2021-04-12-NPin8T](https://image.ldbmcs.com/2021-04-12-NPin8T.jpg)

（图：开发人员的pair开发，往往会达到1+1 > 2的效率增长。）

### 4.4 基于Velocity和Cycle Time的持续改进

持续改进是每个团队必须做的，[回顾会议](https://insights.thoughtworks.cn/7-step-agenda-effective-retrospective/)（Retrospective）是基本形式，回顾的内容很多，从交付质量、实践，到团队氛围。但实质上最基本的改进还是围绕Velocity（即迭代交付的[Story点数](https://insights.thoughtworks.cn/story-point/)）和交付时间[Cycle Time](https://insights.thoughtworks.cn/performance-appraisal-the-wide-gap-of-agile-transition/)。这里的Cycle Time还称不上“端到端”，原因是很多团队前期的产品梳理和后期的用户验收还是遵循客户合作节奏，比如2个月一次上线。当然原则上共识的是，每次持续集成构建出来的版本都应该是可发布的。

Velocity是一个很有争议的话题，这个速率理论上只服务于项目管理，即目前规划和实际情况是否出现偏差，是否需要进行风险管理，调整项目范围等。

ThoughtWorks敏捷开发模式[坚决反对把Velocity作为交付KPI](http://jimhighsmith.com/velocity-is-killing-agility/)，即不作为迭代内的开发合同。假设合作上不存在信任问题，还是有一个无法回避的预测问题，如果Velocity和计划的偏差很大，那么实际的调整成本较高。当然这是一个行业问题，在“大规模手工打造”这个行业现状没有改观之前，最好还是能够坦诚开发过程中遇到的问题和变数。

Cycle Time是从需求进入开发团队，到制造出可工作软件的速度。理论上当然是越快越好，Kanban告诉我们流速快的团队效率高、响应力快。如果不注意Cycle Time去“改进”Velocity，很可能造成更多的[WIP](https://insights.thoughtworks.cn/self-organizing-all-round-team/)（Kanban引入的在制品概念）堆积在迭代内，最后大家赶工埋下质量隐患，得不偿失。所以我们可以看到在ThoughtWorks合作的两个百人以上规模的大型团队里，都强调团队集体学习Kanban实践。

![2021-04-12-Gz9QDa](https://image.ldbmcs.com/2021-04-12-Gz9QDa.jpg)

（图：一个团队三个半月的累计流量图，能够有效看到在某一个阶段的WIP和Cycle Time数值，并分析潜在问题。）

### 4.5 基于客户深度参与的统一团队

从ThoughtWorks历史的交付项目上来看，能够建立互信持久合作关系（>3年）的客户基本都深度参与了迭代开发过程。中国区历史最长，合作规模最大的三个交付项目都是客户团队和开发团队完全整合的。

客户参加迭代内[站会](https://insights.thoughtworks.cn/stand-up-meeting/)、展示会和回顾会是ThoughtWorks敏捷开发提出的要求。即使在通讯手段相对落后的七八年前，离岸的客户也是几乎每天和开发团队通过电话进行“站会”。这样做的好处不言而喻，大家都能够体会到这种模式下快速建立的[信任关系](https://insights.thoughtworks.cn/communication-of-offshore-team/)。当然客户和开发团队都有很多其它工作要做，所以这样的协作模式就要求控制每种会议的时间。一些比较普遍的原则有：

- 迭代站会不超过15分钟
- 需求澄清每次不超过1小时
- 展示会1小时以内
- 回顾会不超过两小时

虽然如此，这样的模式对客户时间要求依然很高。在咨询其它IT组织的过程中，相关业务人员（即开发团队客户）往往会有畏难情绪。但其实只要时间盒控制好，建立这样的协作节奏后，总投入时间是下降的。看似集中方式的合作模式，比如每个月1天时间需求梳理，其实根本没有办法杜绝后续实现过程中发现的细节问题。而到了迭代验收时才说出的“这不是我想要的”，更是巨大浪费的源头。

对比Scrum的经典四会，ThoughtWorks敏捷开发和最后的评审会（Sprint Review Meeting）的差异相对较大。开发团队更希望是针对实现的业务场景进行展示，所以会议的名字也变成了展示会（[Showcase](https://insights.thoughtworks.cn/agile-showcase-se7en/)）。当然不少团队也会评审迭代的产出，有相关的迭代进展报告。

![2021-04-12-z8cE78](https://image.ldbmcs.com/2021-04-12-z8cE78.jpg)

（图：一大型离岸合作客户将Showcase扩展到整个公司月度的跨产品展示。）

不少和国内客户合作的开发团队有相对更重一些的迭代总结材料，会占用PM不少的时间。这点上需要警惕，面向价值的原则意味着整个团队，包含客户，都应该更多去思考迭代实现的业务，而不是关注迭代大家的工作量。后者应该是开发团队自己去持续考量和改进。

## 5. ThoughtWorks敏捷开发管理体系

做了多年的组织转型咨询，如果约束到软件开发领域，管理体系（暂且认为文化部分属于领导力）基本就是以下四个方面：

- [需求管理](https://insights.thoughtworks.cn/tag/需求管理/)：包含从需求澄清到需求最终实现的整个生命周期。
- [技术管理](https://insights.thoughtworks.cn/tech-lead-is-the-service/)：包含开发、测试技术的选择和运用。
- [质量管理](https://insights.thoughtworks.cn/agility-of-qa/)：包含开发过程中的质量管理及软件交付前的质量保障。
- [迭代管理](https://insights.thoughtworks.cn/agile-practices-in-project-management/)：包含开发团队迭代运作规则及纪律。

显然ThoughtWorks敏捷开发需求管理是围绕Story展开，其核心是能够支持小批量、小批次的[精益模式](https://insights.thoughtworks.cn/talk-about-lean/)，同时还要能够尽量保证每个Story业务价值明确。《[用户故事与敏捷方法](https://book.douban.com/subject/4743056/)》这本书可能是ThoughtWorks内部没有任何负面评价的实践级著作了。近十年时间里，大家稍有微辞的地方可能是书中对故事大小评估的描述，但INVEST原则的抽象可为神来之笔。

Story作为需求的管理方法，所有的技术、质量和迭代管理其实都是围绕这个中心，毕竟最后开发目的是实现价值，而Story承载着业务价值。顺便提一句，Story的质量其实是一个核心问题，ThoughtWorks从来不提倡一句话Story描述，即仅仅表面上遵循了As … I want … So that的经典模式，验收条件对于一个Story来说至关重要。

值得一提的是围绕Story的可视化系统，每个团队都会有一面类似下图的迭代看板，看板上流动的是迭代内的Story，而Story的生命周期则通过顺序的泳道展现给团队所有人。

![2021-04-12-mIA0gg](https://image.ldbmcs.com/2021-04-12-mIA0gg.jpg)

（图：基础的Story迭代看板设计，黄色卡片上会写出Story的基本信息。）

![2021-04-12-GDewcE](https://image.ldbmcs.com/2021-04-12-GDewcE.jpg)

（图：一个团队的Story看板，每个团队都会有自己的内部流程设计，所以各个团队的看板泳道设计也不尽相同。）

技术和质量管理的核心仍然是前文提到的质量内建。持续集成和自动化测试实际上都将质量管理融入到了技术工程体系里。在ThoughtWorks敏捷开发体系里，很难将技术管理和质量管理分开，重过程质量是这个管理体系的精髓，由此也在2012年演进为[《持续交付》](https://book.douban.com/subject/6862062/)的概念。

由于是全功能团队，并工作在统一节奏下，所以迭代管理的范围中，除了类似Scrum四会协作仪式机制，其余硬性纪律较少。为了保证（成果和问题）“集体所有”（Collective Ownership），ThoughtWorks敏捷开发实践方面特别注意不会聚焦到个体，比如我们说到的Story估点和Velocity统计都是团队为单位，不会指定或统计到个人。

迭代过程中的缺陷也不会追述到某个特定开发人员。唯一产生个体比较和竞争的可能是在技术卓越原则下的比拼，比如作为TL，至少你写出的代码应该是让团队其他成员赏心悦目。

虽然本文不谈文化，但这里还是必须强调这样的管理模式本质上是将“价值驱动、技术卓越”上升到文化价值观层面作为支撑才得以实现的。即便经常拿[尚奇](https://insights.thoughtworks.cn/author/liushangqi/)口中的“tech@core”开玩笑，但实质上这是ThoughtWorks敏捷开发模式能够工作的重要底座。也是为什么很多其他团队感觉ThoughtWorks这种管理模式不可行的核心症结。

最后问一句，和你感受和认知的ThoughtWorks敏捷开发差异大吗？-；）

