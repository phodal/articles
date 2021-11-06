# 实现编程语言的 IDE 支持

或许是出自于对编写编程语言的兴趣，又或许是对于创建 IDE/编辑器的兴趣，对于『IDE/编辑器是如何提供编程语言的支持』，我充满了兴趣。其中的一个主要原因是，这是每天我们打交道最多的工具，另外一个原因可能是，咦，我们怎么没有国产的 IDE（手动狗头）。

## 编辑器 & IDE

先前，我已经在那篇《[编辑器的自制](https://www.phodal.com/blog/diy-ide-editor/)》中介绍了，怎么去创建一个简单的文本编辑器？这是一个相对简单的问题。对于一个可用的代码编辑器来说，我们对它的基本诉求是：**快速启动** + **语法高亮**，然后能进行基本的文本编辑。不过呢，这是以我角度来看待问题的，我的想法里：一个编辑器，就干好一个编辑器应该做的事情。对于一些开发人员而言，他/她们会配置上强大的各种支持功能，以使它看上去像是一个 IDE。而后呢，它失去了快速启动的能力，或者失去了一部分的快速启动的速度，这便是有些遗憾的。

关于编辑器与 IDE 的这一一点的讨论，似乎会有些偏颇。我自知我是一个 IDE 党，拥有公司提供的 Jetbrains 全家桶。日常我也会使用 Sublime Text、Xi Editor、Vim、VS Code 进行一些快速的文件修改和查找。顺便提一句，尽管过去我是一个 Emacs 粉，但是自我写了自己的 Markdown 编辑器之后，我已经……。好在下一步，我打算做一个自己的代码编辑器，这样一来，也许就不会那么内疚了。或许呢，我已经在实现的路上了。

回到正题上，如果是一个 IDE 的话（以 IDEA 老用户的感受），那么我估摸着需要这么一些功能：

1. 语法高亮
2. 文本编辑
3. **子系统关联与集成**
4. 跳转与引用分析
5. **智能感知**
6. **重构**
7. **快速修复**
8. 语言特性分析
9. 结构化视图
10. ……

PS：仔细一看，诸如于 VSCode  这一类强大的编辑器里，已经内置了大部分的功能，而且它还是免费的。你还只需要一个，不需要启动多个不同的 IDE，还省下了硬盘空间。笑~

不过，总的来说，这些功能都依赖于词法分析，有了这个支持，才能进行其它部分的操作。

## 语法分析

对于开发工具来说，语法分析有几个重要的功能：

 - 语法高亮，是指根据术语类别来显示不同的颜色与字体以增强可读性的一种编辑器特性。 
 - 实现智能感知
 - 实现跳转和引用分析

从我粗糙的调查来看，大致可以分析为四类：

 - 基于正则表达式来实现语法分析
	 - Sublime Text 基于 YAML 形式的正则匹配方式：[Sublime Syntax files](http://www.sublimetext.com/docs/3/syntax.html#include-syntax)
	 - Textmate、VS Code 基于 JSON 的正则匹配方式：[Language Grammars](https://macromates.com/manual/en/language_grammars)
 - 基于语法分析器（如 BNF）生成中间代码
	 - Jetbrins 基于 BNF 生成代码的方式：[Grammar and Parser](https://jetbrains.org/intellij/sdk/docs/tutorials/custom_language_support/grammar_and_parser.html)
 - 自制 DSL 进行语法解析
	 - Vim 基于正则 + 自制 DSL：[Vim documentation: syntax](http://vimdoc.sourceforge.net/htmldoc/syntax.html)、[Rust 示例](https://github.com/rust-lang/rust.vim/blob/master/syntax/rust.vim)
 - 手写解析语法
	 - Eclipse IDE 提供了个 JFace editor，但是似乎是要手写：[FAQ How do I provide syntax coloring in an editor?](https://wiki.eclipse.org/FAQ_How_do_I_provide_syntax_coloring_in_an_editor%3F)
	 - Emacs Mode: [ModeTutorial](https://www.emacswiki.org/emacs/ModeTutorial)

每一类各自有各自的优缺点和编写难度。但是，总的来说，没有一个方式是简单的。

### 正则实现语法分析

对于正则方式来说，不论是 Sublime Text 还是 Textmate 及基于 Textmate 语法规则的 VS Code，它们都有一个显著的缺点：长，如 VCode 的[java.tmLanguage.json](https://github.com/Microsoft/vscode/blob/master/extensions/java/syntaxes/java.tmLanguage.json)，从长度上来说，我看到的这个版本有 1831 行。表达方式也有些繁琐：

```json
"comments": {
	"patterns": [
		{
			"captures": {
				"0": {
					"name": "punctuation.definition.comment.java"
				}
			},
			"match": "/\\*\\*/",
			"name": "comment.block.empty.java"
		},
		{
			"include": "#comments-inline"
		}
	]
},
```

其中还有各种 include 关系等。对于 Sublime Text 也是类似的：

```yaml
  comments:
    - match: /\*\*/
      scope: comment.block.empty.java punctuation.definition.comment.java
    - include: scope:text.html.javadoc
    - include: comments-inline
```

看了看，是不是会怀疑他们建立了语法同盟。

但是呢，yaml 和 json 是一个**编程语言无关**的东西。所以，VS Code 和 Atom 可以基于 Textmate 语法规则，快速建立对于主流语言的词法分析，从而建立了语法高亮的支持。

我们也可以说 BNF 是一种编程语言无关的东西。但是，实际上在我们操作的时候，就会加入一些编程语言特定的要素。

###  语法分析器分析

由于先前编写系统分析工具 [Coca](https://github.com/phodal/coca) 和通用语法分析器 [Chapi](https://github.com/phodal/chapi) ，我对于 BNF 的词法也是颇为上手的——实际上不难。唯一麻烦的地方就是，写完之后，我们要编写代码做一些转换，所以让我们来看看 Jetbrians 插件的示例：

```bnf
      COMMENT          = 'regexp://[^\r\n]*'
      BLOCK_COMMENT    = 'regexp:[/][*][^*]*[*]+([^/*][^*]*[*]+)*[/]'
```

这一点上和 antlr 没有太大的区别：

```bnf
WS:                 [ \t\r\n\u000C]+ -> channel(HIDDEN);
COMMENT:            '/*' .*? '*/'    -> channel(HIDDEN);
LINE_COMMENT:       '//' ~[\r\n]*    -> channel(HIDDEN);
```

然后，就是设计和分析词法了：

```bnf
functionParameters ::=
    LPAREN inputParameters RPAREN outputParameters?
    | IN SUB GT inputParameters
    | outputParameters
```

接着，在 IDEA 里面，我们可以通过这个 BNF 文件生成对应的  Lexer 文件和代码等。对于使用 Antlr 编写的词法来说，Java 部分的代码规模也就在 800 左右。

不过呢，从两者的阅读体验对比来看，显然  BNF 会更加友好一点。

### 自制 DSL 语法解析

颇为遗憾的是，我尚未写过任何的 Vim 插件，好在我还知道 Vim 是如何退出来的。我使用 Vim 作为 git 的 editor，还熟知一些 Vim 编辑的常用快捷键。所以，语法高亮这一部分主要是参考  Vim 的文档编写和代码示例。这里我找到了一个不错的中文翻译：[语法高亮](https://yianwillis.github.io/vimcdoc/doc/syntax.html)

总的来说，语法规则就是：`syn vim关键字 匹配规则`，如：

```vim
syn region rustCommentLine                                                  start="//"                      end="$"   contains=rustTodo,@Spell
syn region rustCommentLineDoc                                               start="//\%(//\@!\|!\)"         end="$"   contains=rustTodo,@Spell
syn region rustCommentLineDocError                                          start="//\%(//\@!\|!\)"         end="$"   contains=rustTodo,@Spell contained
syn region rustCommentBlock             matchgroup=rustCommentBlock         start="/\*\%(!\|\*[*/]\@!\)\@!" end="\*/" contains=rustTodo,rustCommentBlockNest,@Spell
```

看上去依旧是正则匹配，如 Float：

```vim
syn match     rustFloat       display "\<[0-9][0-9_]*\%(\.[0-9][0-9_]*\)\%([eE][+-]\=[0-9_]\+\)\=\(f32\|f64\)\="
syn match     rustFloat       display "\<[0-9][0-9_]*\%(\.[0-9][0-9_]*\)\=\%([eE][+-]\=[0-9_]\+\)\(f32\|f64\)\="
syn match     rustFloat       display "\<[0-9][0-9_]*\%(\.[0-9][0-9_]*\)\=\%([eE][+-]\=[0-9_]\+\)\=\(f32\|f64\)"
```

不过，从算法形式上来说，完胜 Textmate 和 Sublime，毕竟是高级的 DSL。

### 编程语言语法解析

Emacs 的 mode 里包含了对于语法高亮的处理，于是为了这个高亮，我们需要写写 emacs lisp 代码。如：

```lisp
(defvar rust-formatting-macro-opening-re
  "[[:space:]\n]*[({[][[:space:]\n]*"
  "Regular expression to match the opening delimiter of a Rust formatting macro.")

(defvar rust-start-of-string-re
  "\\(?:r#*\\)?\""
  "Regular expression to match the start of a Rust raw string.")
```

对于 Eclipse 来说，这个过程就更加麻烦了。

## 语言的高级支持

在我们实现了开发工具的词法分析接口之后，我们就能按不同的 IDE/编辑器所定义的接口，进行定制了。这是一个繁杂，而又充满挑战的工作。对于不同的工具来说，它们的接口相关也甚多。我也并非都能一一了解 API，所以只能简单的以 IDEA 作为一个示例来展示。主要原因大概有两个：1. 我日常使用的是 Jetbrains 相关的 IDE；2. 我已经有一部分代码了。

### 语法高亮

在进行了复杂的语法分析之后，接着，我们就可以快速进入一个简单的环节，对代码进行高亮。关于高亮的话，我们可以快速进行一个分类：

 - 关键词。即编程语言的关键词，如 C 语言中的 32 个关键词。
 - 标识符。用户定义的字符串，如变量名、结构体名、函数名等等。
 - 特殊词法。
 - 重要的词法。根据需要，可以针对于函数名、静态函数名等进行标识，以提升识别度。

如下是 Go 语言的一些关键词：

```
(defconst go-mode-keywords
  '("break"    "default"     "func"   "interface" "select"
    "case"     "defer"       "go"     "map"       "struct"
    "chan"     "else"        "goto"   "package"   "switch"
    "const"    "fallthrough" "if"     "range"     "type"
    "continue" "for"         "import" "return"    "var")
  "All keywords in the Go language.  Used for font locking.")
```

所以，在这个场景之下，不论是何种的 IDE 又或者是编辑器都可以快速实现。

### 跳转 goto

不同开发工具，有各种的跳转规则，不同的语言也有各自的跳转方式。如 Emacs 的 go-mode 就定义了一系列的跳转：

```lisp
(let ((m (define-prefix-command 'go-goto-map)))
  (define-key m "a" #'go-goto-arguments)
  (define-key m "d" #'go-goto-docstring)
  (define-key m "f" #'go-goto-function)
  (define-key m "i" #'go-goto-imports)
  (define-key m "m" #'go-goto-method-receiver)
  (define-key m "n" #'go-goto-function-name)
  (define-key m "r" #'go-goto-return-values))
```
	
而 IDEA 也提供了一系列接口来实现类似的功能，如：

```
gotoActionAliasMatcher
gotoClassContributor
gotoSymbolContributor
gotoFileContributor
gotoRelatedProvider
```

我们只需要分析光标符所在的位置，其所定义的语法，如 IDEA 里是 PSI，再实现对应的逻辑即可。如：

```
@Override
public @NotNull NavigationItem[] getItemsByName(String name, String pattern, Project project, boolean includeNonProjectItems) {
    List<CharjStructDeclaration> properties = findStructByKey(project, name);
    return properties.toArray(new NavigationItem[properties.size()]);
}
```

这里定义的是数据结构的导航。当我们按下快捷键的时候，会传入 name、pattern 等信息。接着，从所有相关的文件（VirtualFile）中寻找对应的 struct，返回即可。

### 自动填充

主要可以分为两类，一类是：代码段（Snippets），一类是：自动填充（Completion）

好像也没啥说的，就是绑定在特定关键字上的内容。

### 其它

剩下的就是一些比较有意思的功能，诸如于：

 - fileType。文件图标支持。即某一类型的文件，使用特定的图标来展示。
 - commet 。即按下注释的快捷键，能快速的注释和反注释代码。
 - line marker。IDEA 提供的功能，用于在行上通过图标来展示特定的功能。
 - folding。提供特定的代码段的折叠功能。
 - 数据视图。展示特定数据结构关系及参数等的视图。
 - ……

## 其它 

我一直在寻找一直简易的方式，以快速识别编程语言，并标识它们。所以，也就有了这篇文章。

虽然，还在探寻，但是呢，似乎已经有了一个初步的结果。

