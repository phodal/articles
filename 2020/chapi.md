# Chapi —— 一个通用语言元信息转换器

来，一起用高效（hard way）的方式学习多种编程语言，Kotlin + Scala、Python、Go、Java、TypeScript、C#……

## Chapi 起源

为了向开源重构与分析工具 Coca 中提供多语言支持（原先只支持 Java 语言），我又双叕开始造新的轮子。上个月底尝试了使用 Antlr 的 Go runtime，但是遇到一系列的挫折加之因为公司内部的一些项目需要类似的工具，我便开始从 JVM 系的语言中寻找一个合适的选择。。

结合疫情的影响，我结束了打苍蝇为乐的休息时间，在月初（2020.2.1）便启动了 Chapi 项目的开发，使用的语言是 Kotlin。

在过去的半个月里，我在这个项目上编写了大量的代码，一些有意思的内容、特性如下所示：

 - 完全 TDD 的项目。只有充分的测试，才能保证语法解析不出错。
 - Kotlin 语言。Java 是 Antlr 框架的一等公民，Kotlin 是 JVM 系，更加简洁。
 - 主流编程语言支持。已经完全支持 Java 语言，支持 Python、Go、TypeScript 的数据结构解析，正在支持 Scala、C 和 C# 语言。
 - 插件化支持。（正在实现）
 - JSON 输出（基于 ``kotlinx.serialization``）。
 - 统一的代码数据结构模型。

先是重写了 Coca 的基础设施中的 AST，再在其基础上提供了更通用的代码模型，并添加了不同类型语言的支持：

| Features/Languages  |   Java |  Python  | Go  |  Kotlin | TypeScript | C     | C# | Scala |
|---------------------|--------|----------|-----|---------|------------|-------|----|-------|
| syntax parse        |    ✅  |      ✅  |   ✅ |   TBC   |     ✅     | TBC   |  🆕 | 🆕 |
| function call graph |    ✅  |          |      |         |            |       |     |   |
| arch/package graph  |    ✅  |          |      |         |            |       |     |   |
| real world validate |    ✅  |          |      |         |            |       |     |   |

我在维基百科上看到了一个编程语言的分类，便添加了更多编程语言的支持，以尝试从不同的语言中构建出统一的代码的数据模型。

|            | Languages     |  plan support    |
|------------|---------------|-------------|
| Algol      | Ada, Delphi, Pascal, ALGOL, ... |  Pascal? |
| C family	 | C#, Java, Go, C, C++,  Objective-C, Rust, ... | C, Java, C#, Rust? |
| Functional | Scheme, Lisp, Clojure, Scala, ...| Scala  |
| Scripting  | Lua, PHP, JavaScript, Python, Perl, Ruby, ... | Python, JavaScript |
| Other      | Fortran, Swift, Matlab, ...| Swift?, Fortran? |

与 Coca 稍有不同的是，这是一个过程比结果重要的项目。在实现的过程中，慢慢成为代码/代码语法方面的专家。不同的编程语言都有各自独特的语法，都需要不断地熟悉相关的语法。

## Chapi Core：代码的模型

在继续了解，让我们来看看一个项目的结构：

```
系统（system）
 - 项目（project）
	 - 源码（source code）
		 - 包（package）
			 - 类（class)
				 - 函数
					 - 参数
					 - 返回类型
					 - ……
			 - 函数（function）
			 - 对象（object）
			 - 接口（interface）
			 - ……（trait，struct）
		 - 包（package）
	 - 项目信息
		 - 依赖管理
 - 项目（project）
```

它适用于 Java、TypeScript、Go 等语言，基于此，我们就有了 Chapi 的基础的数据结构：

```bash
// for multiple project analysis
code_project
code_module

// for package dependency analysis
code_package_info
code_dependency

// package or file as dependency analysis
code_package
code_container

// class-first or function-first
code_data_struct
code_function

// function or class detail
code_annotation
code_field
code_import
code_member
code_position
code_property

// method call information
code_call
```

随后，我们就可以开始将不同的编程语言转为 JSON。

## 插件化 AST：基于 Antlr 的 AST 解析

有了基础模型之后，我们要做的事情就是程序员应该做的事情：AST 解析。我们需要编写多种编程语言的 AST，好在我们已经有了 Antlr。而社区也已经有各种使用 Antlr 编写的编程语言 AST，见 Antlr 官方维护的 [https://github.com/antlr/grammars-v4/](https://github.com/antlr/grammars-v4/) 。

然后一点点地结合测试，解析我们所需要的数据：

 1. package name
 2. import name
 3. class / data struct
    1. struct name
    2. struct parameters
    3. function name
    4. return types
    5. function parameters
 4. function
    1. function name
    2. return types
    3. function parameters
 5. method call
    1. new instance call
    2. parameter call
    3. field call

于是，下述的 Java 代码：

```
package adapters.outbound.persistence.blog;

public class BlogPO implements PersistenceObject<Blog> {
    @Override
    public Blog toDomainModel() {

    }
}
```

便可以转换为 JSON 对象，存储到数据库中：

```json
{"NodeName":"BlogPO","Type":"CLASS","Package":"adapters.outbound.persistence.blog","FilePath":"","Fields":[],"MultipleExtend":[],"Implements":["PersistenceObject<Blog>"],"Extend":"","Functions":[{"Name":"toDomainModel","Package":"","ReturnType":"Blog","MultipleReturns":[],"Parameters":[],"FunctionCalls":[],"Annotations":[{"Name":"Override","KeyValues":[]}],"Override":false,"Modifiers":[],"InnerStructures":[],"InnerFunctions":[],"Position":{"StartLine":6,"StartLinePosition":133,"StopLine":8,"StopLinePosition":145},"Extension":{},"IsConstructor":false,"extensionMap":{}}],"InnerStructures":[],"Annotations":[],"FunctionCalls":[],"Parameters":[],"InOutProperties":[],"Imports":[],"Extension":{}}
```

## 未来

有了这个 Chapi 生成的 JSON 数据，我们可以：

1. 查找代码中的坏味道
2. 生成数据结构（class/struct）的依赖关系
3. 可视化项目的依赖情况
4. 自动化重构代码
5. ……

除此，我们还可以：

1. 将 A 语言的领域模型转换到 B 语言中（整洁架构条件下：纯编程语言实现，无第三方依赖时）。
2. 将设计模型转换为领域模型
3. 实现领域模型的架构守护
4. ……

想法有多大，Chapi 就有多少可能。

## Chapi 需要你的参与

Chapi 是一个史诗级的工程，所以，如果你想提升你的底层知识、想进入开源社区，欢迎加入到 Chapi 的开发中。

在这里，你将学会：

 - 真实世界的 Kotlin 实战
 - 成为一个代码专家
 - 熟悉某一语言、多个语言的语法树解析
 - TDD 的手把手实战
 - 开源项目经验

怎样？一起玩吧！

