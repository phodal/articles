# 自制简易 Gradle 的 Java 编译器插件器

最近，因为兴趣 + 项目的需要，研究了一下 Gradle 相关源码中关于 Java 编译的实现。也顺便按照相关的 Gradle 插件，做了一个简单的插件，了解了相关的原理。

主要步骤如下：

1. 创建、配置 Gradle 的 configurations
2. 创建 task
3. 在 task 中，获取 configurations
4. 在 task 中，配置编译参数
5. 调用 Gradle 插件中的 JavaCompile
6. 完美

相关的文档主要在：[Defining custom configurations](https://docs.gradle.org/current/userguide/declaring_dependencies.html#sec:defining-custom-configurations)

## 创建自定义的 Gradle Configuration

如果是使用 `build.gradle` 的话，那么步骤相当的简单：

```groovy
configurations {
    jasper
}

dependencies {
    jasper 'org.apache.tomcat.embed:tomcat-embed-jasper:9.0.2'
}
```

而在这里，我们写的是 Gradle 插件，所以要用 Groovy 来编写，稍微麻烦一些：

```groovy
val compile = createConfiguration("implementation", true)
compile.dependencies.whenObjectAdded(SomeImplementationAction())

val phodalCompile = createConfiguration("phodalImplementation", true)
phodalCompile.dependencies.whenObjectAdded(SomeImplementationAction())
```

这样一来，我们就可以在 task 中，获取到相应的配置。

## 获取 Gradle Configurations

当我们在 Gradle 中引入测试的时候，需要通过 `testImplementation` 来声明测试依赖。在编译的时候，`implementation` 会和 `testImplementation` 一起加入到编译的 classpath 中。所以在我们的场景这下，我自定义了 `phodalImplementation`，以便测试是否能正常获取 依赖：

```groovy
val config: Configuration = configurations.getByName("implementation")
compileClasspaths.add(config)

val compileClasspath: Configuration = configurations.getByName("phodalImplementation")
compileClasspaths.add(compileClasspath)

this.classpath = mergeClassPath(configurations, compileClasspaths)
```

在 mergeClassPath 中，主要是调用 `setExtendsFrom` 来整合依赖：

```kotlin
private fun mergeClassPath(configurations: ConfigurationContainer, mutableSet: MutableSet<Configuration>): Configuration {
		val finalPath = configurations.maybeCreate("finalCompileClasspath")
		finalPath.isVisible = false
		finalPath.description = "Resolved configuration for compilation for variant"
		finalPath.setExtendsFrom(mutableSet)
		finalPath.isCanBeConsumed = false
		finalPath.resolutionStrategy.sortArtifacts(ResolutionStrategy.SortOrder.CONSUMER_FIRST)
		return finalPath
}
```

接着，我们就可以配置 JavaCompile 了：

## 配置 JavaCompile

对于编译来说，也是蛮简单了，配置一下 `source`、`classpath`、`outputs`、`destinationDir` 之类的就可以了：

```kotlin
this.classpath = mergeClassPath(configurations, compileClasspaths)
this.source = this.project.files(javaSource.toList()).asFileTree

this.outputs.dir("build/classes")
this.destinationDir = File("build/classes")
this.options.isIncremental = true
```

然后在 `compile` 方法中调用 JavaCompile 的编译即可：

```
override fun compile(inputs: InputChanges) {
		super.compile(inputs)
}
```

## JavaCompile 的过程

最后，运行到了 Gradle 代码里了：

```java
@Incubating
@TaskAction
protected void compile(InputChanges inputs) {
    DefaultJavaCompileSpec spec = createSpec();
    if (!compileOptions.isIncremental()) {
        performFullCompilation(spec);
    } else {
        performIncrementalCompilation(inputs, spec);
    }
}
```

Gradle 会将上一步中的 Configuration 里的东西取出来，并将 FileCollection 转为 `List<File>`。接着，根据配置选择合适的编译方式：

```java
private static class ProjectScopeCompileServices {
    JavaCompilerFactory createJavaCompilerFactory(WorkerDaemonFactory workerDaemonFactory, Factory<JavaCompiler> javaHomeBasedJavaCompilerFactory, JavaForkOptionsFactory forkOptionsFactory, WorkerDirectoryProvider workerDirectoryProvider, ExecHandleFactory execHandleFactory, AnnotationProcessorDetector processorDetector, ClassPathRegistry classPathRegistry, ActionExecutionSpecFactory actionExecutionSpecFactory) {
        return new DefaultJavaCompilerFactory(workerDirectoryProvider, workerDaemonFactory, javaHomeBasedJavaCompilerFactory, forkOptionsFactory, execHandleFactory, processorDetector, classPathRegistry, actionExecutionSpecFactory);
    }

    JavaToolChainInternal createJavaToolChain(JavaCompilerFactory compilerFactory, ExecActionFactory execActionFactory) {
        return new CurrentJvmJavaToolChain(compilerFactory, execActionFactory);
    }

    JavaToolChainFactory createJavaToolChainFactory(JavaCompilerFactory compilerFactory, ExecActionFactory execActionFactory, JvmVersionDetector jvmVersionDetector) {
        return new JavaToolChainFactory(compilerFactory, execActionFactory, jvmVersionDetector);
    }
}
```

如使用 `DefaultJavaCompilerFactory` 的话，将会通过 `JavaCompilerArgumentsBuilder` 来直接调用 `javac` 进行编译，并进行相关的参数配置，如：

```java
private void addSourcePathArg(List<String> compilerArgs, MinimalJavaCompileOptions compileOptions) {
		Collection<File> sourcepath = compileOptions.getSourcepath();
		boolean emptySourcePath = sourcepath == null || sourcepath.isEmpty();
		...

		args.add("-sourcepath");
		args.add(GUtil.asPath(sourcepath));
}
```

嗯，然后我们就编译完成了。


