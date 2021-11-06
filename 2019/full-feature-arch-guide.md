# 全功能架构守护

年初，因为客户项目的缘故，接触了架构守护的相关内容。随后，我便对这个理念颇为感兴趣。四个月前，我编写了一个前端架构守护框架，Dilay （ [https://github.com/phodal/dilay](https://github.com/phodal/dilay) ）。到了下半年，我开始编写系统分析和重构工具 Coca （ [https://github.com/phodal/coca](https://github.com/phodal/coca) ）之后， 我对架构守护又了一些新的理解。

## 测试代码坏味道

故事开始于，我在实现 Coca 的一个功能：**测试代码坏味道**。

歪个楼，虽说大部分的公司开发都不写测试，但是我还是要提一提的。事实上，写测试才能反应出一个程序员的真实水平，因为编写测试比编写功能难得多。代码设计的不合理性，往往难以通过测试来反应，而编写测试的时候，一下子便能反应出设计的问题。譬如，你写了一堆 static 方法是难以测试的；你写了过程式的方法，也是难以实施测试的；你写了又长又……。

也因此，代码上的糟糕设计，一一反应在测试代码上 —— API 不合理性。而你要知道，只凭机器是难以识别诸多的坏味道。如，长的方法可以抽取成多个不表意的短方法，只需要按下键盘上的 Ctrl + Alt + M。

顺带一提，现在你可以执行 `coca tbs` 查找测试代码中的坏味道。尽管当前只支持几种简单的坏味道，如 RedundantPrintTest、SleepyTest、IgnoreTest、EmptyTest 等，未来还会支持更多的测试。

对，从上面的几种坏味道中，就能一一呈现代码的坏味道。也因此，现有的架构守护，缺少相应的职责。

## 再谈架构守护

当我们谈论软件架构的时候，我们讨论的是一系列相关的抽象模式。而架构守护，则是通过测试来确保这些模式被实施。依据这个理念，我把架构划分为了四个层级：


每个层级呈现出不同的模式，如代码级上的设计模式，模块级上的依赖反转等等。

## 全功能架构守护

### 分层快照

我和我的同事们讨论后，一致认为 ArchUnit 缺乏一个自动化的设计 —— 你还要编写代码。所以，我们达成了架构快照才是更合适的做法。

如果你知道前端的 Snapshot Testing，那么你对这个话题就会有点熟悉了。

 - 生成分层架构快照
 - 提交代码时，产出架构快照差异
   - 成功。对比快照差异，合入差异
   - 失败。修改代码，生成新快照



`coca arch`

### 模式快照



