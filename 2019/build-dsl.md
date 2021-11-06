# DSL 的设计

## 定义 DSL

### 起因 

受 Everything as Code 和 Domain Driven Design 思想的影响，我开始研究一系列的 DSL（Domain Specific Language）的抽象与实现。不论在过去的结论、还是在现在，我始终认识，DSL 是一个非常有用的东西。因为 Domain 即业务，它是源自于对业务的抽象。

代码本身便是在映射业务，而受开始者的受限，代码往往无法真实的反应业务。如没有统一业务和代码语言，导致落地的时候出现了二义性。正是这种二义性，导致了程序中出现的一个又一个的 Bug。

所以，在我们有了 DDD，有了  Clean Architecture，有了 core domain，有了 Domain 层，我们下一步应该是：通用编程语言无关的领域层。我们的业务核心代码，剥离了基础设施，剥离了框架依赖，它的下一步应该是剥离通用编程语言。我们设计的 DSL 应当可以直接反应出业务的实现逻辑。进而 DSL 变成一个业务人员可以读懂代码，甚至于可以直接编写相关的核心逻辑。比如采用 BDD 思想的 Cucumber 框架所采用的 DSL：

```
# language: zh-CN
功能: 失败的登录

  场景大纲: 失败的登录
    假设 当我在网站的首页
    当 输入用户名 <用户名>
    当 输入密码 <密码>
    当 提交登录信息
    那么 页面应该返回 "Error Page"

    例子:
      |用户名     |密码      |
      |'Jan1'    |'password'|
      |'Jan2'    |'password'|
```

（PS：我不想再写 BDD 的 E2E 测试）



### 定义

让我 Ctrl + C / Ctrl + V 一下我司 Martin Fowler 对于《领域特定语言》的定义：

书中对于 DSL 的定义有三种类型：

而在这篇文章中，我们主要讨论的是数据型 DSL。

## DSL 过程

我尝试去梳理最近抽象的几个 DSL 的过程，以从中获取一些灵感。

0. 领域名词
1. 关键抽象
2. 行为抽象
3. 场景代入
4. 实现 DSL
5. 迭代优化



这一点倒是与我最近所了解的遗留系统重构，有不少的相似之处。


## 汲取领域名词

对于一个新的领域来说，我们可能不需要这样的操作。

如我们之前对于，如对于一段代码来说，它存在以下的一些相关领域名词：

```dsl
Entity: Member
  VO: Name
  VO: Line Number
  VO: Data Struct ID
  VO: File ID
  VO: NameSpace / Package
  VOs: Position
    VO: Start
    VO: End
```

所以，我突然在想，是否可以在我那个 Coco 工具里，加一个这样的功能，以支持 NLP。【已支持】试试  `coca concept -p deps.json`

## Domain 分析与抽象


组件化中的原子设计：

`Component` + `Page`，

对于一个前端来说， Page 也是一种 Component，只是它除了名称之外，还有路由的唯一标识符。

## Behavior Abstract



## 关键抽象（Aha 时刻）

问题的本质

从 Google Scholar 又或者是寻找相关的概念，如我在设计  Design 的 DSL，看到了：

`See` - `Do` 对应了 `Given` - `When` - 'Then`

便想到了 React

结合

## 场景代入

开始一个新的项目时，我们采用了诸如 Event Storming（事件风暴）、Impact Mapping、实例化需求的方式，从需求和业务发现自源头开始梳理逻辑。



