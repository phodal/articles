# 前端的自动化重构

过去，我一直想着抽时间写一个小的前端工具，以对代码进行自动化的重构。但是呢，经过我再三的考虑，我暂时取消了这个打算 —— 主要是没时间。（PS：人生长乐，写个 Charj） 但是呢，我打算写一篇文章记录一下相关的思路。

原因依据很多：

1. 大部分国内的公司使用的都是 Vue，template、script、style 都耦合在一起；
2. 大量的前端项目都是轻逻辑，不具有复杂的业务场景
3. 前端系统被重写的频率太快了
4. JavaSript 语法太灵活，而 TypeScript 还未普及
5. ……

简单来说，在缺乏复杂场景的情况下，我不太想去写这样的工具。

## 如何构建前端自动化重构工具？

在我之前写的那篇『[重构的自动化](https://www.phodal.com/blog/auto-refactor/)』中，介绍了如何去做这样的工具：

1. 构建特定语言的语法解析器。
2. 设定代码坏味道的内容及标准。
3. 针对于每一项坏味道，编写识别代码。
4. 编写代码坏味道的建议改进和实施代码。
5. 实现坏味道的自动化重构。

以 Vue 为例，这个过程便是：

1. 寻找适用于 Vue 的 AST 生成工具。如 `eslint-vue-parser`
2. 寻找和编写适用于 Vue 编码的相关规范。
3. 对应规范寻找代码中的问题。
4. 针对该问题寻找改进点
5. 实现自动化重构

让我们来看个简单的示例，如我们的代码规范中，针对于**组件库**强制规范了一定要写 `scoped`。而我们有大量的组件都没有相应的实践。这个时候，就可以通过这种方式来处理。分析中代码中不带 `scoped` 的 style，然后自动添加：

```
<style scoped>
</style>
```

添加的模式其实也比较简单：

1. 解析后，AST 将带有标签等等的位置信息。
2. 针对所有相关类型的文件进行识别，记录所需要重构的相关信息。`file`、`location`、`changed`、`length`。
3. **反向**遍历所有的待修改处，读取对应的文件，对应的位置，进行修改。
4. 保存文件。
5. 再次运行。

嗯，就是这么简单。

## 配套工具

根据我先前的一些调研，我整理了一些相关的资料，欢迎大家去玩。

### JavaScript

如果只是针对于简单的 JavaScript 重构来说，我们可以考虑使用 `jscodeshift` 这一类的工具。[jscodeshift](https://github.com/facebook/jscodeshift) 是一个工具包，用于在多个 JavaScript 或TypeScript 文件上运行 codemods（自动代码修改）。

当然了，如果你不嫌麻烦的话，还可以使用类似的工具：

| Source | Esprima 4.0.1 | [UglifyJS2](https://github.com/mishoo/UglifyJS2) | [Traceur](https://github.com/google/traceur-compiler) | [Acorn](https://github.com/marijnh/acorn) 8.0.4 | [Shift](https://github.com/shapesecurity/shift-parser-js) | [Shift (no early errors)](https://github.com/shapesecurity/shift-parser-js) |
| --- | --- | --- | --- | --- | --- | --- |
| jQuery.Mobile 1.4.2 | 149.6 ±1.8% | 170.7 ±1.2% | 178.2 ±6.0% | 214.4 ±13.0% | 429.5 ±13.5% | 203.9 ±9.6% |
| Angular 1.2.5 | 125.0 ±2.8% | 138.2 ±2.9% | 134.5 ±2.3% | 113.8 ±2.8% | 251.5 ±1.3% | 147.1 ±1.5% |
| React 0.13.3 | 127.2 ±1.0% | 158.2 ±1.4% | 160.0 ±0.8% | 128.5 ±2.8% | 310.8 ±2.7% | 182.6 ±2.7% |
| **Total** | 401.8 ms | 467.0 ms | 472.7 ms | 456.7 ms | 991.9 ms | 533.5 ms |

嗯，原理都是相似的。

### TypeScript

官方提供了 AST 解析。

从我的之前写的前端架构守护工具：[https://github.com/phodal/dilay](https://github.com/phodal/dilay)，你就可以看到相似的代码。

### CSS 

针对于 CSS 重构来说，相似的工具有：[https://github.com/csstree/csstree](https://github.com/csstree/csstree)

不过，我们建议你们使用 Lemonj（使用 Antlr 进行语法树解析）：[https://github.com/twfe/lemonj](https://github.com/twfe/lemonj)

### 框架特定

针对于 Angular，官方提供了 Angular Schematics，除了自动代码修改，还可以做各种自动化升级工作。

针对于 Vue，官方也有类似的工具：[https://github.com/vuejs/vue-codemod](https://github.com/vuejs/vue-codemod)

针对于 React，官方也有工具：[https://github.com/reactjs/react-codemod](https://github.com/reactjs/react-codemod)

### 结合 CLI 工具

当我们修改完代码之后，下一步要做的事情就是修改文件，这里推荐一下：`schematics-utilities`，虽然是 Angular 上下游的工具，但是它不限于框架。

有了这个工具，我们就可以快速修改代码，如：

```typescript
recorder = tree.beginUpdate(path);

recorder
	.remove(start, length)
	.insertLeft(start, value);

tree.commitUpdate(recorder);
```

这些都大同小异，没有什么特别之处。

## 总结

嗯，人生苦短，一定要花 1 小时写个工具，解决 10 分钟能完成的事情。


