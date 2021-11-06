# 编程语言的支撑体系：构建系统、IDE 和依赖管理

年关（annual review）将近，这一段时间，我在梳理 2020 年做的一些事情，并试着制定下一年的计划。过程中，我发现我做的一些事情，或是工作相关，或是兴趣上的探索，还都可以继续总结出一些文章。在工作上，很多一部分做的事情就是编程语言的支撑体系。外加业余时间里，和同事一起花了一些时间在研究编程语言。在这几部分的结合之下，我对于整个体系的端到端实现有一个整体的认识。

作为一个职业的程序员，在我们的职业生涯里，不可避免地要学习一个又一个的编程语言。虽然多数情况下，我们对于使用什么语言并没有太多的选择权。但是，当我们选择一门语言时，都要考虑一系列的要素，比如：

1. 构建系统
2. IDE/Editor 支撑
3. 依赖管理
4. ……

PS：当然了，对于那些使用 C/C++ 的人来说，这些可能都是例外：他/她觉得自己不需要这些工具，需要的时候可以自己创造一个。所以，这些语言在很长的一段时间里，都缺乏良好的依赖管理工具。

故事开始之前，让我们让 Android 使用的开发和构建来讲述这个过程。

## 从 Android 应用的开发与构建说起

在移动端开发上，虽比不上这个行业的诸多大佬，但我也算是颇有经验的。而恰好一年中有一半的时间，都在相关的项目上。所以，我从宏观上了解了整体的体系。

当我们开始一个新的移动应用时，会从 IDE 里通过模板创造一个崭新的应用，又或者是从某个地方（如 GitHub）寻找合适的模板。而后，为验证模板的有效性，我们通过执行 Gradle 的相关命令，完成一个应用的过程，运行这个 Demo。（PS：这一点与我们使用 Java 开发应用时，并没有太大区别）。

这个过程中，发生了这么一些事情：

1. IDE 通过某种通讯机制，与 Gradle 进行通讯，以执行对应的命令，如 `build`。
2. Gradle 接收到 IDE 的指令后，解析 `build.gradle` 相关的内容，寻找是否存在对应的 Task，如这里的 `build`。
3. 执行 `build` 时，首先要去解决依赖关系，如从对应的 Maven 仓库中下载依赖。
4. 随后，真正地执行对应的构建任务，如调用 javac。

这个过程看上去非常简单，但是背后还藏着诸多的细节问题。

## 构建与依赖管理

当我用 CLOC 工具统计了一下 Gradle 工具的源码时，我才发现这个工具并不简单。而进一步地，在半深入源码之后，我发现构建系统还是颇为复杂的。一个简单的 Java 应用就分为这么一些步骤：

```bash
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:run
```

而当我们有依赖的时候，需要添加上 `classpath`，即将依赖添加到编译的路径中。而对于一些非 `.jar` 类型的依赖而言，如 `.war`，构建工具还要支持对他们的解析。因此，整体的过程就是：

1. 判断是否存在本地的依赖，如果没有的话，从远程获取。如果有依赖冲突的话，解决这些冲突，或者报错。
2. 获取依赖后，根据需要对依赖进行处理。如 Android 中的 `aar` 包的解压等。
3. 结合依赖，对源码进行编译
4. 将所需要的 Java Resources 从依赖的 Jar 拷贝到指定目录 
5. 打包构建后的产物到一个新的 jar 包中

这些只是表面上的一些工作。而为了更好地表述这个过程，需要抽象出一个 `task` 的概念，在这个概念里，一个 task 有输入和输出。如

1. 解析依赖里。它的输出是 `build.gradle` 文件，输出是处理完的依赖路径。
2. 编译任务里。它的输入是源码，输出是 `.class` 文件。
3. 打包任务里。它的输入是一堆文件夹或者文件，输出是一个 `.jar` 包。
4. ……

于是，在有了这些基础之后，为了加快构建，还需要缓存的机制。它对输入和输出进行计算，当两者发生变化的时候，再进行编译。否则就跳过这个任务。

而这些只是核心功能，在非核心的功能区里，还有诸如于 SDK 版本、多输入多输出的变体等等。

##  IDE 与构建系统

在那篇《[编程语言的 IDE 支持](https://www.phodal.com/blog/language-in-ide/)》中，我们已经介绍了编程语言所需要的 IDE 功能，诸如于：

1. 语法高亮
2. 子系统关联与集成
3. 跳转与引用分析
4. 智能感知
5. 重构
6. 快速修复
7. 结构化视图
8. ……

在这篇文章中，大概再回顾一下它与构建系统之间的关系。 IDE 与构建系统一般会存在这种关联：

1. 解析构建系统中的任务。如 Gradle 提供的 task，又或者是 `package.json` 中的 `scripts`，并将它们显式地展示出来，如 IDEA 中的 line marker，又或者是独立的 Gradle pannel。
2. 执行构建任务。即在 IDE 中的 UI 与构建命令相绑定，典型的如 IDEA 中的 Android 应用的构建。
3. 动态修改构建系统（可选）。如 IDEA 中的更新依赖版本，它依赖于解析构建系统的 DSL，并更新对应的 DSL。

对应的有两种机制可以与构建系统通讯：

1. 由构建系统提供构建 API。如 Gradle Tooling API，在那篇《[Gradle IDEA 的项目模型](https://www.phodal.com/blog/gradle-idea-project-model/)》中，我们实际上介绍了由构建系统主动向 IDE 提供模型的方式。
2. 由 IDE 构造一遍构建系统。如 IDEA 对于 Node.js 的处理方式。

简单来说，就是复杂的系统应该由构建系统提供机制，而简单的构建系统则就不会有这样的问题。

## 依赖管理的基础设施

不同语言对于依赖的管理机制都有所不同，但是它们的原理都是相似的：

1. 源码包。即将源码打包，并以特定的格式发布，适用于脚本语言
2. 仓库源。方式类似于源码包，唯一不同的地方是借助于版本管理工具，如 Golang。
3. 类二进制包。典型的是 Java
4. 其它包。如 Maven 可以支持其它自制的包

最有意思的是Maven 的机制，我可以自制依赖，并上传上去。而整个仓库并不关心这个包的内容，我们只需要依赖于它定义的格式即可。如果我们考虑围绕语言来设计依赖管理体系，那么可以考虑的是类似的方式，并借助于 Git 这样的版本工具。这样一来，我们就可以去中心化。





## 其它

嗯，人生苦短，多了解一些有意思的系统吧。
