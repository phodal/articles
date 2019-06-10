# 【优质代码范式】：IDE 重构

> IDE vs Editor

重构，指对软件代码做任何更动，以增加可读性或者简化结构，而不影响输出结果。可是我们要如何才能不影响输出结果呢？？？答案是：测试。测试的意义在于对输出结果进行测试，用于保障现有代码的功能是正常的。一旦我们修改了代码，导致测试失败了，那我们就知道哪里改错了。

因此重构依赖于单元测试和可测试的代码（即短小、可 mock 的代码）。在重构之前，对应的代码拥有测试是信心的保证。可由于种种情况，我们的代码中不存在测试，我们就放弃代码重构吗？

所以，这篇文章讲述地是，如何在不包含测试的情况下，来对代码进行重构——IDE 重构。需要注意的是，IDE 重构在大部分的情况下是不会影响输出结果的，但是在极少数的情况下可能会影响结果——比如 IDE 出 bug 了。hiahiahiahia~

## 范式：IDE 重构

IDE 重构，即借助于 IDE 的分析功能和提示，来对代码进行重构。它将重构的主要工作交给 IDE，而不是由开发人员痛苦的修改代码来完成。多数时候，我们只需要**按下快速键**，再输入一些必要的名称、参数，重构就完成了。因此，它是每天我们的开发中，不可获缺的一部分。可以随时随地进行，而不需要预留一个专用的重构时间。

### 自然而然发生

那些，我们视为腐烂的代码，并不是在第一天就如此。在最初设计的时候，它们往往有着好的代码结构，良好的设计规范。只是随着时间的推移，业务不断地增加，代码量便不断地增加，缺乏有效的代码改善机制，便会使得代码腐烂。但是，我们也不能提前对代码进行重构，那样的话就是过度设计。

重构，应该是在编写代码的过程中，自然而然发生的。而 IDE 重构也是如此，编写代码的过程中，看出不合适的地方，按下**IDE 的快速键**，迅速地完成战斗。

如下是微前端 Mooa 框架中，最初某个函数的代码的一部分：

```javascript
customEvent('mooa.routing.change', {
  url: eventArguments.url,
  app: activeApp['appConfig']
})
```

而随着业务的演进，代码便进一步便得复杂：

```javascript
if (activeApp.mode === 'iframe') {
  ...
  if (iframeEl && iframeEl.contentWindow) {
    ...
    contentWindow.dispatchEvent(customEvent(MOOA_EVENT.ROUTING_CHANGE, eventArgs))
  }
} else {
  customEvent(MOOA_EVENT.ROUTING_CHANGE, eventArgs)
}
```

一旦，我们添加了一个新的条件，我们的代码会进一步复杂化。而在那之前，对于 if 语句而言，我的做法是**什么也不做**，因为无法预料它的演变。一来，这样的 if 语句没有多少演进的方式，提供重构成多态则进一步复杂代码。二来，当遇到新的需求变更时，原先的重构可能等于白做。

但是在过程中，做了两件事：
 
  - 提取了公有的变量
  - 提取了公有的参数

而这些都可以通过 IDE 的提取变量的快捷键来完成。

### 从容不迫应对

同理，对于那些可以

### 最小犯错成本

构建最小的犯错成本 

### 缺点：有限的代码改善

如下的代码，也是随着

```javascript
function openFile(willLoadFile: string, isTempFile: boolean = false) {
  let imageRegex = /\.(jpe?g|png|gif|bmp|ico)$/i;
  let htmlRegex = /\.(html)$/i;
  let wordRegex = /\.(doc?x)$/i;

  if (imageRegex.test(willLoadFile)) {
    return mainWindow.previewFile(willLoadFile);
  } else if (htmlRegex.test(willLoadFile)) {
    return openHtmlPage(BrowserWindow, willLoadFile);
  } else if (wordRegex.test(willLoadFile)) {
    return shell.openItem(willLoadFile);
  }
}
```

## 工具

### IDE

ThoughtWorks -> 购买 IDE -> Jetbrians -> 真的是生产力工具

### Editor + 重构插件

在 Visual Studio Code 中提供了一些对应的重构插件。最常用的几个，如

 - 提取变量
 - 提取方法
 - 重命名

![TypeScript 提取变量](ts-extract-local.gif)

诸如 JavaScript 这一类的动态语言，则会应了那句话：“动态类型一时爽，代码重构火葬场”。因为各式的智能提示，都依赖于对语言的静态类型检查，但是动态类型往往很难实现这一点。使用 TypeScript 对于可维护的代码是一个可好的选择。

## 示例

### 重命名

Shift + F6

### 提取...

 - 变量：Alt + Command + V
 - 静态变量：Alt + Command +  C
 - 函数：Alt + Command +  M

