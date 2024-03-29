# 第三个系统

最近，我刚好在和我的同事一起重写 VSCode 的一部分小功能，重写过程中有一些特定的感受。正好就着最近流行的操作系统话题，写写我的想法。因为某种协议的存在，不想为自己寻找麻烦，我这里就称为第三个系统。

> 系统是由一些相互联系、相互制约的若干组成部分结合而成的、具有特定功能的一个有机整体（集合）。这些要素可能是一些个体、元件、零件，也可能其本身就是一个系统（或称之为子系统）。

所以，如果是一类操作系统的话，那么以系统来定义为更加合适。

## 三个系统

提及到第三个系统的时候，我想到的是一本很不错的小册子（书），其名为《Linux/Unix 设计思想》。这本书主要是在讲 Linux/Unix 相关的哲学（实际上，我一直觉得这样的书很容易写，学好 Linux 和哲学，然后将哲学套到 Linux/Unix 上即可。唯一的难点是：跨领域知识）。

先简要地说说三个系统的定义：

1. 在背水一战的情况下，人类创建了『第一个系统』。PS：没有足够的时间将事情做好。
2. 『专家』使用『第一个系统』验证过的想法来创建『第三个系统』。PS：『第二个系统』由委员会设计，『第二个系统』臃肿而缓慢。
3. 『第三个系统』由那些为『第二个系统』所累的人们创建。PS：『第三个系统』结合了『第一个系统』和『第二个系统』的最佳特征。『第三个系统』的设计者有充裕的时间将任务做好。

结合之下来看，我们就会发现一些非常有意思的事情：

1. 充裕的时间，才能让我们完成一个更好的系统。
2. 『第二个系统』是拥有足够的专家和时间来完成的。
3. 『第三个系统』结合了『第一个系统』和『第二个系统』的最佳特征。

由上会产出一些有意思的推论：在有充足时间和资源的情况下，我们可能设计出的是 Windows Phone，巨硬（微软）的专家太多了。

## 新的专家

专家不论在哪里都是一种稀缺的资源，要不这个世界怎么会有咨询公司的存在呢。

开发一个操作系统并不困难。市面上已经有了各种琳琅满目的书籍，从《操作系统导论》到《自己动手写操作系统》、《30 天自制操作系统》，马上培训班就会出出《7 天自制操作系统》。

今天，我们基本已经达成了共识，开发一个系统的难点主要在于『生态』。为了生态，它可能要兼容一个系统的 API，这会导致系统臃肿。为了生态，它需要连带上下游一起丰富起来。为了生态，还需要开发各种各样的工具……

举个我们熟悉的 Android 系统为例，它的操作系统的源码（包含上下游工具）大概 120 G，它的开发工具 IDE 大概 60 G……。这个过程中涉及到大量的计算机相关的核心技术：编译器、虚拟机、操作系统、编译器优化、构建系统、图形编程……。就这么来说吧，它几乎快包含这一个领域需要的所有知识。而，你并没有时间预先的进行研究。

就构建来说，Android 系统因为大量的上下游，所以就需要：LLVM、Gradle、CMake、Bazel、GCC、Clang、Soong、Ninja……。而从编程语言上来看，所需要的语言知识有：Java、C++、C、Groovy、Kotlin。而除了这些，还有大量与硬件、芯片相关的知识。

因此，经此一役，这一世界又多了一个能造操作系统的国家。

## 复刻更难

最近，我和我的同事一起在使用 Rust 重写 VSCode 的词法分析工具。起先，我以为这是一件容易的事情，都是 TypeScript 嘛。写写测试，直接翻译就完了。然而，事实并非如此。

 - 需要深入理解原有逻辑。不断地调试旧系统的逻辑，并重新梳理思路。
 - 业务未剥离，导致大量耦合。语言之间存在用法上的差异，需要追溯用法上的差异和类型上的差异。哪怕是原文上写一个无用的 if-else，你都要纠结半天。更不用说，它可能有大量无用的代码。因此，我们需要寻找一种有效的方式来搞定，比如 TDD。
 - 语言交互接口（FFI）。Oniguruma 是我们所使用的正则库，而且还有指针的指针。
 - ……

相似的，对于一个复杂的系统来说，各种子系统之间的耦合度更是难于剖析 —— 需要大量不同领域的知识。每个问题不单纯只是某一语言、技术栈的问题，它往往是跨越了多个系统的问题。

## 结论

没有银弹。


