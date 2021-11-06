# Rust 性能优化日志（上）

最近的几个月里，一直在编写代码识别引擎 [Scie](https://github.com/phodal/scie/) ，好不容易解决了各种奇怪的问题。随后，在尝试做了一次 benchmark 之后，发现我写了的这么一些 Rust 代码，运行起来的速度非常慢。同样是对一个代码文件的分析，Scie 差不多要 12S 完成，而同样的 Node.js Addons 则只需要 200ms。于是，我开始了我的性能优化之旅。

## Clion Profiler

开始之前，先介绍一下 Clion 包含的相关功能，也就是 Profiler。不论是在 IDE 中的右键，还是在菜单栏中都可以轻松找到。对于我这种系统编程的新手来说，这种 IDE 的工具真的非常好用，笑~。

执行一下：

```
/Users/fdhuang/charj/scie/target/debug/benchmark
native-profiler-starter: waiting for profiler...
native-profiler: starting target executable itself...
TOKENIZING 121220 length using grammar source.js 5322 ms

Process finished with exit code 0
```

然后打开 Profiler，查看对应的调用，诸如于：

|     | Method | Samples |
|----|----------|---------------|
| 92.8% | benchmark`scie_grammar::grammar::grammar::Grammar::tokenize_string `| 4883 |
| 64.7% | benchmark`scie_grammar::grammar::grammar::Grammar::match_rule` |  3158 |
| 8.9%   | benchmark`scie_grammar::grammar::grammar::Grammar::handle_captures` | 434 |

考虑到图形化的程度，建议感兴趣的读者可以自己尝试一下。

然后，我们就可以找到对应的突破点。

## Rust 性能优化

好了，接下来就可以正式开始优化的过程了。

### 测试

重构之前，记得有点测试。

### 使用 lazy_static 延迟初始化正则

Scie 中需要使用了大量的正则表达式，其中有一些需要在结构体中使用。在我最开始的版本中，直接在结构体中初始化。在几千次的调用之下，必然会出现性能问题。于是就需要使用 `lazy_static` 来进行赋值。

```rust
lazy_static! {
    static ref CAPTURING_REGEX_SOURCE: Regex = Regex::new(r"\$(?P<index>\d+)|\$\{(?P<commandIndex>\d+):/(?P<command>downcase|upcase)\}").unwrap();
}
```

事实上，我们就是寻找一种类似于单例的方式。

### 清理不需要的 clone

在编写的过程中，由于懒 + 重复修改的原因，导致了代码中遗留了一些不必备的 `clone`。在 Profiler 视图下，还是能明显地看到一些差异。也因此呢，全局搜索一下，然后删除不需要的。

### 优化 clone 

同样的，在取某些值的时候，直接 `a.clone().b`，而非 `a.b.clone()` 也是一个不错的优化点。虽然，我没有真正测试过是否对性能有影响（没细研过编程的处理逻辑），但是至少舒服了。

### 使用传递引用

由于 Scie 之前参考的是其它语言的代码，所以在实现的时候，也没有以 Rust 的最佳方式来实现。难免就出现一些问题，比如没有传递引用，导致需要大量的 `clone()`。以下以一下简单的字符串作为示例：

```rust
#[derive(Debug, Clone)]
pub struct LineTokens {
    pub _line_text: String
}

impl LineTokens {
    pub fn new(
        _line_text: String,
    ) -> Self {
		    ... 
		}
}
```

我们要做的是 `String` -> `&'str`。而由于生命周期的传染性，我们需要改大量的相关代码，这真的是一个痛苦的过程：

```rust
impl<'a> LineTokens<'a> {
    pub fn new(
        line_text: &'a str,
    ) -> LineTokens<'a> {
        LineTokens {
            _line_text: line_text,
				}
}
```

### 判断后再 `clone`

这个主要是我没发现 Rust 的 Option 有一个 `is_some()` 的接口。所以，一直都是 `let Some(capts) = captures.clone()` 这样的写法。发现之后，就改为了：

```rust
if captures.is_some() {
  let capts = captures.unwrap();
	...
}
```

而在这一些场景之下，根本就不需要使用到的值。

类似的操作还有：

```rust
let begin_captures = desc.begin_captures.clone();
if let None = begin_captures {
		desc.begin_captures = desc.captures.clone()
}
```

改成对应的其它先判断的形式，主要是习惯了其它语言的写法，又或者是它们对性能的要求没有这么高。毕竟，我是复制、粘贴过来的。

### chars 转换成 vec

这又是另外一个对于编程语言不熟悉的问题。

```rust
while pos < length {
		let ch = exp_source.chars().nth(pos).unwrap();
```

所以，只需要先转换为数据，再通过它去取值即可：

```rust
        let chars: Vec<char> = exp_source.chars().collect();
        while pos < length {
            let ch = chars[pos];
```

## 其它 

未完待续！


