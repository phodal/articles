# 重构的自动化

> 任何傻瓜都能写计算机能理解的代码，优秀的程序员能够编写人能理解的代码。—— Martin Fowler


这些日子里，由于项目的缘故，我又双叕开始学着造轮子了。故事的开始是代码的不规范堆砌，导致软件大楼摇摇欲坠；故事的终点是，重新唤醒程序员对匠艺的追求。而故事的中间部分，则是我们所要关注的内容：代码坏味道（code smell）、包依赖合理性，应对的方案则是代码重构，目标则是  Clean Code，即易于阅读的代码。而它们（代码坏味道、重构方式等）都已经被归纳为模式。

而所谓的模式指的是：模式，在物体或事件上，产生的一种规律变化与自我重复的样式之过程 [^pattern]。既然，它们是可重复的，那么也存在一定的自动化可能性。自动化解决重复性问题，是程序员的使命之一。

[^pattern]: https://zh.wikipedia.org/zh-hans/%E6%A8%A1%E5%BC%8F

程序员的另外一个使命是造轮子。不求造出完美的轮子，只要是可工作的，并且不断改进地轮子，都终将是一个好轮子。

扯完了，回到正题。

## 架构的混乱

好吧，这个小节的内容是额外多出来的。至少在当前，我还没有想到好的方式，来通过自动化的方式，来识别出一个架构是否出现问题了。如果有的话，那么一定是：

1. 代码行数，反应大泥球。如果**单个**后端服务能达到 1000 * 1000 行级别的代码量，那说明它将变得难以维护——添加新功能缓慢，依赖更新受限等。
2. 代码复杂性。一个最简单的指标就是单一文件的圈复杂度，继承深度等等。
3. 服务过多，反应拆分不合理。诸如于，微服务数量接近于开发人员数量，导致人人疲于奔命。
4. 服务调用流程化，反应拆分不合理。通过 API 间的调用顺序，来反应架构上的问题。
5. 技术导向服务划分，反应拆分不理。识别公共的服务调用，来确认合理性。
6. RESTful API 规范化。可以对进行 API 扫描查看命名，自动化模拟测试查看 API 异常情况返回结果。
7. 服务 API 与数据库映射。对于服务而言，API 与数据库结构能反应其是否独立独在的必要。

好吧，我编不下去了，因为大部分用肉眼就能看出来的了。而若是自动化，则能很大程度上降低人力成本。

而应对于架构拆分的模式，应对的方式无非就是：**拆与合**。

### 工程的规范化扫描

一个良好的软件团队，必然拥有强大的工程能力。正是这些工程能力，让每个新加入的成员，能在约束的情况下快速上手。所以，我们可以在不阅读代码的时候，了解到详细的情况。
 
 - CheckStyle（or Lint） 是否集成？通过检查对应语言的配置
 - SCM（如 Git）commit message 是否规范？通过提交记录来学习模式，而后可以通过类似于 ``commitizen`` 的工具实现自动化。
 - SCM（如 Git）的分支策略是否合理？扫描出所有的分支，了解对应的分支策略。
 - 适应度函数检测。诸如于是否包含对应的测试策略——可以检测的测试工具、度量覆盖率，来判定是否拥有有效的测试策略。
 - 依赖版本滞后扫描。

这显然不是一件容易的事，它涉及到一系列的整体性方案。

## 依赖关系的打破

对于遗留代码来说，我们所做的主要工作是：**解除依赖性**。而要打破依赖并不是一件容易的事，它所涉及的是一层层的函数调用，以及其背后的复杂业务逻辑。只需要构建出目标语言的 AST（抽象语法树），我们便能分析出项目中的各个包间的依赖关系。

然而，即便我们知道了两个类、包、服务之间出现循环引用，要自动化打破这些依赖也不是一句容易的事，因为处理的规则太多了。诸如于：

 - 合并类、包、服务
 - 重新划分职责
 - 引入中间人（委托）
 - ……

所以，可视化依赖关系，在当前而言，是一个不错的选择。我们所要做的便是：

1. 自动化确认出每个文件的依赖
2. 构建依赖关系
3. 生成依赖关系图
4. 标注循环依赖的部分

只是呢，就我当前的经验而言，还无法做到这部分的完全自动化。

## 坏味道的识别

如果代码有味道的话，那么你所能闻到的一定是坏的那种，因为『久居兰室不闻其香』，对于稍有经验的开发人员来说，寻找代码的坏味道，是一件简单的事情——从代码里挑刺嘛。当然了，我不得不再说一下，挑刺很容易，但是难的是给出**改正手段**——即如何重构。

步骤是：

 - 构建特定语言的语法解析器。
 - 设定坏味道的内容及标准。如圈复杂度、嵌套深度、继承深度等。
 - 针对于每一项坏味道，编写识别代码。
 - 组织特定的坏味道识别。
 - 编写坏味道的建议改进和实施代码。

而对于这些坏味道来说，有的易于识别，有的没必要识别：

![重构分类](images/refactor-map-easy.png)

对应的也可以按分组来判断各自的需要。

| 分组 | 坏味道 |
|-------|-----------------|
| Bloaters | Long Method, Large Class, Primitive Obsession, Long Parameter List, Data Clumps |
| Object-Orientation abusers | Switch Statements, Temporary Field, Refused Bequest, Alternative Classes with Different Interfaces |
| Change Preventers | Divergent Change, Shotgun Surgery, Parallel InheritanceHierarchies |
| Dispensables | Lazy class, Data class, Duplicate Code, Dead Code, Speculative Generality |
| Couplers | Feature Envy, Inappropriate Intimacy, Message Chains, Middle Man|

除此，还有一些不在列表里的坏味道，比如：

 - 奇怪的命名。命名很难，但是它不应该是：ZLSYActivity、FraActivity123Manger。
 - 害怕出现 bug 的重复。比如 FileManager1 和 FileManager2
 - 大量的静态方法。new 一个实例，到不了 10w - 100w 以上的量级，是不会有太大的性能差别的。[^refactor_oo]

[^refactor_oo]: https://www.iteye.com/blog/wang-zhenyu-49107

所以，如果能对坏味道进行自动化，就能减少了多人力工作。

当然了，使用 SonarQube、CheckStyle、FindBugs 和 PMD 等也能找到对应的 Code Smells，只是呢，大部分情况下，我们遇到的问题要比这复杂得多。而实践证明，那些“机智”（鸡贼）的开发人员，已经有各种办法绕过这些措施。

## 测试的快速反馈

自动化的测试代码编写，依赖于大量的先验知识。如果我们计划于生成某个函数的测试，那么我们首先必然需要调用这个函数，才能返回预期的结果。而测试的完整性，实质上是依赖于边界条件来构建的。如一个对字符串处理的函数的测试，我们需要：

 - 传入各种的特殊字符，以检验是否能返回正常的结果。
 - 测试条件分支，以保证异常情况的正常性。
 - 测试边界条件，以防止出现意外。

当然了，这种自动化的测试，多数时候都不是那么可靠的。不过，是一个非常不错的 Monkey 测试。

为了完成这部分的测试生成，我们需要：

 - 构建运行时函数调用，或者依赖于 AST 解析 而后计算（这个成本有点高）
 - 强大的测试框架支持。因为代码人人会写……，但是好的代码不是。
 - 丰富的测试编写经验。
 - 大量的测试模式数据。
 - 生成人类可阅读的代码。
 - 框架无关的测试 DSL。

每个都不是一件容易的事，都需要大量的经验。当然了，要编写这样的一个应用进行自动化更不容易。

## 代码重构中的模式

开始之前，让我们先统一一下语言：『什么是重构』

> 重构（refactoring）是指在不改变外在行为的前提下，对代码做出修改，以改进程序的内部结构。

所以，只要改变了现有的行为，都不能称之为重构。按习惯，我们将代码重构分为了三类：

 - 自动化重构。就目前而言，自动化重构是一件非常难的事情，因为有些代码人都看不懂，机器就回天乏术了。
 - IDE 重构。依赖于 IDE （如 IDEA）提供的重构功能，能在不影响功能的情况下，快速进行重构。
 - 手动重构。

对于手动重构来说，我们首先依赖于先前识别的 code smell（代码坏问题）。而在重构一书中，给出了每一种坏问题的对应修改方法。如应对于**被拒绝的遗赠（Refused Bequest）**，推荐了三种对应的修改方法：

 - Push Down Field（下移字段）
 - Push Down Method（下移方法）
 - Replace Subclass With Delegate（使用委托代替子类）
 - Replace Superclass With Delegate（使用委托代替超类）

其中的 3 和 4 在《重构1》中合称为：Replace Inheritance with Delegation（以委托取代继承）。 

基于此，我们就可以拥有一套完整的端到端重构工具集。

## 结论

有没有这样的工具呢？

有。 GitHub：https://github.com/phodal/coca

Coca 是一个使用 Go 编写的 Java 语言自动化重构工具。架构方面能生成对应的依赖树，重构方面支持批量重命名方法、寻找未使用的类、删除未使用的 import、移动类到指定目录等。即将支持本文中的剩下功能。


