# LLVM + Rust JIT hello, world

最近在和我一同事一起，使用 Rust 来创造一门新的编程语言：[Charj](https://github.com/charj-lang/charj)。而在实践方面，我们都是这方面的新手，所以不得不经历一番尝试。而作为其中的一部分，必然就是由一个 hello, world 开始的。由于在这个过程上，遇到一些小坑，所以我决定写篇文章记录一下。

最后代码见：[https://github.com/phodal/rust-llvm-practises](https://github.com/phodal/rust-llvm-practises)

简单介绍一下，一些工具：

 - Rust 是由 Mozilla 主导开发的通用、编译型编程语言。设计准则为“安全、并发、实用”，支持函数式、并发式、过程式以及面向对象的编程风格。
 - LLVM 是一套编译器基础设施项目，为自由软件，以 C++ 写成，包含一系列模块化的编译器组件和工具链，用来开发编译器前端和后端。
 - [Inkwell](https://github.com/TheDan64/inkwell)，一个非官方的 LLVM 的 Rust 封装。
 - [lvmenv](https://github.com/termoshtt/llvmenv)，一个用来管理不同版本 LLVM  的三方工具，主要也是 Rust 编写时使用的。

作为结论来说，我们需要这么一些步骤：

1. 安装 LLVM 相关的编译工具
2. 安装 llvmenv
3. 通过 llvmenv 来编译 LLVM
4. 了解 LLVM 的一些基本概念
5. 编写和运行我们的 hello, world

而实际上，前三步是同一件事情：安装和编译 LLVM，只是使用了道具不同而已。

## 编译和安装 LLVM

现在，我们开始安装 llvmenv（PS：在它的 GitHub 上看到最新的安装步骤：https://github.com/termoshtt/llvmenv ）。从 README 上来看，我们需要安装 `make`, `ninja`, `clang`（使用的是 macOS 系统）这些工具。由于 Clang 是 XCode 自带的，所以不需要安装了。

```bash
brew install cmake ninja
```

紧接着，就可以安装 `llvmenv`

```bash
cargo install llvmenv
```

llvmenv 的官方文档不是那么详细，缺少了如何安装的说明。所以在 issue 上有相关的内容，2333。简单来说，步骤就是：

```bash
llvmenv init
llvmenv entries
llvmenv build-entry 10.0.0
```

`entries` 命名可以列出相关的版本，`build-entry` 则是构建对应的 LLVM 版本。而由于网络的原因，我直接 `build-entry` 并不成功，所以我需要先下载对应版本的 LLVM。接着修改配置：`$XDG_CONFIG_HOME/llvmenv/entry.toml`，将 LLVM 指定我下载的位置，如：

```toml
[local-llvm]
path = "/path/to/your/src"
target = ["X86"]
```

然后，我就可以构建了：

```
llvmenv build-entry local-llvm
```

构建完之后，我们就可以先大概熟悉一下 LLVM 的一些基本概念。

### LLVM 基本概念

详细的说明，我这里不就列举了，可以看官方文档。我只要是罗列一下后面代码会用到的一些基本概念。来源于《[使用LLVM IR 编程](http://richardustc.github.io/2013-06-19-2013-06-19-programming-with-llvm-ir.html)》

**Module**，可以将 LLVM 中的 Module 类比为 C 程序中的源文件。一个 C 源文件中包含函数和全局变量定义、外部函数和外部函数声明，一个 Module 中包含的内容也基本上如此，只不过C源文件中是源码来表示，Module 中是用 IR 来表示。

**Function**，Function 是 LLVM JIT 操作的基本单位。Function 被 Module 所包含。LLVM 的 Function 包含函数名、函数的返回值和参数类型。Function 内部则包含 BasicBlock。

**BasicBlock**，BasicBlock与编译技术中常见的基本块 (basic block) 的概念是一致的。BasicBlock 必须以跳转指令结尾。

**Instruction**，Instruction就是 LLVM IR 的最基本单位。Instruction 被包含在 **BasicBlock 中。

**ExecutionEngine**，ExecutionEngine 是用来运行 IR 的。运行IR有两种方式：解释运行和 JIT 生成机器码运行。相应的 ExecutionEngine 就有两种：Interpreter 和 JIT。ExecutionEngine 的类型可以在创建 ExecutionEngine 时指定。

而一个典型的创建过程便是：

1. 创建一个 Module
2. 在 Module 中添加 Function
3. 在 Function 中添加 BasicBlock
4. 在 BasicBlock 中添加指令
5. 创建一个 ExecutionEngine
6. 使用 ExecutionEngine 来运行 IR

嗯，剩下的东西就没有那么复杂了。

### Rust LLVM hello, world

首先，添加一下依赖：

```
[dependencies]
inkwell = { git = "https://github.com/TheDan64/inkwell", branch = "master", features = ["llvm10-0"] }
```

然后，复制一下我写的 hello, world 代码：https://github.com/phodal/rust-llvm-practises/blob/main/helloworld/src/main.rs 。由于 Inkwell 文档上没有，所以我好不容易参考了一下 C++ 的版本，然后抄了过来。

对应上面的逻辑便是：

```rust
    let context = Context::create();
    let module = context.create_module("repl");
    let builder = context.create_builder();

    Compiler::new(&context, &builder, &module);
```

然后在 Compiler 里写上对应的方法：

```rust
        let i32_type = self.context.i32_type();
        let function_type = i32_type.fn_type(&[], false);

        let function = self.module.add_function("main", function_type, None);
        let basic_block = self.context.append_basic_block(function, "entrypoint");

        self.builder.position_at_end(basic_block);

        let i32_type = self.emit_printf_call(&"hello, world!\n", "hello");
        self.builder
            .build_return(Some(&i32_type.const_int(0, false)));

        let _result = self.module.print_to_file("main.ll");

        self.execute()
```

编写过程中，困扰我比较多的一点是：如何调用 `print` 方法来输出。最后没有想到是这么的简单：

```rust
        let printf = self
            .module
            .add_function("puts", printf_type, Some(Linkage::External));
```

最后构建：

```bash
LLVM_SYS_100_PREFIX=$HOME/llvm/llvm-10.0.1.src/build cargo build
LLVM_SYS_100_PREFIX=$HOME/llvm/llvm-10.0.1.src/build cargo run
```

