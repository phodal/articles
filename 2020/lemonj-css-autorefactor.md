# Lemonj：CSS 的自动化重构工具

最近，在帮助一家大型公司的大型前端项目设计和构建前端体系，期间和我同事 @Liuuy 讨论起了 CSS 的架构和设计问题。开发人员对于 CSS 及其 CSS 预处理器的使用是一个很有意思的问题 —— 过去，我一直在吐槽这些想写好 CSS 的人，却是一点儿也不想高认真学习 CSS。

于是，在我们的讨论之下，我借助了在编写 Coca 的经验，设计和验证了自动化重构的可能性。由我的同事完成了 TypeScript 的 CSS 自动化重构工具：Lemonj —— 名字是我取的 🍋🐔。

## Lemonj

GitHub：[https://github.com/twfe/lemonj](https://github.com/twfe/lemonj)

Lemonj 是一个面向 CSS/LESS/SCSS 的分析、坏味道检查和自动化重构工具。 

从架构上来说，它就是通过  Antlr + Antlr4TS 生成对应的解析器生成器，而后根据需要处理所需的字段对相应的内容进行标记。随后，根据我们的需要和设计，通过 AST 中的位置信息，修改对应的内容值。

如 `importants` 数量分析中的代码：

```typescript
if (Checker.hasImportant(propertyValue)) {
	if (!this.metadata.importants) {
		this.metadata.importants = [];
	}

	this.metadata.importants.push({
		type: '!important',
		file: this.metadata.filePath,
		line: ctx._start.line,
	});
}
```

它便是获取 CSS 属性中的内容，检查是否有 !important，然后记录一下位置信息。

### 与 CSS 转换器的不同之处

或许你也用过各类的 CSS/LESS/SAAS 转换工具，所以会好奇它们与 Lemonj 的相似与不同之处在哪里。

**CSS 转 CSS 预处理器转换工具**。它们都是一键式的上传一个类 CSS 文件，从中提取语法树，转换到新的形式上。而要实现不同预处理器的转换，你可能还需要多个转换工具。而且它们只能在一个文件上修改，而你的代码是分散在代码库中。

**Lemonj 自动化重构 CSS 工具**。也是分析语法树，从中提取文件的信息，但不能直接转换到新的形式上。而是要分析出代码中的问题，结合 AST 中获取到的位置信息，再回到指定的文件中对它们进行自动化修改。

你可以认为一个是在现有的代码基本上自动修改，一个是则是普通的转换工具 —— 毕竟根据位置修改是一个累人的工作。

## 使用 Lemonj 进行 CSS 自动化重构

1.先安装 Lemonj，即：`npm install lemonj -g` 又或者是：`yarn global add lemonj`

2.使用 Lemonj 分析下代码：`lemonj analysis _fixtures`，这里的 `_fixtures` 是个目录。

过程中，会生成对应的坏味道数据：

```
Code Smell:  {
  colors: 24,
  importants: 4,
  issues: 8,
  mediaQueries: 1,
  absolute: 0,
  oddWidth: 1
}
```

因为坏味道功能太普通了，所以我们暂时没有增强，并非我们的核心竞争力所在。除此，在执行 `anlaysis` 的时候，还会生成一些额外的信息，比如颜色的 mapping 文件：

```css
// _fixtures/less/color/border.less
@color1: #ddd;
// _fixtures/less/color/border.less
@color2: green;
// _fixtures/less/color/rgba.less
@color3: rgba(255, 0, 0, 0.3);
// _fixtures/less/color/sample.less
@color4: #ff7f50;
// _fixtures/less/color/sample.less
// _fixtures/less/color/sample2.less
@color5: #800080;
// _fixtures/less/color/sample.less
```

这里的颜色只是暂时的命名，需要根据具体的需要进行调整，如按设计系统的思想，将 `color_1` 改成 `@primary_color`。

3.执行 `lemonj refactor _fixtures` 对代码进行自动化重构。就能将上一步中的代码，进一步地修改到所有的代码文件中。

嗯，重构就是如此的简单。

## 其它

Charj 的功能在完善中，欢迎大家看看你们有哪些场景适合自动化重构。

最后，记得我们的 GitHub：https://github.com/twfe/lemonj 



