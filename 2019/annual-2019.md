#  2019 节点: Love Wife & Change Life

> 为什么你还在 ThoughtWorrks？ 

因为不加班。人生总会有很多的选择，在决策的那一刻，你不知道对与错。但是，开心就好。

12 月初，ThoughtWorks 开始了 Annual Review 的 Kick Off，我开始总结这一年的工作，与此同时，我也开始总结我的 2019 节点。今年仍然是『平淡无奇』也过完了重要的一年。

太长不读版：

 - 爱情上，领证了，和 @ 花仲马一起来到了杭州；还差好多钱买房，还得考虑办婚礼的事情。
 - 职业上，从深圳 office 转到了华东 MU，从华东 MU 转到了咨询团队，开始了在 TW 的出差生涯，还有加班生涯。
 - 设计上，每天画了一张画，一年 365 张画；插画，作为文章的一部分，已无处不在。
 - 写作上，出版了《前端架构：从入门到微前端》，印刷量在 7500 左右，有希望在一年内达到 1 万。
 - 编程上，写了更多的工具，愈加丰富的重构经验，顺带深入软件体系的架构。
 - 斜杠上，尝试电子产品的评测，写作相关的收入差不多是前两年之合。
 - 影响力上，开始了 International 的尝试 —— 时间仍然是一个限制因素。
 - 社交上，我退出了大量的微信群，专注于生产内容。
 
嗯，还有游戏，文明大法好：
 
哦，不对，是这个：



所以，对比一下上一年的目标：

1. **技术隐私**，打造了自己的 Serverless 密码管理器：https://github.com/phodal/mopass ，作为一个 Chrome 插件，它很好地作为了我的二次管理认证工具。
2. 非技术写作。好似没有一个开始，似乎也不是一个好的目标。借这个名义，我看完了《刺客信条》的小说。
3. 工具。开发了更多、更有意思的工具，还有更多的 PoC。
4. 设计。天天练习插画，更快的画画速度，质量上也有所提升。也烧了更多的钱绘画工具上。
5. Coach。幸好在上一年里，它不是一个目标，扶不起的人太多了。对的人，对的事，才能成——借口。
6. 影响力。受众级别比上一年有所提升，还有更深度的内容。

不算太好，也不算太糟糕。

## 编程：平台 + 工具 + DSL

惯例，依旧是工作 + 业余。

### 工作：Platform & Tools

工作上没有圈，也没有点，今年的工作简直是一团糟，还加了人生的第一次班，而明年还会有更多。Work–life balance 不断被打破，就得寻找一个更合适的地方 —— 如果有的话。

#### 平台

上半年，工作的主要内容是大前端开发框架 / 平台，所以研究了一段时间低代码编程，写了那篇《无代码编程》的文章。一番操作下来，发觉重点在于 AST 和 DSL。因此，除了开发一些日常的工具之外，我开始撸 dilay 框架，创建了 subal 项目……。作为一个苦逼的 Tech Lead，除了项目相关的两个团队，还要照顾公司的其它多个团队。日常不是一般的忙，开会、开会、开会，还得做架构？？还要评绩效？？填别人的坑？？还有写代码……

一个也不能落下，每个都落下了。

做了一个大前端开发平台，这样一折腾下来，收获倒也是挺大的，我对研发体系有了更深入的研究。考虑问题的时候，比以往更加系统，更加全面。文档、脚手架、示例应用、CLI 工具、IDE / 编辑器集成、售后 Q & A 等等一个都不能少。于是，在项目上写了对应的 CLI 工具，尝试把文档融入了开发工具中……

没毛病，老子可以各种吹了：不要做平台。我 Phodal 就是……，我也不会……。

#### 工具

下半年，beach 了两三个月，写了个重量级应用 Inception，然后，转到了咨询团队。来到了新的 U，有了更多的灵感和时间去写工具，也从公司大佬新哥那获得一堆 Todo List。所以，下半年在业余时间写了更多的代码，写了更多的 DSL。所以，DSL 成为我这一年的一个主要风向。
 
我有了遗留系统重构工具： [Coca](https://github.com/phodal/coca)，还有了 Badsmell 识别工具：[Sprite](https://github.com/phodal/sprite)，以及对应的重构建议工具：[fanta](https://github.com/phodal/fanta) ……。

它们都是使用 Go + Antlr 写的，target 是便宜的后端开发语言 Java。一顿瞎操作下来，除了更懂 Java 语法，我还学会了 Go。明年，我就可以 Rust + WebAssembly 搞 C or TypeScript 的语法分析了，一下子学会好几种东西的感觉好爽。

卧槽，又要兴奋的失眠了。想想，还是很美好的。

### 业余：工具 + DSL

2018 年底，我的 GitHub 数逼近 40,000；2019 年底，也有 48,615了，可不敢说逼近了。明年我的目标就是 50,000 star 的时候，发个朋友圈，哦，不对应该是 Twitter。

依旧的 Serverless 仍然是我的后端最佳选择，我用它写了我的密码管理工具：[moPass] [https://github.com/phodal/mopass](https://github.com/phodal/mopass) 。继 ADR 和 Phodit 之后，我的另外一个日常使用工具。我的业余项目上还上手了 Golang，嗯，真香。

#### Architecture

今年，有幸可以在项目中引入对于前端架构的探索，进一步地完善了我的前端架构体系，也产生了前端架构守护框架 [Dilay](https://github.com/phodal/dilay)，完美的造了个 PPT。

所以，在实践了 Domain Driven Design 和 Clean Architecture 之后，我开始思考 One Architecture 的可能性，尽管我已经用 JavaScript / TypeScript 证明了它的可能性：[https://github.com/phodal/one](https://github.com/phodal/one)

然而，Java 仍然是后端的主流语言，一个 Java 转 JavaScript 的工具不可缺少，而编程语言有那么多，所以我们需要的一个是 DSL 转任何语言的工具。也就是我最近在做的 Code 项目： [https://github.com/phodal/code](https://github.com/phodal/code) ，实践上还有待完善，只是 hello, world 出来。大抵，还需要半年地时间完善。

#### 基准化

考虑到人总是会老的，Phodal 是人，所以 Phodal 会老的。我继续写工具、文章来沉淀知识，以用于以后甩出一个链接（装 x 神器）：

 - [Clean Architecture]：[https://github.com/phodal/clean-frontend](https://github.com/phodal/clean-frontend) 
 - [React Boilerplate]：[https://github.com/phodal/react-boilerplate](https://github.com/phodal/react-boilerplate)
 - [New Project Checklist]：[https://github.com/phodal/new-project-checklist](https://github.com/phodal/new-project-checklist)
 - Inception 工具：[https://github.com/phodal/inception](https://github.com/phodal/inception) 
 - [Path to Productioon]：[https://github.com/phodal/path](https://github.com/phodal/path)
 - [Tech Lead Assessment]：[https://github.com/phodal/tla](https://github.com/phodal/tla)

对应的还有一篇相关的文章：《[如何创建你的应用脚手架](https://www.phodal.com/blog/how-to-create-application-boilerplate/)》，年轻就是好，对了，还有 Tech Lead 的基准化：《[Tech Lead 的养成](https://www.phodal.com/blog/how-to-be-a-tech-lead/)》。

### Everything as Code：DSL

作为一个 Markdown 资深用户，除了进一步完善我的 Phodit，我还结合 Markdown 写了很多工具：

 - Markdown 定制文档工具，见 《[【架构拾集】基于 Markdown 文档展示系统设计](https://www.phodal.com/blog/architecture-in-realworld-markdown-based-document-system-design/)》
 - Markdown 转思维导图，见 Inception 工具。
 - Markdown 转 PPT 工具 [mdppt](https://github.com/phodal/mdppt) 

不过呢，定制别人的 DSL 始终是比较一个比较 hack 的方式，所以如何卓有成效地开发一个 DSL，便成为了一件非常有意思的事。所以，公司大佬说的 DSL as Data, Data as DSL 仍然是一个不错的目标。

在那篇《[云开发：未来的软件开发方式](https://github.com/phodal/cloud-dev)》中，我提到了在未来几年，我要做的一些事情：
 
 - 更易于实践的微架构
 - 完善的代码化体系
 - 寻求合适的协作设计

所以，设计和抽象 DSL（Domain Specific Language）将成为了我未来几年一个重要战略。也因此，从大体上来说，它仍然是我的下一年目标和计划。

## 写作：100 万浏览量 + International

年初，出现了一个新的里程碑，我的博客 phodal.com 累计访问量突破了 1,000,000 万。

考虑到在微前端和 Clean Architecture 的实践，已经和国外的速度差不多，外加国内的 996 环境。所以在在今年年中，我尝试将 International 作为 Impact 的一个新方向。因此，在这篇总结里，我把写作相关的部分分为了大中华区和 International 区。

### “大中华区”

虽然我在写新书的时候，看了很多小说，试图去改进，但是依旧在豆瓣上被吐槽『写出来太理论太像翻译腔』。没救了，没救了，写过 776 篇博客的我，表达能力依旧还有巨大地提升空间。

#### 出版

今年 5 月出版的《[前端架构：从入门到微前端]》，出版社的总印刷数已经有 7500（并非卖完），豆瓣读书上的评分也有 7.6 分 —— 比前两本书多出了一份。瞬间又有动力准备下一本书了，只是怕是没有那么多时间了。

颇为遗憾的是，出于字数少的原因，我在『前端架构』  一书中多加了一个章节。而由于出版时间太早，少了后来实践的『Clean Architecture』——这是另外一个前端所需要的分层架构模式。将它与 Serverless 配合，就形成了我们所需要的 One Architecture。

#### 文章：体系规划

从内容上来看，我对今年的文章倒是颇为满意的：

 - 《无代码编程》
 - 《整洁架构》
 - 《构建可信的软件架构 10 要素》
 - 《微前端架构》
 - 《管理依赖的 11 个策略》
 - 《云开发：未来的软件开发方式》
 - ……

但是如你所料，我创建了一篇又一篇地长文章，手就有点疼，坐久蛋也疼。

只是呢，好像也没有新的亮点了。

### International 

今年从我的观察来看，我在开源领域开始逐步走向非中文世界。Mooa 和 ADR ，迎来了一个又一个的国际友人的支持。我的 GitHub followers 也多了一个又一个的国际友人。

内容国际化，是今年年中开始的一个新的方向 —— 之前的另外一个国际化目标是：**开源软件的国际化**。

虽然我的英语语法并不是那么靠谱，但是 Google Translate 也差。我相信同翻译腔一样，只要会被人吐槽英语语法不行，说明我已经成功引起大家地注意了。

#### English Articles

由于种种原因（诸如文章太长懒得翻译、高质量的文章不够多），产出仍然相对比较少。

不过，也算还行，我在 Dev.to 上创建了我的账号，发了一篇微前端相关的文章，还有一篇 Clean Architecture 相关的文章，也产生了一定的影响力。除了几十个的掌声，一万左右的单篇文章浏览量，还有 StackOverflow 有相关的问题指向了我的文章，笑而不语~。

所以，继续翻译更多的文章吧，是时候依赖反转一下了。

#### Review

过去的几年里，Review 英文书籍显然是国际赛道的一部分，只是呢，当时呢，这个 business line 还没想好，现在也没想好。

今年还是 Review 了 Packt 出版社的一本书籍《Web Development with Angular and Bootstrap - Third Edition》，遗憾的是近一二年 review 的书，都没有被引入国内。

不管怎样，国际化应当成为 2020 的一个继续前进的方向。

## 设计

我换了一个又一个的工具：

 - iPad + Apple Pencil。买前生产力，买后爱奇艺
 - Wacom Intuos Pro。专业级手绘板，相当的不错。
 - 绘王 Kamvas Pro 16。嗯，解控屏，效率就是高。对于我这种非专业级选手，还是非常好用的。
 - Wacom Intuos Mini。出差专用，个小板子虽然不那么好用，但是我也算是习惯了。

终于，我还是没画好画。

年中的时候，我尝试录制绘画的过程到 B 站、抖音上。但是，画的时候往往发现，录视频的时候，会影响我画画，也就作罢了。

### 画：365 天

上一年考虑到设计的边缘化，我开始采用日常练习的方式，来提升这方面的感觉。

[Daily](https://github.com/phodal/daily)

最初设计的目标是每天 0.5 小时，但是受编程状态的影响，往往会被挤压到 15分钟。好在，随着练习的进一步 ~~一深~~ 深入，我还是能在短暂的时间内，画出一些不错的作品。

稍有不同的是，受出差的影响，我有时候不得不在早上画画。

Whatever，我已经有一堆画了。

所以，如果没有问题的话，它仍将作为我下一年的日常。问题的关键在于：如何结合文章的意图，创建对应的作品？

### Design Thinking

寻找更广泛维度的设计思考。

TBD。

受限于有大量的代码要写，以及好像没有遇到好的设计师。没有灵感嘛，就先这样，慢慢来，路子还长着。万一明年可以遇到可以愉快合作的小伙伴呢。


## 其它

人生苦短，一点点做

## Hello, 2020

嗯，我要登机回杭州了。

咦，2020 呢，要小康。

哦，不对，说好的婚礼呢。

