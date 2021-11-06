#  Rust + LLVM 调用 C/C++ 模块

在上一篇文章『[LLVM + Rust JIT hello, world](https://www.phodal.com/blog/llvm-rust-hello-world-jit/)』中，我们介绍了如何使用 Rust + LLVM 编写一个 hello, world。而随着我们继续在这个领域的探索，我还想到了一个非常有意思的问题：如何使用 LLVM 调用三方模块。

最后代码见：https://github.com/phodal/rust-llvm-practises/tree/main/stdlib

从最后的结果来看，要实现这样的功能也相当的简单。只是呢，作为一个 LLVM 新手，我要学习的东西还有蛮多的。

## Clang + LLVM 字节码

我们所要做的事情其实还是相对比较简单的：通过 Clang 来编译，并输出 LLVM IR。

> Clang 是一个 C、C++、Objective-C 和 Objective-C++ 编程语言的编译器前端。它的目标是提供一个 GNU 编译器套装的替代品，支持了  GNU 编译器大多数的编译设置以及非官方语言的扩展。

在 `clang` 的编译参数中，提供了一个 `emit-llvm`：


```makefile
CC=clang
CFLAGS=--target=$(TARGET) -emit-llvm -O3 -ffreestanding -fno-builtin -Wall -Wno-unused-function

%.bc: %.c
	$(CC) -c $(CFLAGS) $< -o $@

CHARJ=charj.bc

all: $(CHARJ) 
$(CHARJ): TARGET=x86_64-apple-darwin-macho # macos
```

对应的是一个 C 的 hello, world ：

```c
#include <stdio.h>

int main()
{
    printf("Hello, World!");
    return 0;
}
```

## Rust LLVM 调用 LLVM IR

接下来，就是用 Inkwell + LLVM 的 API， 加载这个字节码了：

```
    let context = Context::create();
    let memory = MemoryBuffer::create_from_memory_range(CHARJ_LIB, "charj");
    let module = Module::parse_bitcode_from_buffer(&memory, &context).unwrap();
```

这里的 `module` 改为采用 `parse_bitcode_from_buffer` 的方式，而不是原来的创建方式。然后我们就可以获取到定义的 main  函数了，然后执行：

```rust
    if let Some(_fun) = module.get_function("main") {
        let ee = module
            .create_jit_execution_engine(OptimizationLevel::None)
            .unwrap();
        let maybe_fn = unsafe { ee.get_function::<unsafe extern "C" fn() -> f64>("main") };

        unsafe {
            maybe_fn.unwrap().call();
        }
    }
```


