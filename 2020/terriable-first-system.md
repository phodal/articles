# 让第一个版本的系统混乱一点，或许是件好事

最近，我在设计、开发、维护一个基于『文档代码化』思想的平台。因为丰富的 markdown 经验和文档化系统的设计经验，我在这个系统中实施了很多过去的一些想法。系统工作得很好，但是代码却显得一片混乱。而，我突然觉得这是一件好事。

## 最佳实践很浪费时间

对于敏捷开发来说，我只采纳了持续集成和持续部署的思想，即提交代码便发布到 GitHub Pages。但是，这也浪费了我很多的时间，而且我觉得没有必要，因为我已经有一个本地可以部署的脚本。我只需要在本地运行一下 ``deploy``，那么就会在本地构建，并部署到服务器上。然而，为了最佳实践的理念，我还是花了半天的时间，研究了一下 GitHub Action，然后让它实现自动部署。

系统的 UI 采用的于 Angular 框架，因为我懒得搭建脚手架，而且我还有一些先前的代码可以复用。所以，我 copy / paste 大量的代码，这些代码大部分都是没有测试覆盖的。是的，你很少看到我的开源项目是没有测试覆盖的 —— 毕竟写单元测试也是要花时间的。过去，我们统计过一个相关的数据，在我们日常的开发中，我们差不多有 1/5 的时间花在了单元测试。所以，一周下来，我差不多一天的时间在写测试这件事上。

## 一个问题，三种方法实现

如开头所说，整个系统的核心是一个基于 markdown 的多功能渲染引擎。这部分的组件可以让你用 markdown：

1. 画出条形图、雷达图、思维导图
2. 画出甘特图
3. 画出特定的四象限
4. 调用 Web Components
5. ……

而为了实现这个功能，一共有三套不同的机制，当然了对于写 markdown 的人来说，它们是无感知的。这三种方法分别有：
 
1. 创建占位符，渲染完成后，替换占位符
2. 直接生成最后要渲染的 HTML
3. 生成一个 ID，在渲染的过程中，根据 ID 替换元素。

所以，整个过程就相当于，是解决一个问题有三个方法，然后我都用了这三种方法。

## 起初，我只想创建个原型

起初，我只是想创建一个简单的系统，它只是一个简单的原型。而后，随着越来越多想法的产出，我创建了一个足够复杂的系统。所以，我起初设计的一系列要素都失效了。

我还有一堆糟糕的 SCSS 要管理，因为第一个版本设计的 CSS 体系，无法适用于新的架构。整个系统围绕在 markdown render 上，而这个 render 有大量的样式，就好像早期的多核 CPU 架构，只有一个 CPU 在工作，毕竟有大量的工作。

随着系统越来越复杂，我开始需要一个文档系统来管理这个文档系统，以便告诉我自己：系统里有什么功能？

## 一片混乱，真的很爽

是的，现在，现在虽然看上去界面很美观，功能也很强大。就好多是我们看到的其它软件系统一样，但是内部真的是一片混乱。如果你习惯了这样的系统的代码，那么你可能觉得这不是一个问题。

但是，我习惯了在项目中引用各种最佳实践。看了这样一个系统，觉得非常的爽 —— 大部分系统就是这样的：**同样的时间压力之下，我做得也就那样**。虽然，我的手速可能比大部分人还快，实现的功能可能更多。但是，这些都是无关紧要的。

如果没有充足的时间改善，我们的系统都变得一片混乱。没有符合未来变化的设计，更何况你可能没有时间设计。

## 好了，是时候重构

所以，经历了一系列的过程之后，我决定挑个合适的时候重建这个系统。

1. 开始重写之前的 markdown converter。
2. 将 markdown render 所有相关的组件提取为框架。
3. 重写系统。

于是，我们就出现了第二个系统，它设计良好。它的灵感都来自于我们在做第一个系统中的感受。如果没有来自于我们第一个系统的经验，我们无法设计出第二个系统。

## 经验：快速验证概念，创造业务价值

事实上，我们在市面上看到的大部分系统，都是以如此的方式演进的：

 - 第一个系统，赚了钱，创造了价值，但是缺少各种最佳实践，生存周期短。
 - 第二个系统，设计良好，包含了各种实践，生存周期变长，但是慢慢变得臃肿

而我们的第二个系统很快将变成一个臃肿而缓慢的系统。

> 『第三个系统』由那些为 『第二个系统』 所累的人们创建。 —— 《Linux/Unix设计思想》

没有新系统，哪来的 KPI？

感谢您的支持，祝您生活愉快！