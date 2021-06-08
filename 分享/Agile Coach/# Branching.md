# Branching(Feature branching your way to greatness)

Almost all version control systems today support branches–independent lines of work that stem from one central code base. Depending on your version control system, the main branch may be called mainline, default, or trunk. Developers can create their own branches from the main code line and work independently alongside it. 

今天几乎所有的版本控制系统都支持分支--来自中央代码库的独立的工作线。根据你的版本控制系统，主分支可能被称之为主线(mainline)分支，默认(default)分支，主干(trunk)分支。开发人员可以从主分支创建自己的分支并且独立运行。

## Why bother with branching?

为什么需要分支？

Branching allows teams of developers to easily collaborate inside of one central code base. When a developer creates a branch, the version control system creates a copy of the code base at that point in time. Changes to the branch don't affect other developers on the team. This is a good thing, obviously, because features under development can create instability, which would be highly disruptive if all work was happening on the main code line. But branches need not live in solitary confinement. Developers can easily pull down changes from other developers to collaborate on features and ensure their private branch doesn’t diverge too far from the main.

分支允许开发团队在一个中央代码库内轻松协作，当开发人员创建一个分支的时候，版本控制系统会在该时间点创建代码库的副本，对分支的更改不会影响团队中的其他开发人员。显然，这是一个很好的事情，因为正在开发的分支会影响系统的稳定性，如果所有的开发工作都在主分支上，对系统来说这将是非常具有破坏性的。但是分支并不会一直都孤独运行，开发人员可以轻松的从其他开发人员那拉取更改以在功能上进行协作并确保他们的私有分支并没有偏离主线分支太远。

> Branches aren't just good for feature work. Branches can insulate the team from important architectural changes like updating frameworks, common libraries, etc. -- PROTIP

> 分支并不仅仅适用于功能开发，分支可以将团队和重要架构变化隔离开来，比如更新框架，公共库等。--PROTIP

## Three branching strategies for agile teams

敏捷团队的三种分支策略

Branching models often differ between teams, and are the subject of much debate in the software community. One big theme is how much work should remain in a branch before getting merged back into main. 

分支模型通常因团队而异，并且是软件社区很多争论的主题。一个重要的主题是分支合并到主分支之前需要保留多少工作。

### Release branching

发布分支

Release branching refers to the idea that a release is contained entirely within a branch. This means that late in the development cycle, the release manager will create a branch from the main (e.g., “1.1 development branch”). All changes for the 1.1 release need to be applied twice: once to the 1.1 branch and then to the main code line. Working with two branches is extra work for the team and it's easy to forget to merge to both branches. Release branches can be unwieldy and hard to manage as many people are working on the same branch. We’ve all felt the pain of having to merge many different changes on one single branch. If you must do a release branch, create the branch as close to the actual release as possible. 

发布分支是指发布完全包含在分支中的想法，这意味着在开发周期后期，发布经理会从主分支创建一个分支（比如，1.1 development branch）。1.1版本的所有更改都需要应用2次：1次应用到1.1分支，然后应用到主分支。使用2个分支对团队来说是额外的工作，而且很容易忘记合并2个分支，发布分支可能很笨拙并且难以管理，因为许多人都在同一个分支上开发。我们都感受到了在一个分支上必须要合并许多不同更改的痛苦，如果你必须要创建一个发布分支，请创建尽可能接近实际的分支。

> WARNING: Release branching is an important part of supporting versioned software out in the market. A single product may have several release branches (e.g., 1.1, 1.2, 2.0) to support sustaining development. Keep in mind that changes in earlier versions (i.e., 1.1) may need to be merged to later release branches (i.e., 1.2, 2.0). Check out our webinar below to learn more about managing release branches with Git.

> 警告：发布分支是支持市场上版本化软件的一个重要的组成部分，单个产品可能有多个发布分支（比如：1.1，1.2，2.0）以支持持续开发（比较经典的就是Python），请记住，早期版本(即：1.1)可能需要合并到更高版本的分支中（即：1.2，2.0），查看下面的网络研讨会，了解更多的关于Git管理发布分支的更多信息。

### Feature branching

功能分支

Feature branches are often coupled with feature flags–"toggles" that enable or disable a feature within the product. That makes it easy to deploy code into main and control when the feature is activated, making it easy to initially deploy the code well before the feature is exposed to end-users. 

功能分支通常与功能切换（启用或禁用产品中的功能的切换）相结合，这使得在激活功能时可以轻松地把代码合并到主分支中，从而在功能暴露给最终用户之前，可以轻松地对代码进行部署。

> Another benefit of feature flags is that the code can remain within the build but inactive while it's in development. If something goes awry when the feature is enabled, a system admin can revert the feature flag and get back to a known good state rather than have to deploy a new build. -- PROTIP

> 功能切换的另外一个好处是代码可以保留在构建中，但在开发过程中保持未激活的状态，如果启用该功能时出现问题，系统管理员可以恢复该功能到已知的良好的状态，而不必部署新的版本。 -- PROTIP

### Task Branching

任务分支

At Atlassian, we focus on a branch-per-task workflow. Every organization has a natural way to break down work in individual tasks inside of an issue tracker, like Jira Software. Issues then becomes the team's central point of contact for that piece of work. Task branching, also known as issue branching, directly connects those issues with the source code. Each issue is implemented on its own branch with the issue key included in the branch name. It’s easy to see which code implements which issue: just look for the issue key in the branch name. With that level of transparency, it's easier to apply specific changes to main or any longer running legacy release branch.

在Atlassian，我们专注于每个任务的分支工作流。每个组织都有一个自然的方式来分解issue tracker内单个任务中的工作，像Jira Software，Issues随后会成为团队中在工作中的中心联络点。任务分支，也被称之为issue分支，直接将这些issue与代码连接到一起。每个issue都在自己的分支上实现，并且分支名称都包含issue key。很容易看出哪个代码实现哪个issue：只需要在分支名称中查找issue key即可。有了这种级别的透明度，可以很轻松地将特定更改实现在主分支或者任何不再运行的旧版本发布分支上。

Since agile centers around user stories, task branches pair well with agile development. Each user story (or bug fix) lives within its own branch, making it easy to see which issues are in progress and which are ready for release. For a deep-deep dive into task branching (sometimes called issue branching or branch-per-issue), grab some popcorn and check out the webinar recording below–one of our most popular ever. 

由于敏捷以用户故事为中心，任务分支与敏捷开发相得益彰，每个用户故事或者bug修复都存在于自己的分支中，可以很轻松的查看正在处理中的和已准备好发布的issue。要深入的了解任务分支（有时候被称之为issue branch 或 branch-per-issue），请拿上爆米花查看下面的我们有史以来最受欢迎的研讨会录音。

# Now meet branching's evil twin: the merge

现在来认识分支的邪恶双胞胎：合并

We’ve all endured the pain of trying to integrate multiple branches into one sensible solution. Traditionally, centralized version control systems like Subversion have made merging a very painful operation. But newer version control systems like Git and Mercurial take a different approach to tracking versions of files that live on different branches.

我们都忍受了尝试将多个分支合并到一个分支的合理解决方案痛苦，传统上，像Subversion这样集中式的版本控制系统使合并成为一项非常痛苦的操作。但是，像Git和Mercurial这样的较新的版本控制系统采取了不同的方式去跟踪不同分支上文件版本。

> With Git, merging is trivial–freeing us to exploit the full power of branching workflows.

> 使用Git，合并是微不足道的，因为我们可以充分利用分支工作流的强大功能。

Branches tend to be short-lived, making them easier to merge and more flexible across the code base. Between the ability to frequently and automatically merge branches as part of continuous integration (CI), and the fact that short-lived branches simply contain fewer changes, "merge hell" becomes is a thing of the past for teams using Git and Mercurial.

分支往往是短暂的，这使得他们在整个代码库中更容易合并和更加灵活。作为持续集成的一部分，频繁和自动合并分支的能力以及短暂分支仅包含较少更改的事实之间，对于使用Git和Mercurial的团队来说，”merge hell“已经成为过去。

That's what makes task branching so awesome! 

这就是任务分支如此棒的原因！

# Validate, validate, validate

验证，验证，验证

A version control system can only go so far in affecting the outcome of a merge. Automated testing and continuous integration are critical as well. Most CI servers can automatically put new branches under test, drastically reducing the number of "surprises" upon the final merge upstream and helping to keep the main code line stable.

一个版本控制系统只能影响合并的结果，自动化测试和持续集成同样非常重要。大多数CI服务器可以对新分支进行测试，从而大大减少最终合并时的”意外“的数量，有助于保持主代码的稳定。
