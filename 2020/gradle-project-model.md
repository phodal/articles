# Gradle 中的 IDEA 项目模型

最近在为新的工具 [Scie](https://github.com/phodal/scie/) 设计一个模型，以用于描述一个具体的代码工程。而刚好最近也在学习 Gradle 相关的项目模型，便想写一篇文章记录一下相关的模型，以支撑起 Scie 的开发。

PS：相关的代码位于 Gradle 源码中的 IDE 部分。

总体来说，用于支撑 IDE 开发的这个模型主要由这么几部分组成：

 - Project，最顶层的类，包含项目的所有信息。
 - Modules，模块，包含单个模块的信息。
 - GradleProject，包含一个模块的 Gradle 的相关信息。
 - ContentRoot，源码部分。
 - Dependency，依赖

从目录上来看总体的类便是，这个就是我们所需要的所有 Gradle 的 IDEA 相关的接口模型了。

 - BasicIdeaProject
 - IdeaCompilerOutput
 - IdeaContentRoot
 - IdeaDependency
 - IdeaDependencyScope
 - IdeaJavaLanguageSettings
 - IdeaLanguageLevel
 - IdeaModule
 - IdeaModuleDependency
 - IdeaModuleIdentifier
 - IdeaProject
 - IdeaSingleEntryLibraryDependency

不过，还是让我们从头看起，即 IdeaProject 部分。

## IdeaProject

IdeaProject 部分的代码相对来说还是比较简单的，主要描述了项目相关的模块（Module）、JDK 信息和对应的语言版本等。

```java
public interface IdeaProject extends HierarchicalElement {
   IdeaJavaLanguageSettings getJavaLanguageSettings() throws UnsupportedMethodException;
    String getJdkName();
    IdeaLanguageLevel getLanguageLevel();
    DomainObjectSet<? extends IdeaModule> getChildren();
    DomainObjectSet<? extends IdeaModule> getModules();
}
```

为了文章的完整性，我补充一下相关的接口类：

```java
public interface HierarchicalElement extends Element {
    @Nullable
    HierarchicalElement getParent();
    DomainObjectSet<? extends HierarchicalElement> getChildren();
}

public interface Element extends Model {
    String getName();
    @Nullable
    String getDescription();
}
```

接着，就是 IdeaModule 部分了。

## IdeaModule

代码也相对比较简单：

```java
public interface IdeaModule extends HierarchicalElement, HasGradleProject {
    @Nullable
    IdeaJavaLanguageSettings getJavaLanguageSettings() throws UnsupportedMethodException;
    String getJdkName() throws UnsupportedMethodException;
    DomainObjectSet<? extends IdeaContentRoot> getContentRoots();
    GradleProject getGradleProject();
    IdeaProject getParent();
    IdeaProject getProject();
    IdeaCompilerOutput getCompilerOutput();
    DomainObjectSet<? extends IdeaDependency> getDependencies();
}
```

除了 Java 相关的配置信息，还有几个点值得关注：

 - 描述源码相关的 IdeaContentRoot
 - 描述 Gradle 信息的 GradleProject
 - 描述依赖的 IdeaDependency
 - 描述输出的 IdeaCompilerOutput

这里我们主要关注于 `IdeaContentRoot` 和 `IdeaDependency`

## IdeaContentRoot

IdeaContentRoot，是用于描述源码相关的信息。这里，只需要看一下对应的接口，就能了解对应的信息了：

```java
public interface IdeaContentRoot {
    File getRootDirectory();
    DomainObjectSet<? extends IdeaSourceDirectory> getSourceDirectories();
    DomainObjectSet<? extends IdeaSourceDirectory> getGeneratedSourceDirectories();
    DomainObjectSet<? extends IdeaSourceDirectory> getTestDirectories();
    DomainObjectSet<? extends IdeaSourceDirectory> getGeneratedTestDirectories();
    DomainObjectSet<? extends IdeaSourceDirectory> getResourceDirectories() throws UnsupportedMethodException;
    DomainObjectSet<? extends IdeaSourceDirectory> getTestResourceDirectories() throws UnsupportedMethodException;
    Set<File> getExcludeDirectories();
}
```

简单来说，就是我们在使用 IDE 的过程中，用于开发的那些要素：源码、测试代码、资源等等。

## IdeaDependency

由于依赖采用的是 `DomainObjectSet<? extends IdeaDependency>`，即继承了这个接口的类都行，于是我找到了：

```java
public interface IdeaSingleEntryLibraryDependency extends IdeaDependency, ExternalDependency {
    File getFile();
    File getSource();
    File getJavadoc();
}
```

不过，我并没有在 Gradle 的代码里找到对应的实现类，而是在 Intellij IDEA 的源码里找到了：`InternalIdeaSingleEntryLibraryDependency`，真是有趣。

## 结论

抽象和描述一个项目并不是一件容易的事情。
