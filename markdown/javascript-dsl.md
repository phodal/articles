如何在业务代码中提升：创建领域特定语言
===

最近的一些日子里，又陷入了平凡、无聊、繁琐的业务代码开发中，生活变得无比的枯燥。每天面对着大量重复、而又没有办法得胜的代码，总会陷入忧虑之中。

而在实现几个重复的业务代码时，我发现了一个更好的方式，使用**领域特定语言**。

最初，我是在设计一个工作流的时候，发现自己正在使用 DSL 来解决问题。因为这是一系列重复而又繁琐的工作，所以便想着抽象出一个服务来专门做这样的事情。

 - 第一个版本里，我使用了 ``->`` 操作符来实现一个简单的 DSL：``operate -> approve -> done``。在使用的时候，我只需要传相应的数据即可。
 - 第二个版本里，我意识到并不需要这么复杂，JavaScript object 拥有更强的语言表达能力。我只需要传递对应的对象过去即可，再通过 ``Object.keys`` 就可以获取处理的顺序。

于是，我就这么将一个高大上的 DSL，变成了一个数据结构了。我一想好像不太对，JavaScript 的 ``object`` 不仅仅只是数据结构，它可以将方法作为对象中的值。随后，我又找到了之前写的一个表单验证的类，也使用了类似的实现。这种动态语言特有的数据结构，也可以视之为一种特定的 DSL。

便想着写一篇文章来介绍一下业务代码中的 DSL。

DSL 简介
---

不过，在开始之前，相信有很多人都不知道 DSL 是什么东西？

> DSL，即领域特定语言，它是一种为解决**特定领域问题**，而对某个特定领域操作和概念进行抽象的语言。

在深入了解之前，先让我们了解 DSL 的两个大的分类：

 - 外部 DSL，即创建一个**专用目的**的编程语言。诸如用于 BDD 测试的 ``Cucumber`` 也是一种外部 DSL，从某种意义上来说，我用于写作的 ``markdown`` 也算是一种 DSL。它们通常都需要语法解析器来进行语法分析，并且通常可以在不同的语言、平台上实现。
 - 内部 DSL，即：指与项目中使用的通用目的编程语言（Java、C#或Ruby）紧密相关的一类 DSL。它是基于通用编程语言实现的，由它来处理复杂的基础设施和操作。[^DSL]

[^DSL]: http://www.infoq.com/cn/articles/External-DSL-Vaughn-Vernon

依这种定义而言，使用 JavaScript object 来实现这一类的方式，应该归类于内部 DSL。在我写这篇文章的时候，我总算找到了一个相关 “数据结构 DSL” 相关的介绍：

> [数据结构 DSL](https://blog.hellojs.org/declarative-programming-is-it-a-real-thing-e59fe5e893fd) 是一种使用编程语言的数据结构构建的 DSL。其核心思想是，使用可用的基本数据结构，例如字符串、数字、数组、对象和函数，并将它们结合起来以创建抽象来处理特定的领域。

而，就实现难度而言：

```
数据结构 DSL < 内部 DSL < 外部 DSL < 语言工作台
```

这里的数据结构 DSL，更像是一种**内置函数的配置文件**。代码，读的时候远多写的时候多。一行配置与十行代码相比，自然是一行配置更容易阅读。所以，使用 object 是一种更容易的选择。

接着，让我愉快地展开这些 DSL 的使用历程吧。

难以构建的外部 DSL
---

某些外部 DSL，看上去已经可以说是一门编程语言了，它也可以编译为可执行的程序，也可以是边运行边解释，类似于解释型语言。不过，它通常是作为程序的一部分存在的，如 Emacs Lisp，可以编译为程序，但是多数时候是作为 Emacs 的一部分而存在。

这算得上是一种复杂的 DSL，而简单的外部 DSL，而诸如我们平时开发用的前端模板：

```html
<View style={{ flexDirection: 'row' }}>
  <Icon style={{ marginLeft: 5, marginRight: 5 }} name={'ios-chatboxes-outline'} type={'ionicon'} color={'#333'} />
  <Text>{topic.attributes.commentsCount}</Text>
</View>
```

对于这样一个模板来说，我们要做的就是使用 JavaScript 实现一个解析器。在构建的时候，将其编译为 JavaScript 代码；在运行的时候，再将其转换为 HTML。

以我几次、有限的创建 DSL 的经历来说，诸如：[stepping](https://github.com/phodal/stepping)，我觉得外部 DSL 并不容易实现——虽然已经有了 Flex 和 Bison（在 JavaScript 世界里，有一个名为 Jison 的实现）这样的工具。其相当于是自己写一个编程语言，与此同时设计出一个容易使用的语法。

如我之前设计用于 DDD 的 ``stepping`` 看上去就像是一个配置文件，而我是使用 Jison 写了自己的语法分析：

```
domain: 库存子域
  aggregate: 库存
    event: 库存已增加
    event: 库存已恢复
    event: 库存已扣减
    event: 库存已锁定
    command: 编辑库存
```

Whatever，要实现这样一个 DSL 并不是一件容易的事。就目前而言，使用最广泛的 DSL，恐怕要数 ``markdown`` 了？

当然了，对于大的项目，或者大的组织团队来说，要实现这样一个 DSL 并不是问题。它也有利于组织内部的沟通，DSL 在这里就像是一个领域知识的存在。

而就使用习惯来说，更常见的是内部 DSL。

易于实现的内部 DSL
---

内部 DSL，通常由编程语言内部来实现，一种常见的实现方式就是：**流畅（fluent）接口**。如，jQuery 就是这种内部 DSL 的典型的例子。

```javascript
$('.mydiv')
  .addClass('flash') 
  .draggable()
  .css('color', 'blue')
```

内部 DSL 是在一门现成语言内，实现针对领域问题的描述。如上述代码中的 jQuery 语法就是专用于 DOM 处理的，它的 API 也就是其最出名的``链式方法调用``。

如下，也是一种内部 DSL 的实现：

```
var query =
  SQL('select name, desc from widgets')
   .WHERE('price < ', $(params.max_price), AND,
          'clearance = ', $(params.clearance))
   .ORDERBY('name asc');
```

而对于我们实现来说，则可能是：

```
function SQL (param) {
    this.WHERE = function(){
        return this;
    };
    this.ORDERBY = function(){
        return this;
    };
	return this;
}
```

这种 DSL 专门针对的是开发人员的使用，对于复杂、重复应用来说，它特别有帮助。可以设计出专用于业务的 DSL。

可问题来了，在前端领域的业务代码里，要实现这样一个 DSL 的机会并不大——一个合理的项目来说，复杂的业务逻辑应该由 BFF 层实现，内部 DSL 更常见于框架的 API 设计上。除非，我们在设计一个框架，诸如 Jasmine，这样的测试框架：

```
const simDescribe = (desc: any, fn: any) => {
  console.log(desc)
  fn()
}

const simIt = (msg: any, fn: any) => {
  simDescribe('  ' + msg, fn)
}

...

export const SimTest = {
  describe: simDescribe,
  expect: simExpect,
  ...
}
```

PS：上述的简化代码见：[https://github.com/phodal/oadsl](https://github.com/phodal/oadsl)

在业务复杂的情况下，则可以有针对性的设计出这样的 API。

从外部 DSL 到内部 DSL 工作流
---

我喜欢 JavaScript、Python 这一类动态语言，是因为其拥有优秀的**语言表达力**。而 JavaScript 这门语言在一点上，那便更为突出。JSON 和 JavaScript Object 可以帮助我们快速地创建这样的一个 DSL。

最初，我产生了一个 DSL 的想法是因为：Angular 框架的动画形式的：``void => inactive``，或者是 ``inactive => active`` 的形式。这让我联想到了一个工作流可以这么设计：

```
process = 'transact -> approve -> bank';
```

对应的，我们只需要写相应的数据即可：

```
[{
    name: 'transact',
    icon: 'success'
},{
    name: 'approve',
    icon: 'processing'
},{
    name: 'bank',
    icon: 'todo'
}]
```

（PS：现在看来除了帮助我写文章，它的意义并没有那么重要。）

但是这样的 DSL，并不容易使用。为了使用它，我们需要一个数据，一个流程，两个参数。而我们面向的是开发人员，越简单地 API 也就越容易使用。而 JavaScript 里的 object 正好可以起一个顺序的作用，我们保需要使用 ``Object.keys`` 就可以获取到对应的值。其对应的实现也比较简单（简化版本）：

```javascript
export function workflowParser(data: any) {
  const keys = Object.keys(data)
  const results = []
  for (let key of keys) {
    let process = data[key] as IWorkflow
    results.push({
      name: process.name,
      status: process.status,
      icon: `icon-${process.status}`
    })
  }

  return results
}
```

对应的我们只需要一个参数：

```
transact: {...},
approve: {...},
bank: {...}
```

于是，一个有点复杂的 DSL 就变成了一个 Object。而更像是一个 JSON，随后我们只需要定义好一系列的流程，然后获取即可:

```
<process data={{WorkflowMap.SUCCESS}}></process>
```

这样一来，我们就将复杂度转移到了组件 process 内部了。

JSON 到数据结构 DSL
---

与 JSON 相比，JavaScript Object 有一点相当的迷人，即可以支持使用函数。

除了组件上的重用，还有一种常见的例子就是：**表单验证**。表单验证是一种相当繁琐的工作，我们也可以看到一系列相应的 DSL 实现。如下是一个用于表单验证的 DSL：

```javascript
const LoginFormValidateMap = {
  phone: {
    require: true,
    regular: RegexMap.phone
  },
  country: {
    requireBy: 'phone'
  },
  email: {
    requireByNot: {
      country: 'CN'
    }
  }
}
```

它与 JSON 形式不同的是，我们可以动态修改对象中的值，传入函数。其实现与 JSON 的示例来说，也一样的简单。我们就只需要遍历这些值即可：

```javascript
export function FormValidator(validateMap: any, data: any) {
  let validateKeys = Object.keys(validateMap)
  for (const key of validateKeys) {
    const map = validateMap[key] as IValidate

    if (map.require) {
      if (!data[key]) {
        return {
          key: key,
          error: VALIDATE_ERROR.REQUIRE
        }
      }
    }
    ...
  }
}
```

然后，就可以验证字段是否有错：

```
const data = {
  phone: '1234567980',
  country: 'US',
  email: ''
}

let result = FormValidator(LoginFormValidateMap, data)
```

上述的实现是为了解析方便。一个更加 DSL 的实现，应该是：

```
const methods = [
  ['不能为空', isNotEmpty],
  ['不得长于', isNotLongerThan]
]
```

然后，我们只需要对应于我们的错误信息，写一个 ``${key} 不能为空`` 即可。

结论
---

如我们所看到的，要实现这样一个 DSL 并不困难。因为难的并不是去做这样的设计，而是这种保持设计的思维。随后，不断的练习掌握好如何去设计一个 DSL。

当下次我们遇到这样的场景时，是否会想：有没有更好的实现方法？

如果有更充裕的时间，我想设计一些更优雅、容易使用的 DSL：[https://github.com/phodal/oadsl](https://github.com/phodal/oadsl)


