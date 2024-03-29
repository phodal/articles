# 『头破血流』学编程语言（Rust 篇）

学习 Rust 已经有一段时间了，断断续续地在业余时间造了一些轮子。通过这一系列的练习和仿造，对于如何学习新的编程语言有一些新的感悟。这篇文章讲的方式并非是捷径，也不是什么 7 天精通，而是继续使用笨办法地方式来进行学习。

关于编程语言学习，我已经写过两篇相关的文章：

 - 『[学习的艺术——如何学好一门技术、语言](https://www.phodal.com/blog/how-to-learn-language-technology/)』，文章的主旨是：介绍如何通过造相似的轮子、进行相关内容输出的方式，来提升对于编程语言的理解。
 - 『[如何同时学会两门编程语言？](https://www.phodal.com/blog/how-to-learn-two-languages/)』，介绍的则是用硬核的方法：造语法、词法解析的方式，来掌握新的编程语言。同时，如果我们是对新的编程语言的解析，那么我们就等于学习了两门新的语言。

从我的角度来看，前者的介绍过于简单，只是告诉了你应该这么做，但是没有说要怎么做。而后者则难度太大，对于大部分的人来说，几乎是不会想着去做这样的事情。而本文的难度呢，刚好介于两者之间，至于是不是中间嘛，也不好说。难度，因人而异，因时间也有区别。

对于编程和计算机理解越来越深刻，那么原先难度适中的事情，因为做过会变得更加简单；而原先复杂的事情，如果我们还没做过，那么我们可能还觉得它依然相当的复杂。

## 为什么学习新的编程语言？

工作多年，我们依然会和同事、朋友讨论到：业务是永恒的，技术是永远在变的。所以，成为一个业务专家更容易、更持续，成为一个技术专家更难、更需要持续提升。选择很难，因为我们不是火星人，也没有上帝视角。所以，成为一个技术上的专家，我们需要不断地接触一些新的东西，接受一些新的概念。其中的一种模式便是，人们口中经常说的：**每年学习一门新的语言**。

从个人的角度来看，这是一个非常 SMART （具体、可度量、可实现、相关性、有时限）的目标。所以，它还会存在这么一些优点：

1. 保持学习的习惯。
2. 为技术热情添到香油。
3. 学习不同的编程模式。
4. 拓展职业机会和前景。

除此，从职业鸡汤上来说，就是：机会是留给有机会的人。如果你学习了一门新的编程语言，那么未来有相关的机会，你更有可能触摸到。

若是你将学习新的编程语言，视为非常火热的什么内卷化、奋斗逼之类的说辞，那么我倒是没啥好说的。有的人是真的在 “奋斗”，有的人是想了解各种有意思的东西。从我的角度来看，学习新的编程语言和上述的说辞是不存在关系的 —— 不存在竞争，只是加一条赛道，笑。

## 寻找语言学习的高效路径

在上文中，我提及的第一篇文章《[学习的艺术——如何学好一门技术、语言](https://www.phodal.com/blog/how-to-learn-language-technology/)》在今天对于我来说，已经是一个相当浪费时间的事情 —— 重复劳动。文中提到的方法，无非是造重复的轮子、重写旧的应用，这种方式和诸如在 30 天里去练习不同的项目，都只是在特定的场景之下，出于特定的目标而练习的垃圾产物。随着我们的成长，生活和工作上的一些事情，会占据我们更多的时间。尽管，我尚未被这些问题困扰着，但是我已经有了一个又一个的方案。不过，我相信你们都会有这些问题。

简单来说，我们需要即学好一门编程语言，又不重复劳动。所以，可行的方式是学习新的语言，并在新的编程语言里寻找新的轮子。诸如于《『[如何同时学会两门编程语言？](https://www.phodal.com/blog/how-to-learn-two-languages/)』》就是一种不错的方式，但是对于多数的人来说，它有点难。不过，从个人的角度来看，如果你是选择从一个 XML 解析、JSON 解析开始的话，可能就没有那么难。但是，就是在重复的造轮子。

这么一圈废话下来，其实我们的结论就是：在语言的适合场景下，造适合的轮子 —— 这可能意味着一定的时间成本。比如，用 JavaScript 来处理非关系型数据，用 Go 来开发跨平台命令行工具，用 Rust 来开发 WASM 应用等等。

### 高效路径

在我尝试了一系列的造轮子工作之后，我有了一个初版的模型（基于 Rust 语言）。我暂时划分了四条路径：

1. 工程实施。即使用该语言时，开发应用时需要哪些实践。
2. 应用开发。理解完整的开发应用所需要的知识体系。
3. 框架设计。使用该语言如何进行各种抽象设计。
4. 语言练习。要么用它来写语法解析，要么来解析这门语言。
5. 领域特定编程/场景编程。即寻找适合这门语言的场景。

作为初版，这条路径可能不一定能 match 上你的需要，但是随着我们不断也提升，我们终将能形成一个更完整的路径。

## 工程实施

从工程实施，这个角度来看，我们所要掌握的是一些基本的编程能力：

1. 自动化测试。诸如于单元测试、集成测试等等，以帮助我们开发出高质量的应用，并节省 debug 的时间。
	 - TDD（测试驱动开发）。同上。从个人的角度来看，若是掌握 TDD 这一项技能，可以编写高质量的代码。
	 - 测试覆盖率。
2. 持续集成。真实的软件开发需要持续集成，这也是我们学习编程语言时，要掌握的工程技能。
3. 构建管理。寻找适合于这门语言的构建体系，以帮助我们构建出可信的软件。

如我们在使用 Rust 开发应用时，就可以使用 GitHub + Travis CI 的方式完成对于持续集成的了解；结合 Justfile/Makefile 等，完成自动化的构建。

## 应用开发

应用开发是基于真实项目的角度出发，来完成对于语言的练习。这些内容包含了：

1. 自动化部署。主要用于学习在真实项目下，如何提交效率。
	 - 容器化部署。
2. 分层架构。如何合理的划分项目的目标结构，常见的方式有两种：
	 - MVC 架构。传统的三层架构
	 - 整洁架构。基于抽象的形式设计的架构
3. DevOps 体系。根据需要，完成从需求到上线流程的支持。如：
   - 应用性能监控。
   - 日志。

不同的语言之间，或许存在一些差异，但是从最终的情况来看，它们都需要提供一致性的接口，或者是采用一致性的接口。如对于数据库的访问，使用的接口是一致的；提供 RESTful API，其对于消费者来说，也需要提供一致地 API。

## 框架设计

框架设计是基于造轮子的需求场景下的路径。它包含了：

1. 抽象。语言如何进行抽象
   - 支持 OO。
   - 不支持 OO。如何使用诸如于 Rust Trait 完成类似的工作
2. 语言无关。如何进行跨语言的设计支持。如：
   - 语言无关的数据格式。
3. 模块化开发。如何完成跨团队、跨业务模块的代码、服务共享。
	 - 包管理/依赖管理。如如何构建，并发布到制品仓库，实现复用。

框架设计从理论上来说是稍微复杂一些。至于有没有必要，就看你想学习到什么程度了。

## 语言练习

语言练习是《[如何同时学会两门编程语言？](https://www.phodal.com/blog/how-to-learn-two-languages/)》模式之下的一种路径方式，相对会陡峭一下。

1. 编写其它语言/DSL 的解析器。
2. 使用其它语言编写该语言的解析器。
3. 使用该语言解析该语言。

嗯，是不是有点意思了。从场景上来说，当我们拿到了一个语言的 AST，然后就可以尝试去做一些高端的事情。如我在 Coca 里做的自动化重构、架构可视化等等。

## 领域特定编程

领域特定编程是在该语言擅长的场景下，做该语言擅长的事情。如 Rust 里的

1. 跨平台
2. WASM
	- 一门应用跨端运行
3. 系统级编程
	 - 结合系统接口，如获取用户输入，并修改输出。

这依赖于我们识别场景，并知晓出什么时候才是合适的场合。

## 其它 

没有银弹，如果有的话，那就不需要人类了。


