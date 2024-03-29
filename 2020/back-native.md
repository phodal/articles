# 六年之后：回到底层编程

一个月以前，我加入到 ThoughtWorks 已经进入了第 7 个年头。我本来不想再花时间写一篇相关的文章。只是呢，最近发生的一些事情，或许和每个 IT 人息息相关。我便想着，再花点时间思考一下：我们的行业以及行业背后的 IT 体系。

从某种程度上来说，我更可能是在弥补一块重要的短板 —— 一来，不是从事相关的领域；二来，平时的开发中真的用得少。

## Web 开发的核心：CRUD？

在我先前的文章《[项目初期的最优技术实践](https://www.phodal.com/blog/short-time-project-best-practise/)》里，我把产品开发的生命周期分为了：

1. 技术准备期
2. 业务回补期
3. 成长优化期

从技术维度来看待的话，我们会发现随着时间的推移，整个项目对于人能力的要求会越来越低，直至大家都变成 CRUD、Copy/Paste 程序员。唯一经常还被提及的难点就是：对于数据库的各种优化，又或者是对于遗留系统的迁移，又或者是对于某个开源软件的优化。

所以啊，我们听过别人提及一个明确的大公司技术发展路线：成为一个技术 + 业务专家。回过头来看，事实也是如此 —— 在大部分的公司里，成为一个技术专家，是一个投入大收益低的事情。对于个人来说，不一定有足够的回报，不一定有用武之地，不一定能找到相匹配的工作；对于组织来说，难以创造更高的价值。

可是呢？对于多数人来说，他们真的还是技术专家吗？如果不写代码的话，他们只是一个**技术决策者**，而技术决策者和技术专家是两种不同的发展路径，只是他们做的某些事情是重叠的。

所以，在 CRUD 之外，我们的一个侧重点还在于对基础设施的优化，而这些基础设施大多数是开源的。

## 开源是否在扼杀技术创造力？

自由软件和开源软件运动，解放了我们在整个行业的生产力。过去，我们关注于如何实现某个功能，现在我们可以转而关注于某个业务的具体实现。

不可避免地，在国内的一些大公司里，受过训练的、编程能力极强的程序员，变成了一个会写代码的业务专家。他们带领着大量的**平均水平**一般的外包程序员，一起编写着业务价值巨大的低质量代码。

这一切看上去似乎非常合理，但是行业总体的创造力在下降。特别是，如果中国的 IT 公司在开源领域的影响力过小时，整个市场都将充斥在水平一般的程序员。

所以，这样一来，我们又不得不考虑一个问题，开源是否在不断降低组织的技术创造力？对于个人来说，可以通过学习造轮子来提升，对于组织呢？

## 阅读源码的本质：解析字符串？

这一年里，受项目及公司大佬的熏陶，我开始寻求以半自动化的方式来理解代码。这其中的一个过程产物就是 Coca，以及 Chapi。往往复复的练习之后，对于类似工作的自动化，有了更充足的把握。

在机器执行一段代码之前，我们要进行一系列的前端转换：词法解析、语法解析、语义分析、生成中间代码，随后再交由编译器后端处理，直至目标程序。而对于人类来说，我们理解的过程是相似的，稍有区别的是，生成的中间代码是人类语言。

而对于一个新的项目来说，我们要做的就是把理解汇制成架构图，又或者是它们之间的调用关系。于是，Coca 和 Chapi 就是用来做这部分工作的自动化 —— 解析字符串。

所以，我一直想写一个工具，用于自动化生成不同语言的代码结构。只是吧，一直缺乏一个明确地用户场景，理论有了，技术有了，就差有人买单了。

## 可执行的背后：二进制？

起先，因为在完成之前的一个 Todo 列表，我便开始折腾写一个 TLV 编码、解码器。TLV，即标识域（Tag/Type）+ 长度域（Length）+ 值域（Value），它是用在通信协议的一种编码格式。

随后，我便开始尝试解析 Java 生成的 class 文件，嗯，我发现做的事情和 TLV 解析，又或者是 Coca 做的语法解析有些类似。Java .class 文件毕竟有固定的格式，所以反而比直接字符串处理更加简单。

紧接着，我继续扩展对于 Java 虚拟机的理解，尝试模仿造一个简单的 JVM，理解整个 Java 应用是如何运行起来的。

直到了，我发现有一个新的领域：Android 对于 Java 二进制的各种骚操作：class 转换为 dex，对 dex 进行优化，dex 转 oat 二进制……。

生活就是这么充满乐趣。

## Go、Rust 成为编程语言的新希望？

作为一个浪漫的程序员，我们无不时刻想着创建一个新的语言/DSL，以丰富我们的业余生活，调节一下 copy/paste 的乏味感。

在陆陆续续地学习 Go 和 Rust，并用它们创造了一系列的工具。我对于创建领域特定语言的事情，又有了一定的感悟 —— 适合的时机，适合的场景，便容易产生适合的语言。

特定的语言解决某一类特定的问题。当然了，新的编程语言还需要有一定体量的公司才能支撑起来。不过，领域特定语言就不需要了，你想干些什么，你就可以轻松上手了。

## 回到底层编程和基础设施

所以，既然大部分 Web 开发都是重复性 CRUD 的。那么回到底层编程、系统编程，偶尔造造系统软件，也是相当的不错。特别是当前，整个市场的大环境对于基础设施的需求，一直在持续不断地提升。

创造和学习是一种乐趣。

嗯，我打算业余生活写写底层代码，进行进行系统编码。

