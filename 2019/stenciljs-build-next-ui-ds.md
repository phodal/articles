# 【前端必知】胶水（框架） Stencil.js

去年的同一时间，我写了那篇《[前端下半场：构建跨框架的 UI 库](https://www.phodal.com/blog/build-cross-framework-ui-library/)》推荐了 Stencil.js，当时是在项目的试验期。而 Stencil.js 已经在今年（2019 ）的 6 月份，推出了 1.0 正式版，这意味着你已经可以开始采用这个~~框架~~（编译器）了——当然了，我不对我的这句话『负责』，你自己的锅自己背。

故事是这么开始的。

上周我在为 @花仲马 ~~复制~~ （开发）一个内容主题生成器 （ https://content.phodal.com ）（受限于其中数据的版权问题，暂时不敢开源——我都是直接复制别的网站的），便开始在寻找一个能用得上的前端框架。毕竟：

 - React 用腻了
 - Angular 用腻了
 - jQuery 可能会被笑话
 - 原生 JavaScript 我不会了

所以，我开始绞尽脑汁找一个新的框架，直到我从 Web Components 框架里又找到了 Stencil.js。

过去，我曾经使用过 Ionic 开发了超过两位数的应用，除了 stars 数上千的 Growth，还有一个交易额 xxxxx 千亿的商业应用。所以，既然是 Ionic Framework 团队开发的框架，那我还是有点信心 ~~担心~~ 的。

## 为什么是 Stencil.js

Ctrl + C/V 一下官方的介绍：Stencil 是一个生成 Web Component 的编译器（更具体地说，Custom Elements）。Stencil 将最流行框架的最佳概念，结合到一个简单的构建时工具中。

它提供了以下的特性。

 - Virtual DOM
 - Async rendering
 - Reactive data-binding
 - TypeScript
 - JSX

为什么说它结合了最流行框架的最佳概念——它的代码看上去就像 Angular + React 的混合体：使用了``装饰器`` 来声明类型，还有类似于 Angular 风格的 Component 声明方式，以及还有 React 的 TSX，还有 Vue 融合了多个框架的思想和能力……。

```typescript
@Component({
  tag: 'my-component',
	styleUrl: 'my-component.css',
	shadow: true
})
export class MyComponent {
  @Prop() first: string;

  private getText(): string {
    return this.first;
  }

  render() {
    return <div>Hello, World! I'm {this.getText()}</div>;
  }
}
```

So，Stencil.js 便是面向未来的集大成者——你们都抄不过我的。

由于使用 Stencil 开发的组件只是 Web Components，所以这些组件可以运行在所有的主流框架（AVR）中，也可以独立地运行。这一特质使它可以成为**新的前端容器框架**——毕竟 Web Components 是一个新的前端容器。

## 用途一：开发前端应用

好吧，没啥说的，大家都懂。

不过呢，Stencil.js 用来开发前端应用的另外一个优势是，已经有一个成熟的组件库：Ionic 系列的组件库。而 Ionic 系列的组件，已经在大量的应用验证过了。

除此，还与有内建了  Service Workers 生成工具。

顺带夸奖一下 Ionic 团队，这一步步下的棋非常棒。先是开发了用于 Angular.js 和 Angular 的 Ionic，然后开发了 Ionic Native，并且让 Ionic 可以支持不同的框架，现在又有了 Stencil.js。

## 用途二：连接混合应用

尽管 Flutter、React Native 已经很流行了，但是对于没有 Native 能力的团队来说，混合应用仍然是一个非常友好的选择。

标题的说法可能不太准确，不过呢，Stencil.js 是 Ionic 框架背后的组件库的支撑框架，它变成了 Ionic 这个混合应用框架的核心之一。

## 用途三：连接其它前端框架

嗯，Web Components 的一大特质，就是让你编写的应用可以嵌入其它框架中。

## 用途四：构建跨框架的 UI 库 / 设计系统

由于 Stencil.js 是一个基于 Web Components 设计的框架，因此由它创建的组件库，可以轻松地在可以引入  Web Component 的浏览器和框架上运行。

Stencil.js 存在的另一大特质是 **按需加载**。基于 Stencil.js 构建的组件库，呈现的结构是：单一组件以单一 chunk/entry 的形式存在。

如下是在 Stencil.js 中引入  Ionic 组件库时，生成的组件与 ``*.entry.js`` 的绑定关系：

```javascript
[[1, "ion-avatar"]]], [{
  "ios": "p-lyuug53j",
  "md": "p-hixhmhon"
}, [[1, "ion-badge", {"color": [1]}]]], [{
  "ios": "p-q6fmpdfd",
  "md": "p-daqtamux"
}, [[0, "ion-card-content"]]], [{
  "ios": "p-xn1drh7m",
  "md": "p-b9b9dqwt"
}
```

以这里的代码为例，如果我们用了 ``ion-avatar``，而且对应的系统是 iOS，那么就会加载 ``p-lyuug53j.entry.js``，如果对应系统是 Android（md，Material Design），那么就会加载 ``p-hixhmhon.entry.js``。

从这等意义上来说，Stencil.js 真的是一个工具链，哈哈。由于遗憾的是当前生成的组件目录结构比较乱——如果你有 100 个组件，那么至少会生成 100 个 entry.js 文件。不过，可能是我没有找到配置的地方。

## 用途五：微前端应用

结合**按需加载** 和**跨框架的模式**，使用 Stencil.js 开发的组件或者是功能模板，就可以解决部分微前端应用之间的依赖重复问题。也因此 Stencil.js 特别适合用于开发微前端框架模式中的**微件化**架构。

我知道你已经从我的新书中，获取到足够多的微前端相关知识了，这里就不详细展开了。

## 结论：胶水框架

Web Components 大法虽好，但是你还是需要一个快速开发  Web Components 的框架/工具。
