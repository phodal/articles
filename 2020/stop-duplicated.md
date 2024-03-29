# 停止复用

> 最大化重用会使得可用复杂化。 —— 《Java 应用架构设计》

这个标题有点标题党了，但是我觉得你能理解：为什么我会用这么一个吓唬人的标题？

文章起源于我对于模块化、微服务、Serverless 以及单体应用几种不同的架构模式的思考。而这其中的一个原因就是：人们经常从一个极端走另外一个极端。既然单体不好，那么我们就要 FAAS 来替换单体；既然模块化架构有各种问题，那么我们应该回到大单体。

## 微架构

> 微架构是一系列架构风格的总称，通过构建一系列责任单一的小型应用，再组合出一个复杂的大型应用系统。受限于不同的运行环境，它还具备有不同的特性。

对于运行操作系统的微架构来说，它具备了语言无关的特性，如微服务；对于运行在浏览器的微架构来说，它具备了易于集成遗留系统的优势，如微前端；对于运行在移动操作系统虚拟机环境的微架构来说，它具备了应用可嵌入式的特质，如插件化。

微架构具体了一系列的好处：

 - 适应组织架构。即康威定律
 - 实现更多的价值。提升开发团队的能力水平、组织级的 DevOps 提升；
 - 降低开发复杂性。解决单体应用边界不清晰的问题
 - 改造遗留系统。采用绞杀者模式逐步重写；或者直接封装 API 不再添加新功能；
 - 减缓架构腐化。

从某种意义上来说，它符合人们所宣称的几点特性：可部署、可管理、可重用、可组合。

反过来，我们可以思考如果不在这几种模式下的微架构是否合理？比如说，单一团队维度大量的微服务、组件，不合理的技术划分模块和服务……

而实现上最容易出现微架构失效的原因就是：复用。

## 复用及其粒度

> 如果模块过于细粒度和轻量级，那么我们将面临模块和上下文依赖的爆炸。

复用可能导致的最大问题是，我们划分的所有边界都失效了。原本我们通过组织架构、变更频率、权限等划分的服务边界，都在此时失去了作用。

### 代码复用危机

过去，我们的复用方式是，模块复用、包复用，整个系统间的关系相当的混乱。明明我们只是依赖于一个内部包里的几个函数 ，它们可能就是一两百行的代码，然而我们要引入一个几 M 的包。出于同样的目的，这个包又一次被其它系统使用了。

随后，我们因为自己的需求，改动了一次代码，发现其它几个系统都出了问题，便陷入了无穷无尽的开会和协调中。随后，我们越来越倾向于不使用这个包，也不去改动这个包，这个包变成了一个**遗留的包** —— 没有人敢改，没有人想改。一直持续到其它项目重写的那一天，这个包对于每个系统的人来说都是一个包袱。

而这一切可能只是我们调了其中的一个方法，复制过来 1 分钟的事，却要花费几天的时间去解决。

于是，我们在帮助企业进行遗留系统重构和改造时，不得不依赖于 tequila 和 coca 这样的工具，来完成对架构的可视化。而现在，我们开始考虑了微服务的复用。

### 微服务复用危机

同样的，明明只是一个简单的功能、API。故事是很相似的，我们选择了已有的其它团队的接口，这样一来就不用花费时间写这个接口了。一切都很完美。

然而，因为 A 团队的业务发生了一些调整，导致了接口的返回内容和我们预期的不一致。现在出现了预期之外的 bug，这个 bug 打乱了我们的节奏 —— 除了修复这个 bug，我们还要开会讨论后续的接口变更问题，还有各种的扯皮。

双方开始陷入了这场复用的恶梦中。

## 重复优于耦合

重用和复杂性是成正比的，越是关注于重用，则系统的复杂度会进一步上升。

有些人讨厌设计模式的原因，在于滥用设计模式的人太多了。而设计模式是真的没用吗？它是一些解决问题的模式，只能套用在合适的问题上。而且随着架构的变化，原有的模式可能不再适用。所以，我们总可能觉得前人用错了模式，设计错了架构。

也因此 DRY （Don't Repeat Yourself）这一原则并不总是适用的。反而，你可以认为呢，它只在少数的时候是适用的。

## 组合优于继承

> 继承是实现代码复用的有力手段，但它并非永远是完成这项工作的的最佳工具。

这里的组合指的不单单是代码组织级的组合，对于微服务级别也是如此。

## 建立防腐层

嗯，就这么简单。

对于代码级来说，我们复用三方接口时，一旦三方接口发生过变化，封装便是我们的防腐方式。

对于微服务来说，我们复用三方接口时，一旦三方接口发生过变化，封装、BFF 便是我们的防腐方式

## 最佳实践

哦，它不是免费的 —— 因为不会有最佳实实践。

不过，我觉得你可以试着做这些事：

 - **提升重构能力**。当它们真的重复的时候，你就可以提取公共的部分；当它们真的不重复的时候，你可以内联回去，再提取新的组件、函数、服务。
 - **避免过度设计**。我的意思是，不要在设计阶段，过于草率地决定：它们就应该复用。
 - **阶段性检视**。当复用带来复杂度时，重新梳理一下问题发生的原因。

事实上，看看标题就够了。

