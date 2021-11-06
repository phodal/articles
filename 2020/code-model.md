# 为代码建模

去年年底，在公司大佬的带领下，我们结合架构守护的需要，对代码进行了简单的建模。在过去的几个月里，我一直工作在相关的事项上，不断地优化、改进相关的模型：

 - 重构 Coca 的模型，以支持 Java 以外的语言
 - 基于 Kotlin MultiPlatform 技术重写模型，以在未来提供多平台、语言的支持

所以，大致上有了现在称之为 3.0 版本的代码模型。花了这么大的力气干活，还是应该分享一下相应的经验。这样一来，不 996 的你，也许能做出更好的轮子。[狗头][狗头]

## 引子 1：文本即代码，代码即测试数据

PS：在那一篇《[如何同时学会两门编程语言？](如何同时学会两门编程语言？)》中，我大抵提到了这一小节的内容，所以它对你来说可能有些重复。

首先，让我们来看段代码。如下是一段 Scala 语言编写的 hello, world ：

```scala
object HelloWorld {
  def main(args: Array[String]): Unit = {
    println("hello, world!")
  }
}
```

哦，不对， 它真的是一段代码吗？哦，不对，在这里它的用途不是给机器执行的，它的用途只是一段测试用例：

```kotlin
@Test
internal fun shouldIdentObjectName() {
    val container = ScalaAnalyser().analysis(helloworld, "hello.scala")
    assertEquals(container.DataStructures.size, 1)
    assertEquals(container.DataStructures[0].NodeName, "HelloWorld")
    assertEquals(container.DataStructures[0].Type, DataStructType.OBJECT)
}
```

反过来，这些测试文本也需要是真正可以工作地代码。更多地测试示例可以见：[https://github.com/phodal/chapi](https://github.com/phodal/chapi)

## 引子 2：代码即语法，语法即代码

将代码转换为特别的模型，我们还需要做的一件事情是：识别代码。代码本身是依照特别规则编写的字符串。这些特定的代码规则便是语法。也因此，如果我们要将不同的编程语言的源码转为模型，就需要不同语言的语法。

如下是一个 TypeScript  的 function 语法规则的部分代码示例：

```antlr
functionExpressionDeclaration
    : Function Identifier? '(' formalParameterList? ')' typeAnnotation? '{' functionBody '}'
    ;
```

它可以匹配上下面的代码：

```typescript
function sum(x: number, y: number) : void {
	return x + y;
}
```

对应的便可以映射上代码：

 - Function -> 'function'
 - Identifier -> 'sum'
 - formalParameterList -> 'x: number, y: number'
 - typeAnnotation -> ': void'
 - functionBody -> 'return x + y'

这样一来，我们就能迈向下一步了，抽取代码模型。

## 引子 3： 代码即模型

>  在通信和信息处理领域，代码（code）是指一套转换信息的规则系统，例如将一个字母、單詞、声音、图像或手势转换为另一种形式或表达，有时还会缩短或加密以便通过某种信道或存储媒体通信。

所以，我们可以先简单地把代码视为：行为 + 数据结构，它们统一称为模型。而模型又分为两种**数据结构的模型**和**行为的模型的模型**。举个例子，在 Golang 中，我们使用 struct 作为结构体，来存储同一类型的数据：

```
type Books struct {
   title string
   author string
   subject string
   book_id int
}
```

如果只是这样的代码，我们可以轻松地把它转换为数据结构。

> 在计算机科学中，数据结构是计算机中存储、组织数据的方式。

然后，还有行为呢？行为事实上，就是各种表达式，而表达式，归根到底还是各种各样的模式，因为我们需要存储这些表达式。

## 代码描述代码，模型描述模型

终于，我们回到了正题：如何用代码描述代码。事实上，我们已经讲完了这个故事的大纲，剩下的就只是一些连线了。

好激动，我们终于要开始造轮子了，那么我们要怎么开始呢？

### 0. 寻找语法解析器及现成语法

市面上已经有一系列现成的词法解析器、语法解析器：

 - JavaCC
 - Lex 和 Yacc
 - Flex 和 Bison
 - Jison （for JavaScript）
 - Parsec
 - Antlr（for All）

最后，我选择了用 Antlr，因为公司的大佬们告诉我用 Antlr：先用 Antlr 解析它们，再写个 Antlr-like 来解析它们，再写个语言来写解析器。这个楼，有点歪。

大家选择 Antlr 的主要原因，Antlr 官方维护着社区贡献的各种语言的 Antlr 编写的语法：[https://github.com/antlr/grammars-v4/](https://github.com/antlr/grammars-v4/)

### 1. 设计代码模型

我们已经有足够的知识，来将一段代码转为数据模型，并设计一个测试体系来保障代码的健壮性（测试 + TDD）。接下来，便是要以**迭代**式的方式来设计一系列模型作为容器，容纳一段一段的代码：

 - **包体系与结构**
 - **结构体/函数体**。
 - **继承体系**
 - **函数结构**
 - ……

这是一个复杂的过程，而且我保证**现在的模型不是完整的**，所以等我真正写完 Chapi 之后，我会重新起一篇文章来解释这个模型。在那之前，请阅读代码：[https://github.com/phodal/chapi](https://github.com/phodal/chapi/tree/master/chapi-domain) 。

因为我也是从一个简单的模型开始，重构了三次之后，才有了一个勉强可以工作地模型。

### 2. 将代码数据放到容器中

在我们有了模型之后，我们便可以编写模型的代码，作为容器来放置内容。

```kotlin
@Serializable
open class CodeCall(
    var Package: String = "",
    var Type: String = "",
    var NodeName: String = "",
    var FunctionName: String = "",
    var Parameters: Array<CodeProperty> = arrayOf<CodeProperty>(),
    var Position: CodePosition = CodePosition()
) {
  ...
}
```

这样一来，我们可以 “完整” 的将代码转为模型，还有对应的序列化的 JSON 结果。

```json
[{"NodeName":"Outer","Type":"CLASS","Package":"","FilePath":"hello.scala","Fields":[],"MultipleExtend":[],"Implements":[],"Extend":"","Functions":[],"InnerStructures":[{"NodeName":"Inner","Type":"OBJECT","Package":"","FilePath":"hello.scala","Fields":[],"MultipleExtend":[],"Implements":[],"Extend":"","Functions":[],"InnerStructures":[],"Annotations":[],"FunctionCalls":[],"Parameters":[],"InOutProperties":[],"Imports":[],"Extension":{}}],"Annotations":[],"FunctionCalls":[],"Parameters":[{"Modifiers":[],"DefaultValue":"","TypeValue":"i","TypeType":"Int","ReturnTypes":[],"Parameters":[]}],"InOutProperties":[],"Imports":[],"Extension":{}}]
```

然后存储到数据库中。

### 3. 打包容器成项目

进一步地，如果我们想以项目的维度来构建出完整的模型，我们还需要代码包的相关信息。考虑到和代码相关度不高，有兴趣的同学可以参考 Coca 中的源码： https://github.com/umijs/qiankun 。

### 4. 识别不同语言中的细节

> 细节是魔鬼。

尽管我们针对于类、函数已经有解了，但是仍然还有不同语言的核心部分还需要有解决：

 - 高阶函数（Higher-order functions）。
 - 嵌套函数（Nested functions）。
	 - 命名（Named）
	 - 匿名
 - 嵌套数据结构/类。 

基于此，我们需要进一步完善模型。

### 5. 应对奇技淫巧

如我们在 Chapi 大本营里讨论的，还有各种奇怪的代码，如 C 语言的：

```c
for(int i=0, j= 0; i<20&&j<30; i++) {
    j++;
}
```

我想不出来他们为什么要这么写，但是你还是要编译成功。

### 6. 重复步骤 4~6

### 还有，技巧 1： All in One Test

你需要一个包含所有语法的文件，以避免 NullException。

### 以及，技巧 2：回归测试

在真实项目运行的时候，记得编写测试，以确保下次是正常的。

## 下一步，表达式转为模型

如何将表达式准确转换为类型？

