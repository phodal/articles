# 构建的抽象

最近，在研究 Gradle 和 Java 相关构建的实现，让我对不同编程语言的应用构建燃起了一点点的兴趣。

不同编程语言编写的应用，在它运行的状态下，会有不同的运行机制，有的是以二进制的方式运行的，有运行在编程语言的虚拟机之上。而构建所做的事情呢，就是将那些我们写给人类看的代码，转换为机器/程序能看懂的代码。所以，构建的本质就是翻译（~~复读机~~）。

PS：本文旨在尝试性的整理我所了解的构建知识。部分内容限于对某一些编程语言的理解有限，并非非常准确。如有偏颇之此，希望大家指正。

## 引子 1：从 Java 的编译说起

绝大多数程序员都是从 `hello, world!` 开始自己复制、粘贴的人生生涯。对于那些刚上手 Java 的程序员也是类似的：

```
javac HelloWorld.java
```

而当我们依赖于其它的软件包时，就需要在编译时和运行时加入 `classpath` 来加入依赖项。于是，对应的运行命令就如下所示：

```
java -classpath .:libs/joda-time-2.10.6.jar HelloWorld
```

这样，我们就能得到预期的结果了：

```bash
Hello, World
Millisecond time:     in.getMillis():           1599284014762
```

而如果我们需要打成 `jar` 包就需要一个复杂一点的过程：

```
jar cvfm hello.jar manifest.txt HelloWorld.class libs/*
```

这个过程中，涉及到几个关键的要素：

1. 工具链。即 `java` 和 `javac`，以及对应的 Runtime 等。
2. 构建过程。即我要先执行 `javac` 进行编译，再通过 java 命令来启动应用。
3. 依赖管理。即我们的 `joda-time-2.10.6.jar` 的位置获取等问题，以及在打包时加入的过程。
4. 源码配置。即转换过程中的 `class` 和 `java`
5. 过程中的输入和输出。

## 引子 2：任务及任务的输入和输出

对于一个制品的构建来说，我们往往会把它拆分为一系列的任务，每个任务有自己的输入和输出。当输入发生变化的时候，需要变化对应的输出。紧接着，我们只需要对任务进行编排即可：

```javascript
exports.build = series(
  clean,
  parallel(
    cssTranspile,
    series(jsTranspile, jsBundle)
  ),
  parallel(cssMinify, jsMinify),
  publish
);
```

如上展示的是：哪些任务可以并行，哪些任务需要按顺序执行——也可以认为是任务的依赖。

当然了，还有一种任务是 watch 任务，只用于开发时，而非构建时。如下是 Node.js 中的 Gulp 构建工具的文件监控示例：

```javascript
function javascript(cb) {
  // body omitted
  cb();
}

function scss(cb) {
  // body omitted
  cb();
}

watch('src/*.scss', scss);
watch('src/*.js', series(javascript));
```

两间结合之下，我们就会看到增量任务的概念：只针对修改的部分进行编译，以提升构建效率。在这方面做得比较好的就是 Gradle ，看个官方的示例[InputChanges](https://docs.gradle.org/current/dsl/org.gradle.work.InputChanges.html)：

```groovy
abstract class IncrementalReverseTask extends DefaultTask {
    @Incremental
    @InputDirectory
    abstract DirectoryProperty getInputDir()

    @OutputDirectory
    abstract DirectoryProperty getOutputDir()

    @TaskAction
    void execute(InputChanges inputChanges) {
        inputChanges.getFileChanges(inputDir).each { change ->
            if (change.fileType == FileType.DIRECTORY) return

            def targetFile = outputDir.file(change.normalizedPath).get().asFile
            if (change.changeType == ChangeType.REMOVED) {
                targetFile.delete()
            } else {
                targetFile.text = change.file.text.reverse()
            }
        }
    }
}

```

同样的，它也需要我们监控对应的输入和输出。稍有不同的是，Gradle 会对文件进行索引，每次只提供变化的部分，让我们根据自己的实际需要进行处理。

增量构建相关资源：

 - [tup](http://gittup.org/tup/) 是用于 Linux、OSX 和 Windows 的基于文件的构建系统。它输入文件的更改列表和有向无环图（DAG），然后处理DAG 以执行更新依赖文件所需的适当命令。
 - [ninja](https://github.com/ninja-build/ninja) 是一个专注于速度的小型构建系统，类似于GNU Make。
 - [SCons](https://scons.org) 是一套由Python 语言编写的开源构建系统，类似于GNU Make。

## 引子 3：可选的依赖管理（地狱）

关于依赖的管理槽点，我已经写过一系列的文章，诸如于：[管理依赖的 11 个策略](https://www.phodal.com/blog/dependency-management-strategy/)、[依赖孪生：低成本的依赖安全方案](https://www.phodal.com/blog/dependency-twins/)。

单纯从构建这件事情上，对于依赖的管理是可有可无的。出现这个状况的主要原因是：历史上的编程语言都不考虑这个问题。所以，在古老的 C/C++ 语言中，构建系统就是一个头疼的问题。当然了，新晋的 Golang 也缺少良好的设计。

好在，对于依赖管理来说，这个过程并不复杂：

1. 包命名和版本机制
2. 包管理服务器
3. 构建和运行时的依赖管理
4. 包冲突处理
5. ……

## 构建的抽象

好了，有了上面的这一系列基础知识之后，接下来我们就可以看看不同的构建系统里，对于同一概念的抽象，整合了 Bazel、Gradle、Cargo、NPM 等之后有了一个基础的抽象层次：

 - 工作空间（workspace）。工作空间是一个或者多个软件包的集成，它们可以共享依赖、输出目录配置等等。典型的有 Java 中的 Gradle `settings.gradle`、Rust 中的 Cargo 的 `Cargo.toml` 等。
 - 仓库。仓库可以映射到 Git 的 repository 中，代表一个可独立构建的软件。
 - 包。最小的可执行单位的项目结构。
     - 包布局。对应于不同的语言、构建系统来说，它用于定义代码的存放位置和结构。
 - 制品。即构建产生的产物，可能是可复用的软件包，也可能是可运行的应用。
 - 任务。定义构建的规则，并执行。

**FAQ**

1. 为什么是没有项目？在业务领域和技术领域，我们对于项目的定义存在着一定的歧义性。为了减少二义性，我们使用工作空间 + 仓库来解决这个问题。工作空间可以视为一个完整的业务项目。而仓库呢，则是单一个的代码库，可能是一个库，也可能是包含库的完整工程。
2. 现有的最佳方案是 Bazel。

### 工作区

工作空间是一个或者多个软件包的集成，它们可以共享依赖、输出目录配置等等。典型的有 Java 中的 Gradle `settings.gradle`、Rust 中的 Cargo 的 `Cargo.toml` 等。

我们可以将其视为最终的产物，如 Android 生成的 APK，Rust 最后生成的可执行文件。过程中，生成的共享的包都是为了支持这个工程的一部分。

先看 `CMakeLists.txt` 的目录，我们在工作区的根节点，定义了这个工程，并添加了 `projectA` 和 `projectB`。

```cmake
cmake_minimum_required(VERSION 3.2.2)
project(globalProject)

add_subdirectory(projectA)
add_subdirectory(projectB)
```

以用于生成最后的构建产物。相似的还有 Rust 中的 workspace：

```
[workspace]

members = [
    "adder",
]
```

又或者是前端的 Yarn 中的工作区：

```
{
  "private": true,
  "workspaces": ["workspace-a", "workspace-b"]
}
```

它们做的都是相同的事情。

### 仓库

这个概念的再提取是来源于 Bazel。仓库是一系列包的合集，我们可以将其视为团队的边界，从某种意义上可以看作是代码仓库。对于一个庞大的工程来说，它的代码来源是多种多样的，来自组织内的其它团队，来自组织外的其它团队。每个独立的部分，即是一个仓库。

值得注意的是，从最终产物来看，每个团队的产出都是仓库，但是呢，在团队内部，他们就是工作区。

让我们看个 Gradle 的多项目构建示例（Android 工程）：

```
.
├── README.md
├── library_a
├── app
│   ├── build.gradle
│   └── src
├── build.gradle
├── local.properties
├── settings.gradle
└── third-partys
    ├── ...
    ├── build.gradle
    └── settings.gradle
```

从目录结构来看，这个是一个工作区，而在工作区呢，它包含了一些三方的代码仓库（third-partys），以及自身的库 `library_a` 和应用 `app`。

因此，在这里的 `library_a` 和 `third-partys` 的各个项目都算是仓库。

### 包

包是一系列代码的合集，它可大可小。最主要的原因在于，因为构建时，我们可能会把一个仓库（哪怕是最小的 Gradle 项目）产出多个包，如 Java 项目中的 `src/main` 和 `src/test`。

于是在诸如 bazel 这样的构建工具中，支持自定义的包：

```bash
src/my/app/BUILD
src/my/app/app.cc
src/my/app/data/input.txt
src/my/app/tests/BUILD
src/my/app/tests/test.cc
```

对于一个包来说，往往我们还需要定义一系列的相关信息，如包名、依赖信息、入口等等。如 Bazel 中对于 Java 构建的示例：

```bazel
java_binary(
    name = "ProjectRunner",
    srcs = ["src/main/java/com/phodal/ProjectRunner.java"],
    main_class = "com.phodal.ProjectRunner",
    deps = [":greeter"],
)
```

这已经实现了对于不同包的信息抽象。顺带的再看个 Java 包中的 `MANIFEST` 的示例：

```
Main-Class: HelloWorld
Class-Path: libs/joda-time-2.10.6.jar
```

我们就可以知道之间的联系。

#### 包定义

在打包阶段，我们以简单的形式定义了这个包——因为它并非那么重要，我们也不关心。而当我们决定发布这个包到互联网时，我们就需要好好定义这个包。对应的一些必要信息有：

 - name
 - version
 - authors
 - license
 - description
 - ……

这些信息用于在包管理中心展示，并向使用者提供包相关的信息等。不同的语言中使用的是不同的形式，Rust 使用了自定义的 toml，而诸如 Maven 仓库中则使用了 XML：

```
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
	<name>...</name>
  <description>...</description>
```	

类似的在 NPM  的 [package.json](https://docs.npmjs.com/creating-a-package-json-file) 中也使用了类似的字段：`name`、`verison` 等信息。

而在这些编程语言中，这个东西就设计得过于简单了，如 Python 的 pip 中使用的 requirements.txt 来管理依赖，当你要发布包的时候使用 `setup.py` 进行配置。于是，你的应用如果不发布，那就没有包名了……。

#### 包布局

构建工具在设计的时候，会设计默认的软件包分层结构，这个分层架构就是包布局（package layout）。构建工具通过这个布局，来获取所需的输入源和配置等信息。它也包含了一些默认的配置，如 `src/main` 指向了源码的目录，`src/test` 指向的是测试代码（不会加入到制品中）

```bash
├── build.gradle
└── src
    ├── main
    └── test
```

对于使用者来说，它们也可以针对于它们的需要扩展这个布局，如 Gradle 里的 SourceSets：

```groovy
sourceSets {
  main {
    output.resourcesDir = file('out/bin')
    java.outputDir = file('out/bin')
  }
}
```

对于其它语言也是类似的。但是呢，对于某些语言来说，并非有这么强的关联，如在 Golang 中，就没有这么强的约束。只是呢，原先是默认值，现在需要开发人员来手动配置。

### 制品

制品是最终的构建产物。同样的，在不同的语言中有不同的命名方式。在 Gradle 中称为 artifacts，在 Rust 中称为 targets……。制品，主要涉及到的是各种文件的流转及其流转规则。

举个简单的例子，一个 `jar` 文件中必须包含一个 `MANIFEST.MF`，以用于配置应用程序、扩展和类装载器等相关信息。而相关的文件又会以 `META-INF` 的方式组织起来。

因此在整个制品的创建过程中，就是复制对应的文件，进行相应的转换，如 java -> .class，再复制到对应的目录，最后再打包在一起的过程。

### 任务：规则引擎 + DSL

在上述我们看到的例子中，很多就是创建了自身的 DSL，而后用于构建。只有这样才能让使用者得到最大的方便。这是一个相当复杂的过程，它相当于我们要设计一个和平台、语言无关的 DSL。而这种演变方式有多种：

1. 使用 API 抽象的内部 DSL。诸如于 Webpack、Gulp 等实现。
2. 自制的外部 DSL 语言。如 Gradle 所使用的 Groovy、多语言的 Bazel。

规则引擎本身是一组关于任务的 DSL，看个 Gradle 的例子：

```groovy
task copyReportsDirForArchiving2(type: Copy) {
    from("$buildDir") {
        include "reports/**"
    }
    into "$buildDir/toArchive"
}
```

它所做的事情就是复制。对应的 Gradle 打包示例也是蛮简单的 DSL 抽象：

```groovy
task packageDistribution(type: Zip) {
    archiveFileName = "my-distribution.zip"
    destinationDirectory = file("$buildDir/dist")

    from "$buildDir/toArchive"
}
```

Gradle 使用的就是外部 DSL。再看看 Webpack 的打包示例：

```javascript

module.exports = {
	entry: './path/to/my/entry/file.js',
	output: {
		filename: 'my-first-webpack.bundle.js',
		path: path.resolve(__dirname, 'dist')
	},
	module: {
		rules: [
			{
				test: /\.(js|jsx)$/,
				use: 'babel-loader'
			}
		]
	},
	plugins: [
		new webpack.ProgressPlugin(),
		new HtmlWebpackPlugin({template: './src/index.html'})
	]
};
```

这里的 rules 就是一个简单的规则引擎（使用正则表达式来匹配）

两种模式各自有自己的优缺点，复杂场景下，使用 DSL +  自定义的脚本更容易完成。

PS：看来有空，我也应该写一个的规则引擎

## 构建的扩展

对于主流的构建系统来说，他们都支持不同形式的扩展支持：

1. 外部 DSL 扩展
2. 插件化的接口编程
3. 项目内编程语言扩展
4. 项目外编程语言扩展

大部分的东西，我们已经在文中的先前部分提到了，这里就不重复描述了。

## 结论

应用的构建是一个相当有意思的过程。

设计一个构建系统也变得颇为有趣的。

参考资料：

 - [Gradle vs Bazel for JVM Projects](https://blog.gradle.org/gradle-vs-bazel-jvm)
 - [Bazel: Concepts and terminology](https://docs.bazel.build/versions/3.5.0/build-ref.html)
 - [Yarn: Workspaces](https://classic.yarnpkg.com/zh-Hans/docs/workspaces)
 - [Gradle: Authoring Multi-Project Builds](https://docs.gradle.org/current/userguide/multi_project_builds.html)
 - [Cargo: Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html#workspaces)
 - [Gulp: Tasks](https://www.gulpjs.com.cn/docs/getting-started/creating-tasks/)

相关目的开源库：

 - [lerna](https://github.com/lerna/lerna)  A tool for managing JavaScript projects with multiple packages. 
 - [bazel](https://github.com/bazelbuild/bazel)
 - [Blueprint](https://github.com/google/blueprint/) is a meta-build system that reads in Blueprints files that describe modules that need to be built, and produces a Ninja manifest describing the commands that need to be run and their dependencies.
