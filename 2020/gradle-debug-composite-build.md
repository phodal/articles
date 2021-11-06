# Gradle Debug：基于 Composite Build 的方式

最近，在研究 Gradle 的编译和插件相关的东西，配置了两次 Gradle Composite Build，结果还是忘了。所以，我决定写一篇文章记录一下：如何使用 Gradle Composite Build 调试 Gradle Plugin。

## 常规 Composite Build

对于一般项目来说，采用 Gradle 官网的配置方式即可：[Composing builds](https://docs.gradle.org/current/userguide/composite_builds.html)

1. 在 `settings.gradle` 中添加模块：

```
rootProject.name = 'my-composite'

include ':plugin-example'
includeBuild 'some-plugin'
```

如果没有的话，需要创建一级父目录，并将插件（`some-plugin`）和插件示例（`plugin-example`）放置在同一级目录。然后修改一下插件示例（`plugin-example`）的 plugins 为：

```
plugins {
    id("com.phodal.gradle")
}
```

然后，对于插件来说，需要使用插件：`java-gradle-plugin`，同时配置一下插件：

```
gradlePlugin {
    plugins {
        create('com.phodal.gradle') {
            id = 'com.phodal.gradle'
            implementationClass = 'com.phodal.gradle.MainPlugin'
        }
    }
}
```

这样一来，IDEA 就可以根据 Plugin ID 找到对应的 class，就可以进行愉快地调试。

## 多 Projct 项目集成

对于复杂一点的项目来说，就会麻烦一点，比如说，我们的插件和插件示例，都有 `settings.gradle`。我们就需要 merge 一下这两个项目，所以就会变成一个项目包含另外一个项目的关系。

一个比较容易实现的场景是插件示例，包含插件工程：

```
├── plugin-demo
│   └── src
├── build.gradle
├── settings.gradle
└── gradle-plugin
    ├── build.gradle
    ├── buildSrc
    └── settings.gradle
```

这样，只需要稍微改一下 `settings.gradle` 就可以了。


