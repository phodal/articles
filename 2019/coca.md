，# 为了更好的代码，我写了一个工具：Coca

> 好的代码是可以重构出来的。

如我在先前的文章所说，我最近的工作主要是在做架构重构、代码重构。所以，一如既往地，我又写了个工具来帮助我完成相关的工作。这样一来，下次我可以更快速地完成相关的工作。

在这之前，已经有大量的工具可以做类似的事情。如我司已有大佬开源了 Tequila （ https://github.com/newlee/tequila ） 这样的架构、依赖分析工具。只是呢，简单的架构分析是无法满足我的需求的。并且，本着写了工具就是赚经验的思想，我决定写一个自己的工具。

## Coca 简介

从按我的实践经验来看，我将重构分为四种类型：

 - **分层架构重构**。 在不改变业务逻辑的情况下，进代码架构进行调整。即根据单一职责和依赖倒置原则的思想，对系统进行模块拆分与合并，以明确职责降低耦合度；对包进行重新规划，划分包之间的边界，减少代码间的耦合。
 - **模式重构**。针对特定模式的坏味道，采用设计模式来提升可扩展性，增加可读性。
 - **模型重构**。在包含测试的情况下，通过识别和发现模型的行为，将行为聚合到模型中。
 - **微重构**。对于一些小的代码坏味道，可以通过 IDE 重构来快速改善即有代码，而不会影响到业务功能。

而《重构：改善既有代码的设计》一书主要针对的是微重构。因为重构项目的难度不是一般的大，对于经验不足的个人、团队来说，重写往往比重构来得便捷。

所以，根据我的需要我写了自己的工具，以用于改善即有代码的设计：

> Coca 是一个用于遗留系统重构的瑞士军刀。它可以分析代码中的 badsmell，行数统计，分析调用与依赖，进行 Git 分析，以及自动化重构等。

GitHub 地址：https://github.com/phodal/coca

安装：

1. `go get -u github.com/phodal/coca`
2. 直接从 GitHub 上下载可执行文件

在执行所有的命令之前，需要先执行 `coca analysis` 以生成对应的依赖关系数据。

So，让我们看看 Coca 1.0 包含了哪些功能？

## API 调用

这个功能目前主要针对的是 Spring 开发的，它似乎已经是 Java 世界的后端服务的标准。

### REST API 生成

只需要执行 `coca api` 就可以生成我们想要的结果：

1. 函数调用图
2. API 调用层级

下图是 API 的统计信息：

| SIZE | METHOD |                      URI                       |                                 CALLER                                 |
|------|--------|------------------------------------------------|------------------------------------------------------------------------|
|   36 | GET    | /aliyun/oss/policy                             | controller.OssController.policy                                        |
|   21 | POST   | /aliyun/osscallback                            | controller.OssController.callback                                      |
|   17 | GET    | /subject/list                                  | controller.CmsSubjectController.getList                                |
|   17 | GET    | /esProduct/search                              | search.controller.EsProductController.search                           |
|   17 | GET    | /order/list                                    | controller.OmsOrderController.list                                     |
|   17 | GET    | /productAttribute/list/{cid}                   | controller.PmsProductAttributeController.getList                       |
|   17 | GET    | /productCategory/list/{parentId}               | controller.PmsProductCategoryController.getList                        |
|   17 | GET    | /brand/list                                    | controller.PmsBrandController.getList                                  |
|   17 | GET    | /esProduct/search/simple                       | search.controller.EsProductController.search                           |

对应的全景图：

![API Call](coca-api.png)

当然了，你也可以轻松通过参数过滤一下你想要的内容。

这个功能的目标是用于未来向 DDD 重构时，构建出限界上下文。

### 调用关系图

也可以只看某一部分的依赖关系图：

```
coca call -c com.phodal.pholedge.book.BookController.createBook -r com.phodal.pholedge.
```

输入对应的完整方法名，和想要去除的包含即可：

![Method Call](coca-call.png)

### 反向依赖关系图

还能生成对应的反向调用关系图：

```
coca rcall -c org.bytedeco.javacpp.tools.TokenIndexer.get
```

结果如下图所示：

![Cocal Rcall](coca-call.png)

## 行为分析

由于代码只是反应系统的另一部分，我们不得不从版本管理工具中获取更多的信息，于是有了：

```
coca git
```

### 文件修改统计

排名靠前的文件，可以帮我们看到一些问题：`coca git -t`，于是乎，我们就有了：

|                                                     ENTITYNAME                                                      | REVSCOUNT | AUTHORCOUNT |
|---------------------------------------------------------------------------------------------------------------------|-----------|-------------|
| build.gradle                                                                                                        |      1326 |          36 |
| src/asciidoc/index.adoc                                                                                             |       239 |          20 |
| build-spring-framework/resources/changelog.txt                                                                      |       187 |          10 |
| spring-core/src/main/java/org/springframework/core/annotation/AnnotationUtils.java                                  |       170 |          10 |
| spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java                |       159 |          15 |

对，这是 Spring Framework 中最常修改的文件，前面三个文件看上去是合理的，但是 `AnnotationUtils.java` 显然有问题：

![Spring AnnotationUtils](spring-annotaion-utils.png)

对应的 DefaultListableBeanFactory.java 也有 2000+ 行左右的规模：

![Spring DefaultListableBeanFactory](spring-DefaultListableBeanFactory.png)

从代码的行数和修改次数来看，它们都是上帝类，并且经常出现  Bug。

### 代码年龄统计

嗯，还有诸如于 ``coca git -a`` 这样的普通功能：

```
+-------------------------------------------------------------------------------------------------------------------------+-------+
|                                                       ENTITYNAME                                                        | MONTH |
+-------------------------------------------------------------------------------------------------------------------------+-------+
| helloworld.go                                                                                                           |  2.04 |
| README.md                                                                                                               |  2.04 |
| imp/imp.go                                                                                                              |  2.04 |
| .gitignore                                                                                                              |  2.01 |
| cmd/root.go                                                                                                             |  2.01 |
| test/coca_suite_test.go                                                                                                 |  1.94 |
| learn_go_test.go                                                                                                        |  1.94 |
| core/languages/java/JavaLexer.tokens                                                                                    |  1.84 |
| core/languages/java/java_lexer.go                                                                                       |  1.84 |
| examples/step2-Java/domain/ValueObjectD.java                                                                            |  1.84 |
```

### 其它功能

还有诸如基本的分析功能等： `coca git -b`

```
+-----------+--------+
| STATISTIC | NUMBER |
+-----------+--------+
| Commits   |    350 |
| Entities  |    514 |
| Changes   |    255 |
| Authors   |      2 |
+-----------+--------+
```

有兴趣可以自己去探索。

## 知识提炼

嗯，这也是我计划在明年实践的功能。

### 方法提取

作为此功能的第一步，我想的是先从代码中提取单词：`coca concept`：

```
+------------------+--------+
|      WORDS       | COUNTS |
+------------------+--------+
| context          |    590 |
| resolve          |    531 |
| path             |    501 |
| content          |    423 |
| code             |    416 |
| resource         |    373 |
| property         |    372 |
| session          |    364 |
| attribute        |    349 |
| properties       |    343 |
| headers          |    330 |
+------------------+--------+
```

作为第一个简单的版本，我只是做了分词和统计，下一步便是专业性的统计。而后，用于生成领域专用的特定语言。

### 提交信息提取

你懂的，这里面的信息才是足够的丰富。

 TBD

### 提取中文注释

下一步，我应该做类似的事情，哈哈哈

## 坏味道识别

这是一个非常通用的功能，你可以在各种各样的工具里找到。所以，我并没有特意地去增强里面的功能，也没有添加太多的功能——因为我知道他们比我的工具专业。

只需要：`coca bs` 就会得到一个建议修改的 JSON 文件：

```json
{
   "lazyElement": [
      {
         "File": "examples/api/model/BookRepresentaion.java",
         "BS": "lazyElement"
      }
      ...
   ]
}
```

结合我写的 fanta 工具，可以生成对应的修改建议。

## 批量重构

主要是用于结合上述工具的分析结果，通过人工 + 智能的方式来实现批量化自动修正。

当前 API 处于试验阶段，请不要在生产环境使用。支持以下功能：

 - 批量重命名
 - 批量移动文件
 - 批量删除未使用的 import
 - 批量删除未使用的类

使用的方式也非常简单：

```
coca refactor -m move.config -p .
```

你可以试试，哈哈

## 代码统计分析

代码统计工具，采用的是 SCC：

> scc is a very fast accurate code counter with complexity calculations and COCOMO estimates written in pure Go

愉快地： `coca cloc`，然后：

```
───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Go                          58     31763     7132       890    23741       2847
Java                        44       971      208        21      742         62
Markdown                     8       238       75         0      163          0
Gherkin Specificati…         2        32        2        16       14          0
Document Type Defin…         1       293       36         0      257          0
License                      1       201       32         0      169          0
SQL                          1         2        0         0        2          0
SVG                          1       199        0        34      165          0
Shell                        1         3        1         1        1          0
XML                          1        13        0         0       13          0
gitignore                    1        61        8         4       49          0
───────────────────────────────────────────────────────────────────────────────
Total                      119     33776     7494       966    25316       2909
───────────────────────────────────────────────────────────────────────────────
Estimated Cost to Develop $803,822
Estimated Schedule Effort 14.120551 months
Estimated People Required 6.743156
───────────────────────────────────────────────────────────────────────────────
```

其它功能可以见 `scc` 工具的官方文档。

## 重构适合度评估

TBD

## 其它 

这是我第一个使用 Golang 写的工具，希望我的用法足够的 Go Style。

首页：https://coca.migration.ink/

GitHub 地址：https://github.com/phodal/coca

愉快地：`go get -u github.com/phodal/coca`

