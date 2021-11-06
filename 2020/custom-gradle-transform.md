# 自制 Gradle 依赖格式及转换插件

在我们使用 Java + Gradle 开发各类应用的过程中，会发现我们声明的一些包不是 jar 包，诸如 zip、war、aar 等。它也会下载下来，并转换到我们所需要的格式。

这个过程蛮有意思的：

1. 注册对应的 DependencyHandler Transform
2. 创建自定义的 configuration
3. 在接收到对应的 configuration 时，进行依赖转换
4. 在使用时，通过 configuration 获取到对应的制品（artifacts）
5. 在编译的时候，取出这些制品的路径

在我找了好久之后，终于在官方项目的 issue 里找到了一个示例：[https://github.com/gradle/gradle/issues/13200](https://github.com/gradle/gradle/issues/13200) 。这里就基于这个示例来进行讲解。

## 定制 Transfrom

先看这个逻辑：

```java
def artifactType = Attribute.of('artifactType', String)
dependencies.registerTransform(MyTransform.class) {
    it.from.attribute(artifactType, "jar")
    it.to.attribute(artifactType, "my-custom-type")
}
```

当我们接收到 `jar` 这一类的制品时（`artifact`），会调用 `MyTransform`，并将其转换为 `my-custom-type`。`my-custom-type` 只是一种状态标记，方便于我们后面对其进行二次处理。

## Transform 逻辑

对应的 Transfrom 实现如下：

```java
abstract class MyTransform implements TransformAction<TransformParameters.None> {
    @InputArtifact
    public abstract Provider<FileSystemLocation> getPrimaryInput();

    @Override
    void transform(TransformOutputs transformOutputs) {
        File file = getPrimaryInput().get().asFile;
        println "Processing $file. File exists = ${file.exists()}"
        if (file.exists()) {
            File outputFile = transformOutputs.file("copy");
            Files.copy(file.toPath(), outputFile.toPath())
        } else {
            throw new RuntimeException("File does not exist: " + file.canonicalPath);
        }
    }
}
```

没有什么特殊处理，只是简单的记录一下。

## 自定义 Configuration

自定义 Configuration 也依旧很简单


```java
Configuration myConfiguration = configurations.create("myConfiguration")
dependencies.add(myConfiguration.name,  "org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.3.70")
```

## 在 Task 中消费处理完的 Transform

当我们运行 Gradle 的命令时，会先调用依赖处理逻辑，于是就进行了转换。因此，我们可以写一个 task 取出依赖完的依赖，并使用它。

```
tasks.register("consumerTask", ConsumerTask.class) {
    it.artifactCollection = myConfiguration.incoming.artifactView { ArtifactView.ViewConfiguration viewConfiguration ->
        viewConfiguration.attributes.attribute(artifactType, "my-custom-type")
    }.artifacts
    it.outputFile.set(new File("build/consumerTask/output/output.txt"))
}
```

对应的 Task 实现：

```java
abstract class ConsumerTask extends DefaultTask {
    @Internal
    public ArtifactCollection artifactCollection;

    @InputFiles
    public FileCollection getMyInputFiles() {
        return artifactCollection.artifactFiles
    }

    @OutputFile
    public abstract RegularFileProperty getOutputFile();

    @TaskAction
    public void doTask() {
        File outputFile = getOutputFile().get().asFile
        outputFile.delete()
        outputFile.parentFile.mkdirs()
        String outputContent = "";
        for(File f: getMyInputFiles().files) {
            outputContent += f.canonicalPath + "\n"
        }
        Files.write(outputFile.toPath(), outputContent.getBytes())
    }
}
```


嗯，就是这么简单 :) 。




