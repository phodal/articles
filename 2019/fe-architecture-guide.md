# 前端架构守护

你是不是经常烦恼于放错层级的代码？你是不是经常在为一坨坨代码而奋斗？你是不是经常这样？

在过去的日子里，如我在那篇『如何提升 Web 应用的代码质量』所提及，我们采用了一系列的措施 如 pre-push, pre-commit 在基础层级对架构进行守护。但是，这样仍然是远远不够的。

因为在『架构金字塔』里，这些基础的规范、原则等只是沧海一粟，往往无助于对软件架构的辅助设计作用。所以，我们需要架构守护这样的工具。

## 从 ArchUnit 说起

架构守护是在过去的两三年里，新起的一个话题。因为人们需要在：

 - **系统演进后架构守护，减缓系统再次腐化**
 - **应用架构适应度函数（Architectural fitness function）来驱动架构演进**

于是乎，在 Java 世界里，越来越多的项目采用了 ArchUnit 这样的工具来防止架构的腐烂。

> ArchUnit 是一个基于 Java 的测试库，用于检查代码的结构特性，如包和类的依赖关系、注解验证，甚至还能检查代码分层是否一致。

你可以通过如下的测试，来确保某些类型的文件放置于指定的包名 & 文件名下：

```java
    @ArchTest
    static final ArchRule layer_dependencies_are_respected_with_exception = layeredArchitecture()

            .layer("Controllers").definedBy("com.tngtech.archunit.example.layers.controller..")
            .layer("Services").definedBy("com.tngtech.archunit.example.layers.service..")
            .layer("Persistence").definedBy("com.tngtech.archunit.example.layers.persistence..")

            .whereLayer("Controllers").mayNotBeAccessedByAnyLayer()
            .whereLayer("Services").mayOnlyBeAccessedByLayers("Controllers")
            .whereLayer("Persistence").mayOnlyBeAccessedByLayers("Services")

            .ignoreDependency(SomeMediator.class, ServiceViolatingLayerRules.class);
```

并通过诸如于 ``mayNotBeAccessedByAnyLayer`` 或者 ``mayOnlyBeAccessedByLayers`` 来限制它们的层级调用。一旦出现了不合理的调用，如 Controller 层，直接调用了 Persistence 层，那么测试就会失败。

PS：上述的代码只是其众多功能中的一部分。

自了解了相关的概念之后，我开始思考如何在前端去做这样的事情。

## 架构守护

> 所谓的架构守护，是**以测试的方式，来防止架构模式的抽象被破坏**。

当我在项目上实施了整洁架构，我便大致知道如何去实现这样的工具。因为所谓的架构守护，实际上便是**以测试的方式，来防止架构模式的抽象被破坏**。即，我们在设计架构的时候，会制定出一些通用的规则，诸如于函数的命名方式、文件的命名规则等等；与此同时，我们还会通过分层架构来限制调用，而分层架构是否合理的体现，便可以通过依赖的方式来分析，等等。

从我的角度来看，理想的架构守护应该要有这样的特征：

 * 无需编写的测试
 * 框架无关
 * 自定义的架构范式

### 无需编写的测试

测试一直都是最好的代码质量/业务守护方式。但是，你知道的大清自有国情在此——大家测试写得少。所以从实践的意义上来说，自动化的无需写代码的测试，其实是最好的测试。

PS：我在上一个契约测试框架 mest 中，采用的也是这种模式，详细内容见：『[前后端分离：使用 mest 做契约测试跟踪 API 接口变更](https://www.phodal.com/blog/frontend-contract-test-use-mest-way/)』。

笑，也许架构守护，能让更多的人对测试心动。

### 框架无关

嗯，是的，特别是对于前端来说，不同的前端框架采用不同的架构模式。这些抽象需要由测试框架来把握。

### 自定义的架构范式

大家都懂的

## 前端架构守护框架：Dilay

于是乎，我造了个简单的轮子， 前端架构守护框架 Dilay——当前只支持 TypeScript 编写的应用。它的使用方式很简单：

```
npm install -g dilay
```

然后，运行下面的命令就 O 了：

```
dilay -p path/afsdf
```

它的基本原理是：

1. 查找目录下的 package.json，并通过它来识别框架
2. 读取 .gitignore 和 src 来进行代码扫描
3. 通过目录与文件名的映射关系，来判断是否合理。如 utils 与 time.util.ts 的关系
4. 使用 TypeScript 编译器解析出依赖树，再对依赖进行分析

对，就是这么简单。

## 结论

这只是个 Demo？

不，这并不是一个 Demo，它是一个 PoC（概念证明），笑。

欢迎基于 Dialy 创建你的前端架构守护工具。我的意思是，这个坑我好像填不完了，但是你完全可以基于 Dilay 开发你的工具。

架构守护，会驱动出更好的架构吗？不会。


