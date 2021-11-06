# 如何快速识别烂项目

软件开发是一个非常有意思的复制 + 粘贴活动。开发业务代码的时候，大部分人都不会不加思索地添加代码。毕竟，聪明的产品经理/项目经理们，天才式地想出了用代码行数的方式来计算 KPI，又或者是通过提交次数来进行考核 —— 虽然小步提交是个好东西，但是吧，大部分人不经过练习还是掌握不会的。

最近，我还我的朋友们说到，她们公司的打算强制一天只能提交一次代码。这绝对是代码行数计算 KPI 之后的，又一个伟大地创举式的地发明。如果我有直接颁发诺贝尔奖的权力，我一定给送给他一奖杯。

好了，回到正题。

## 自上而下的代码分析

最近，刚好因为项目的关系，需要分析某一系统的代码行数。通过一系列的复制 + 粘贴和 Excel 操作，我大致有了一套 DIY 的自动化分析方案：自上而下的代码分析。当然了，这肯定不是我先发明的，在某处一定有论文和代码、工具。只是我依据自己的想法和需求，完善了一下现有的方案。要知道，已经有大量地代码分析工具了。

其实总体的思路非常简单：项目行数 -> 包行数 -> 修改历史 -> 引用分析。

具体来说，就是：

1. 通过代码行数（LOC）统计工具，统计总体的代码情况。
2. 结合代码行数（LOC）统计工具，统计各个包的代码情况
3. 获取 Git 提交历史，统计出经常修改的包或者是类。
4. 构建语法树、制品（如 jar）分析，统计出引用次数最多的包。

唯一麻烦的地方就是做一些自动化。所以，这些功能就被我完善到 [Coca](https://github.com/phodal/coca) 里了，笑~。

好了，让我们来看个示例。这里以开源项目 intelli-community （即 IDEA 的社区版）为例。

## 项目级代码行数

市面上已经有大量的行数统计工具，大家可以自行寻找。这里我用的是 Coca （GitHub：https://github.com/phodal/coca ），集成了三方用 Go 实现的 CLOC 统计功能。

首先呢，我们要实现的是分析整个项目的行数情况 `coca cloc .` ：

```bash
───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Java                     66554   5172301   688054    512630  3971617     603221
Python                   10017    424614    31629     34876   358109      22329
Kotlin                    6383    602814    89130     35660   478024      51292
Plain Text                4105    635689     5799         0   629890          0
Groovy                    3397    154817    23296     12364   119157       4683
XML                       2549    494074    10056      3008   481010          0
HTML                      2329     63331     2988      3623    56720          0
SVG                       2124     21078       23        87    20968          0
JSON                      1155    346795      352         0   346443          0
Shell                      535      8295     1138       734     6423        811
Markdown                   425      9660     1434         0     8226          0
Properties File            384     42069     2545      1348    38176          0
YAML                       384      3264      202        55     3007          0
XML Schema                 345    196649    17963         0   178686          0
JavaScript                 169     30569     1562      5151    23856       3895
...
───────────────────────────────────────────────────────────────────────────────
Total                   101908   8389984   898893    629497  6861594     703260
───────────────────────────────────────────────────────────────────────────────
Estimated Cost to Develop $288,297,976
Estimated Schedule Effort 132.017220 months
Estimated People Required 258.681675
───────────────────────────────────────────────────────────────────────────────
```

嗯，从规模上来看，这真的是一个超级大的项目，接近 700 万行的规模。所以，我第一次看到的时候，也不知道从哪里下手，于是我便想着是不是从包（目录）结构能解决这个问题。

PS：Coca 当前只支持单体分析，考虑有多模块和微服务系统的存在，我会在未来必要的时候，添加对应的实现。

## 按目录分析 

简单来说就是，我们可以按目录执行 cloc，然后汇总结构即可。

所以，进一步地我们就可以执行 `coca cloc . --by-directory`，得到一个 CSV 数据，根据自己的需要进行编辑：

| package |	summary | Java | Python | Kotlin | Plain Text |
|---------|---------|------|--------|--------|------------|
| platform | 1800542 | 1460686 | 106 | 244586 | 4669 |
| java | 1479891 | 1059828 | 0 | 35224 | 267792 |
| plugins | 1765695 | 983860 | 70301 | 151816 | 150158 |
| android | 1865010 | 769437 | 52 | 325659 | 101848 |
| python | 664760 | 240080 | 287641 | 24626 | 17855 |
| xml | 866926 | 108794 | 0 | 207 | 174471 |
| jps | 66671 | 63437 | 0 | 1498 | 729 |

还可以绘制成图表。


除此，我还提供了一个 `--top-file --top-size 10` 的参数，以了解行数最多的几个文件。

```bash
| LENGTH | COMPLEXITY |             LOCATION              |
|--------|------------|-----------------------------------|
|   1642 |        236 | ConstraintLayoutHandler.java      |
|   1492 |        375 | ConstraintComponentUtilities.java |
|   1189 |        166 | CommonActions.java                |
|   1184 |        325 | ConstraintWidget.java             |
|   1169 |        129 | SingleWidgetView.java             |
|   1115 |        213 | ScoutArrange.java                 |
|   1097 |        281 | ScoutWidget.java                  |
|   1081 |        224 | 3d/Rasterize.java                 |
|   1016 |        159 | LayoutlibSceneManager.java        |
|   1014 |        220 | TimeLinePanel.java                |
```

接着，只需要层层下推，我们就可以分析出哪个是系统最复杂的一部分。如下图中的复杂点，依次是：platforms、java、plugins、android。

## 变更频次

紧接着，我们就可以通过获取 Git 提交历史来知道，对应文件的修改变化。这里，我依旧使用的是 `coca git -t`。它源自于对于 `git log --all --date=short --pretty="format:[%h] %aN %ad %s" --numstat --reverse --summary` 命令的分析结果，有兴趣的读者可以参考 Coca 的源码，自行编写不同版本地对应实现。

可怕的是，我在 intellij-community 执行了 `coca git -t` 之后，生成了一个 241M 的文件，回去 GitHub 看了一眼：累计 290，459 次提交。

在我第一次没意识到应该记录下 log 之后，我又重新执行了一遍。最终，拿到了结果：

```
|                                                                                            ENTITYNAME       | REVSCOUNT | AUTHORCOUNT |
|-------------------------------------------------------------------------------------------------------------|-----------|-------------|
| platform/util/resources/misc/registry.properties                                                            |      2473 |         224 |
| platform/platform-impl/src/com/intellij/openapi/editor/impl/EditorImpl.java                                 |      1211 |         149 |
| platform/platform-api/resources/messages/IdeBundle.properties                                               |      1209 |         181 |
| platform/platform-resources/src/META-INF/LangExtensions.xml                                                 |      1206 |         192 |
| plugins/InspectionGadgets/InspectionGadgetsAnalysis/resources/messages/InspectionGadgetsBundle.properties   |      1113 |         159 |
| platform/platform-resources-en/src/messages/ActionsBundle.properties                                        |      1004 |         161 |
| platform/platform-resources/src/META-INF/PlatformExtensions.xml                                             |       937 |         162 |
| platform/util//src/com/intellij/util/ui/UIUtil.java                                                         |       779 |         120 |
| platform/platform-impl/src/com/intellij/openapi/application/impl/ApplicationImpl.java                       |       763 |         133 |
| platform/platform-resources/src/META-INF/LangExtensionPoints.xml                                            |       762 |         150 |
| platform/lang-impl/src/com/intellij/util/indexing/FileBasedIndexImpl.java                                   |       684 |         126 |
| java/java-analysis-impl/src/com/intellij/codeInsight/daemon/impl/analysis/HighlightUtil.java                |       675 |         117 |
| platform/platform-resources/src/idea/PlatformActions.xml                                                    |       671 |         139 |
```

然后，看一眼 `registry.properties` 是一个有 1800+ 行的配置文件，EditorImpl.java 是一个有 5000+ 行的 Java 代码，UIUtil.java 也有 3600+ 行……。嗯，效果是不是也相当理想，再看看 UIUtil.java 这一个名字，一看就非常适合重构。

## 高引用

最后，可能会进入慢的一步，分析代码，生成 AST。考虑到 IDEA Community 的这个代码量。我就不重复演示了，以 GitHub 的示例为例 `coca count`：

```bash
+------------+--------------------------------------------------------------------------+
| REFS COUNT |                                  METHOD                                  |
+------------+--------------------------------------------------------------------------+
|          2 | com.phodal.pholedge.book.BookRepository.byId                             |
|          2 | com.phodal.pholedge.book.model.Book.toRepresentation                     |
|          2 | com.phodal.pholedge.book.BookRepository.save                             |
|          2 | com.phodal.coca.analysis.JavaCallApp.parse                               |
|          2 | com.phodal.pholedge.book.BookRepository.save                             |
|          2 | com.phodal.coca.analysis.JavaCallApp.parse                               |
|          1 | com.phodal.pholedge.book.model.Book.save                                 |
```

最后，我们又回到了这个模型上。

![高引用与高修改](https://migration.ink/images/refs-change.png)

考虑到 AST 的慢的程度，我已经有一个更好的实现方式。

## 结论

分析代码是一件很有意思的事情。一番操作下来，能学习到非常有意思的东西。



