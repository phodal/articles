# 迁移自动化：Ant 转 Maven

过去的一两个星期里，我一直在设计一个遗留系统的改造方案。作为改造的第一步是，改造基础设施，其中的一部分就是对于构建工具的迁移。

在早期的系统中（大概就 10 年前），像 Maven 这样的中间仓库还不流行。为了开发方便，开发人员往往将 jar 包放置到项目中，与代码一起提交到代码仓库中。尽管随着时间的流逝，越来越多的相似架构的系统被替换。但是对于一些公司来说，由于改造成本的原因，这样的系统仍然存在。

于是乎，我就有机会去练习我的重构自动化，自动化迁移构建工具。

当然了，这里的自动化是半自动化的，因为自动化并不是那么靠谱，所以需要一个手动核对的过程。

## build.xml 转 pom.xml

从 Ant 迁移 Maven 并不是一件困难的事，如果对应的依赖都在 Maven 仓库中心有的话，并且使用 maven 进行构建的话。那么，我们只需要写一个工具，它可以：

1. 从 `build.xml` 中读取依赖的 jar 路径
2. 遍历 jar 路径，从中提取 `pom.xml` 和 `pom.properties`
3. 解析 `pom.properties` 中的 `groupId`、`artifactId` 和 `version`
4. 生成这些依赖关系保存到 `pom.xml` 中
5. 校对依赖信息

所以，我先撸了一个工具来做这样的事，只需要 `merry boom -p .` 就可以自动化这一步了。但是，随后我遇到的一个问题是：有一些古老的 jar 包，并没有这些依赖信息，它只有一个 manifest.mf 文件。

## Jar 的 MANIFEST.MF

如果你打开过 `jar` 文件，就会看到 jar 中包含着一个 META-INF 目录，里面有一个 `MANIFEST.MF`，这个文件定义了与扩展和包相关的数据。 

如下是一个 Spring 框架的某一个包的 MF 文件部分内容：

```ini
Ant-Version: Apache Ant 1.7.0
Bundle-Vendor: SpringSource
Bundle-Classpath: .
Bundle-Version: 3.1.0
Bundle-Name: Backport Util Concurrent
Bundle-ManifestVersion: 2
Created-By: 1.4.2_14-b05 (Sun Microsystems Inc.)
Bundle-SymbolicName: com.springsource.edu.emory.mathcs.backport
```

通过这个信息以及文件名，我们可以简单的得到一个包的三个关键信息： `groupId`、`artifactId` 和 `version`。

### 社区依赖

不过呢，多数时候我们还是需要从 Google + Maven 仓库搜索才能拿到这些信息。

然而呢，只能到这些信息还不够，我们还需要获取它的依赖信息。比如说 A 依赖于 B，而 B 缺少它的依赖信息时，我们在开发的过程中就容易编译错误。所以，我们还需要获取它的 `Import-Package` 和 `Require-Bundle` 从中解析出它的依赖关系。

如下代码所示：

```ini
Import-Package: org.springframework.asm;version="2.5.6.SEC01"
Require-Bundle: org.springframework.beans;bundle-version="2.5.6"
```

有了，这些基础之后，我们就可以继续往下走了。

### 内部依赖

不过呢，我们会遇到的一种情况是，我们依赖的外部的包。并没有这些版本信息，除此呢，由于在 Import-Package 里 import 的是 `package` 级别的包，而不是单纯的项目的 groupId + artifactId 的信息。举个例子来说，我们要引的依赖的包是 `org.springframework.beans`，但是引用的时候，会引入多个依赖：`org.springframework.beans`、`org.springframework.beans.utils`、`org.springframework.beans.factory`。

所以呢，我们又需要从 MANIFEST.MF 中读取 `Export-Package` 的信息，并建立一个 map。这样一来说，我们才能准确地识别包信息。

## Ant 转 Maven 自动化

这样一来，我们的工具就完成了：

1. 根据 `build.xml` 扫描 jar 信息
2. 遍历 jar，并解压出 `MANIFEST.MF`
3. 解析 MANIFEST.MF 生成依赖的 map 文件
4. 再次遍历 jar 包，结合 map 文件，生成 pom 文件

对，原理就这么简单，对应到我写的工具上，也就是这么几步。





