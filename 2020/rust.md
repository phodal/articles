# Rust.unwarp()： 编译器驱动开发

用 Rust 来写个应用，这个想法颇久了。之前呢，要么找不到合适的场景，要么觉得 Rust 门槛有些高。直到最近呢，刚好对底层编程有点想法，便想着用这门语言做点东西玩玩。

考虑到，我用这门语言的时间只有一星期多，某些观点和感受并非那么准确。因此，我的观点并不适合作为一份参考材料。

## Rust 是什么？

让我来 copy 一下

> Rust 是由 Mozilla 主导开发的通用、编译型编程语言。设计准则为“安全、并发、实用”，支持函数式、并发式、过程式以及面向对象的编程风格。 Rust 语言原本是 Mozilla 员工 Graydon Hoare 的私人项目，而 Mozilla 于 2009 年开始赞助这个项目，并且在 2010 年首次揭露了它的存在。

顺便加上 MDN 上的介绍：

Rust 是一个全新的开源系统编程语言，由 Mozilla 和社区的义务劳动者创造，它帮助开发者创造高速与安全的应用，同时能享受到现代多核处理器的强大特性。Rust 使用易懂的语法避免了段错误 (segmentation faults) 并保证了线程安全。
 
不仅如此，Rust 提供了零成本抽象，更多语义，内存安全保证，不会发生竞争的线程，基于特性 (trait) 的泛型，模式匹配，类型推导，高效的 C 绑定，和最小运行时大小。

所以，就这个简介来说，这个语言已经差不多有十年的历史了。

## Rust 优点

开始之前，我仍然是要强调一下的，这里的优点是对比我所学的其它语言而言的，如 Go、Java、Kotlin、TypeScript或者其它。

### 底层语言 && 系统编程语言

我正在寻找一门不是那么复杂的底层编程语言，以陪我完成一些更有意思的工作，而且还不需要那种 “指向指针的指针”。就这方面来说，Go 是一门不错的语言，但是没有 OO。哦，不对，对于 Go 来说的，它的 OO 应该是：**Yes and No**。

从某种意义来说，Go 也有类似的潜质，面向的也是类似的人群。然而，从现有的情况来说，Go 更像是面向网络编程，而非系统编程。

### 编译器驱动

我记得我听闻到的一个关于 Rust 的观点是：只要编译成功，基本呢，不会出错。比如烦人的内存泄漏之类的问题（当然还是会有一些的，只是要写出来并不是那么容易）。

编译器内建了强大的纠错功能。它把我们在运行时遇到的问题，提前到了编译时。也因此，相比于其它语言，它可能会降低你的开发速度。

从某中意义上来说，它相当于是内置了一个 Sonarqube、Findbugs 这一类的 SAST（静态应用程序安全测试）工具。并在编译时失败，以强迫你修复潜在的漏洞。

这其实是个缺点，哈哈哈。

### 交叉编译

在 Go 一样，在这一点上远远比 C/C++ 还是优秀。

### 包管理 + 构建

在几个底层语言里，C/C++、Go、Rust 里，几乎只有 Rust 的包管理是好用的。虽然说 Go 也有，但是就是渣啊。为了使用方方便，我基本选择的是拷贝，而不是用 go mod。

与此同时，我们还可以拓展 Cargo 的功能，以进行更多的操作。

==，Go 有构建工具吗？可能是 Makefile 吧

### 和 Web 的无缝结合

是的，作为一个追求跨平台的开发人员，我特别看好 Rust 的两个 Web 相关的方向。

 - 高性能 Web。Rust + WASM，
 - 更高性能地跨平台应用。Rust + Electron + Node.js，结合 Neon Binding，可以编译为 Node.js 的模块，并在 Electron 应用中调用，开发跨平台桌面应用。

在一切都云化之后，它的这一点特质将会越来越重要。

### 编程语言优点

从社区来说，它还有这么一些优点：

 - 优秀的 Macro 宏定义机制
 - 可 OO。基于 Traits 的简洁而强大的范型系统
 - 错误处理。基于 `Option & Result` 的空值和错误处理
 - 防 OOM。基于 Ownership、Borrowing、Lifetime 的内存管理机制

## 缺点

从上手难道上来说，我觉得 Rust 是比 Go 更加复杂的。

### 学习成本 + 处理更多的细节

大抵这是一门系统集成编程语言，对于原先我们使用的那些编程语言来说，原先的这些事都是由 bug 和编译器来体现。于是乎，我们要处理更多的细节。

Rust 的诸多语法，都有些不合直觉。除此，Rust 还有一个功能非常强大的宏（macro）系统。嗯，每多一个特性，就多一点点的复杂度。

### 复杂的所有权机制

Rust 引入了所有权的概念和工具，以在没有垃圾回收机制的前提下保障内存安全。这是一个相当复杂的概念——主要是在其它语言中都没有。

一个非常有意思的例子就是对于字符串的操作。

```rust
let mut owned_string: String = "hello ".to_owned();
let borrowed_string: &str = "world";

owned_string.push_str(borrowed_string);
```

嗯，对，处理一个字符串，你都要小心翼翼的。所以，我还是用 format 来做这样的事情：

```rust
format!("{}{}", owned_string, another_owned_string);
```

### 缺少支持完备的 IDE

Rust 并非没有可用的 IDE，而是还没有像 Intellij IDEA 那么完备的 IDE。

我使用 Clion + Rust 插件来开发应用，但是它并非非常完美 —— 主要是，我依赖于 IDE 来进行重构，以及借助于 IDE 的智能提醒。

## 其它

当前，我正在写的两个 Rust 应用：

[phodal/exemd](https://github.com/phodal/exemd): 一个简单的 Rust 应用，读取 markdown 文件，解析其中的不同编程语言，并执行。

[phodal/stadal](https://github.com/phodal/stadal): 参考（复制） Google 的 xi-editor 的 json RPC  + 进程分离架构而设计的系统监视器。Stadal 使用 Rust 开发核心，使用 Electron 开发界面。

欢迎入坑讨论学习。
