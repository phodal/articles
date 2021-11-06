# Rust OO：多态与继承

> 学习编程语言的最好方式最反复练习。


最近在用 Rust 重写 VSCode-Textmate 库：[scie](https://github.com/phodal/scie/)。原有的代码中，大量地使用了 OO 相关的东西，而 Rust 要实现 OO 也需要一些奇技淫巧，而我本身对 Rust 也不是非常熟练，所以我写了这一篇笔记来记录如何实现一个复杂的 OO 场景。

至于 Rust 是不是 OO 语言，这个场景我不擅长讨论，我只是需要一个使用 Rust 实现 OO 的场景。不过呢，引用 Rust 官方文档对于 Rust OO 的介绍中，的 GOF 的引用部分：

> 面向对象的程序是由对象组成的。一个对象包含数据和操作这些数据的过程。这些过程通常被称为方法或操作。

PS：我使用 Rust 的时间并不长，里面一些对于相关的描述并非非常准确，如果有不合理地地方，欢迎读者指正。

在这个场景下，主要是通过：

1. Struct 共享数据结构
2. Trait 定义行为
3. Enum 获取实例

简单来说，就是要在 Rust 的继承里，类要被拆分两部分 struct 和 trait，即数据和行为。当我们需要值的时候，从 struct 中获取，当我们需要方法调用、使用时，使用 trait。

## 场景说明

这个场景之下，我们一些语法规则，如（懒得打了，复制一下）：

```rust
pub enum RuleEnum {
    BeginEndRule(BeginEndRule),
    BeginWhileRule(BeginWhileRule),
    CaptureRule(CaptureRule),
    MatchRule(MatchRule),
    EmptyRule(EmptyRule),
    IncludeOnlyRule(IncludeOnlyRule),
}
```

他们都需要相同的数据结构 `Rule` 和相同的行为 `AbstractRule`。然后在消费这些 Rule 的过程中，需要根据条件取到 Rule 实例，并调用特定的方法。

## Struct 共享数据结构

首先，来看一下我们的 Rule `struct`：

```rust
#[derive(Clone, Debug, Serialize)]
pub struct Rule {
    pub _type: String,
    pub _location: Option<ILocation>,
    pub id: i32,
    pub _name: Option<String>,
    pub _content_name: Option<String>,
}
```

对应的每个 Rule 实现，都会使用这个 Rule struct 以实现数据共享：

```rust
#[derive(Clone, Debug, Serialize)]
pub struct BeginEndRule {
    #[serde(flatten)]
    pub rule: Rule,
    pub _begin: RegExpSource,
    pub begin_captures: Vec<Box<dyn AbstractRule>>,
    ...
}
```

这里的 `#[serde(flatten)]` 是用于 JSON 序列化的时候，进行扁平化操作。

嗯，就这一点上来说，这里并没有什么特定之处。

## Trait 定义行为

接着，我们使用 Trait 来实现的行为：

```rust
pub trait AbstractRule: DynClone + erased_serde::Serialize {
    fn id(&self) -> i32;
    fn type_of(&self) -> String;
    fn display(&self) -> String {
        String::from("AbstractRule")
    }
    fn get_rule(&self) -> Rule;
    fn get_rule_instance(&self) -> RuleEnum;
		...
```

为了实现子类的可复制和可序列化，我使用了 `dyn-clone` 和 `erased_serde` 来实现对应的操作。然后，先让我们看个获取 id() 的接口的实现：

```rust
impl AbstractRule for BeginEndRule {
    fn id(&self) -> i32 {
        self.rule.id
    }
    fn type_of(&self) -> String {
        String::from(self.rule.clone()._type)
    }
    fn get_rule(&self) -> Rule {
        self.rule.clone()
    }
```

嗯，对，我们就是直接去 Rule 里面的值。

所以，其实继承就是这么简单。

## Enum 获取实例

随后，在我们的开发过程中，还需要获取真正的 Rule 实现，所以：

1.我们可以定义一个 Rule 相关的枚举：

```rust
#[derive(Clone, Debug, Serialize)]
pub struct Rule {
    pub _type: String,
    pub _location: Option<ILocation>,
    pub id: i32,
    pub _name: Option<String>,
    pub _content_name: Option<String>,
}
```

2.在 Trait 中创建接口来返回对应的枚举值：

```rust
fn get_rule_instance(&self) -> RuleEnum {
    RuleEnum::BeginEndRule(self.clone())
}
```

3.使用它们：

```rust
match capture_rule.get_rule_instance() {
    RuleEnum::CaptureRule(capture) => {

    }
    _ => {}
}
```

嗯，流程大概就是这样了。

