# Gradle 的 IDEA 支持：自定义 Gradle Tooling Model

最近一直在编写 Gradle 以及 Intellij IDEA 相关的东西。由于市面上最复杂的相关的开源软件就是 Android 相关的，即 Android Studio 和 Android Gradle Plugin，于是我也在不断学习相关地源码。所以，我写了很多相关的文章，在介绍相关的内容。文章中包含了大量的技术细节和实现，所以这些技术文章也就只能在我的博客上看到了。

由于 Intellij IDEA 对 Java 最好，所以这里的文章依旧是基于 Java 场景之下的设计之下。在这方面来看，不得不说 Gradle 和 Intellij IDEA 配合得相关的完美。就这一点而言，它们之间的契合度有点出乎我的意料。

其实在最开始的时候，我一直以为 Intellij IDEA 会解析 build.gradle 的相关信息，然后执行对应的操作。但是，思来想去好像也不是很正常 —— 毕竟源码上也有相关的代码。其中还有两个问题：

 1. 资源浪费。毕竟 Gradle 已经处理了一遍，如果 IDEA 再处理的话，那就两遍了。
 2. 无法确保一致性。毕竟是在两个工具里各种处理了一遍。

直到我找到通过总总手段在 Intellij IDEA 的做工 “垃圾” 的论坛上找到了：[ how to use/create a gradle custom model? ](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206762805-how-to-use-create-a-gradle-custom-model-) ，这样一来，我总算解开了一个个的谜团。

## Gradle Tooling API

首先呢，在 Gradle 的官方文档中，我们看找到一个相关的章节：[Gradle & Third-party Tools](https://docs.gradle.org/current/userguide/third_party_integration.html#embedding)，其中有一个段落是关于：Embedding Gradle using the Tooling API
。

> Gradle provides a programmatic API called the Tooling API, which you can use for embedding Gradle into your own software. This API allows you to execute and monitor builds and to query Gradle about the details of a build. The main audience for this API is IDE, CI server, other UI authors; however, the API is open for anyone who needs to embed Gradle in their application.

简单来说，就是 Gradle 提供了一个名为 Tooling API 的 API，它以编程方式能将 Gradle 嵌入到应用软件中。这个 API 可以让我们执行构建、监测构建，以及查询一个构建的相关信息。

文档说了很多东西，但是就是没有代码 —— 估计用的人太少了。不过，好在最后还提供了一些关键字，如：`GradleConnector.connect() `。这样一来，我就可以验证我找的网上资源否正确。

接下来，只需要搜索类似于：`gradle custom tooling model`，我们就能获得我们想要的内容了。

这里我找到了一个示例代码：https://github.com/bmuschko/tooling-api-custom-model 。

## GradleConnector

[GradleConnector](https://docs.gradle.org/current/javadoc/org/gradle/tooling/GradleConnector.html) 是 Gradle Tooling API 的主入口，通过这个 API，我们可以：

1. 调用 `newConnector()` 来创建一个新的实例
2. 配置连接器
3. 调用 `connect()` 来创建连接到某个项目
4. 当连接结束时，通过调用 ` ProjectConnection.close()` 来清理信息。

然后，还有一个简单的示例代码：

```java
 ProjectConnection connection = GradleConnector.newConnector()
    .forProjectDirectory(new File("someProjectFolder"))
    .connect();

 try {
    connection.newBuild().forTasks("tasks").run();
 } finally {
    connection.close();
 }
 ```
 
 稍后，通过这个示例代码，我们就可以获取到项目中的 ToolingModel 了。如：
 
 ```java
 GradleConnector connector = GradleConnector.newConnector();
connector.forProjectDirectory(new File("../sample"));
ProjectConnection connection = null;

try {
    connection = connector.connect();
    ModelBuilder<CustomModel> customModelBuilder = connection.model(CustomModel.class);
    CustomModel model = customModelBuilder.get();
    assert model.hasPlugin(JavaPlugin.class);
} finally {
    if(connection != null) {
        connection.close();
    }
}
 ```

我们可以从 connection 中找到我们所注册的模型。而在那之前，我们需要先注册模型。

## ToolingModel

为了使用上自定义的 ToolingModel，我们需要两个东西：`ToolingModelBuilderRegistry` 和 `ToolingModelBuilder`。顾名思义，一个用于创建模型，一个用于注册。主要的部分还在于 `ToolingModelBuilder`

在 `ToolingModelBuilder` 中定义了一个接口 `buildAll(String, Project) `，用于为指定的项目创建模型，而这个模型将会用于序列化到客户端，将传输到客户端应用中。Gradle 为这个模型相关的处理注入了各种黑魔法，比如 `missing properties and methods`，详细的见：

> The model object is adapted to the Java type that is used by the client by generating a view, or wrapper object, over the model object. The model object does not need to implement the client Java type, but it does need to have the same structure as the client type. This means that the model object should have the same properties and methods as those defined on the client type. The tooling API deals with missing properties and methods, to allow evolution of the models. It will also adapt the values returned by the methods of the model object to the types used by the client. 

除此，Gradle 还对定义的模型对象有一个不可变的要求，以用于缓存和其它的一些性能优化。同样的，让我们来看看示例：

```java
public class ToolingApiCustomModelPlugin implements Plugin<Project> {
    private final ToolingModelBuilderRegistry registry;

    @Inject
    public ToolingApiCustomModelPlugin(ToolingModelBuilderRegistry registry) {
        this.registry = registry;
    }

    @Override
    public void apply(Project project) {
        registry.register(new CustomToolingModelBuilder());
    }

    private static class CustomToolingModelBuilder implements ToolingModelBuilder {
        @Override
        public boolean canBuild(String modelName) {
            return modelName.equals(CustomModel.class.getName());
        }

        @Override
        public Object buildAll(String modelName, Project project) {
            List<String> pluginClassNames = new ArrayList<String>();

            for(Plugin plugin : project.getPlugins()) {
                pluginClassNames.add(plugin.getClass().getName());
            }

            return new DefaultModel(pluginClassNames);
        }
    }
}
```

嗯，主要的逻辑还在 `buildAll` 中。代码也是相当的简单。

这样，我们就可以在 connector 中获取到这个模型的信息了。

PS：顺便一提，为了测试序列化的准确性，建议用 connector  测试一下。

## ProjectResolver

这里我们就需要使用到前面提到的官方链接：[ how to use/create a gradle custom model? ](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206762805-how-to-use-create-a-gradle-custom-model-) 了。

好几年前，有开发者给 Jetbriain 提了相关的 issue，而官方也给了对应的解决方案：实现一个自定义的 projectResolve 。

```java
<extensions defaultExtensionNs="org.jetbrains.plugins.gradle">
  <projectResolve implementation="my.package.MyCustomPluginProjectResolverExtension"/>
</extensions>
```

然后再实现一个对应的 Resolver 就可以了：

```java
@Order(ExternalSystemConstants.UNORDERED)
public class FooModelPluginProjectResolverExtension extends AbstractProjectResolverExtension {

  @Override
  public void populateModuleExtraModels(@NotNull IdeaModule gradleModule, @NotNull DataNode<ModuleData> ideModule) {
    FooModel myModel = resolverCtx.getExtraProject(gradleModule, FooModel.class);
    if (myModel != null) {
      // do something or store for further processing
    }
    nextResolver.populateModuleExtraModels(gradleModule, ideModule);
  }

  @NotNull
  @Override
  public Set<Class> getExtraProjectModelClasses() {
    return Collections.<Class>singleton(FooModel.class);
  }

  @NotNull
  @Override
  public Set<Class> getToolingExtensionsClasses() {
    return ContainerUtil.<Class>set(
     FooModelBuilderImpl.class
    );
  }
}
```

代码中的 `FooModel` 便是我们定义的模型。

## 其它

这样一来，我们就串起了所有的线。

 1. 创建 ToolingModel，用于通讯过程中的数据传递
 2. 在 Gradle 插件中注册 ToolingModel
 3. 在 IDEA 中读取 ToolingModel 的值
 4. blabla



