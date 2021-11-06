实施微前端的六种方式
===

微前端架构是一种类似于微服务的架构，它将微服务的理念应用于浏览器端，即将 Web 应用由单一的单体应用转变为**多个小型前端应用聚合为一的应用**。

由此带来的变化是，这些前端应用可以**独立运行**、**独立开发**、**独立部署**。以及，它们应该可以在**共享组件**的同时进行并行开发——这些组件可以通过 NPM 或者 Git Tag、Git Submodule 来管理。

**注意**：这里的前端应用指的是前后端分离的单应用页面，在这基础才谈论微前端才有意义。

结合我最近半年在[微前端](https://github.com/phodal/microfrontends)方面的实践和研究来看，微前端架构一般可以由以下几种方式进行：

1. 使用 HTTP 服务器的路由来重定向多个应用
2. 在不同的框架之上设计通讯、加载机制，诸如 [Mooa](https://github.com/phodal/mooa) 和 [Single-SPA](https://github.com/CanopyTax/single-spa)
3. 通过组合多个独立应用、组件来构建一个单体应用
4. iFrame。使用 iFrame 及自定义消息传递机制
5. 使用纯 Web Components 构建应用
6. 结合 Web Components 构建


不同的方式适用于不同的使用场景，当然也可以组合一起使用。那么，就让我们来一一了解一下，为以后的架构演进做一些技术铺垫。

基础铺垫：应用分发路由 -> 路由分发应用
---

在一个单体前端、单体后端应用中，有一个典型的特征，即路由是由**框架**来分发的，框架将路由指定到对应的组件或者内部服务中。微服务在这个过程中做的事情是，将调用由**函数调用**变成了**远程调用**，诸如远程 HTTP 调用。而微前端呢，也是类似的，它是将**应用内的组件调用**变成了更细粒度的**应用间组件调用**，即原先我们只是将路由分发到应用的组件执行，现在则需要根据路由来找到对应的应用，再由应用分发到对应的组件上。

### 后端：函数调用 -> 远程调用

在大多数的 CRUD 类型的 Web 应用中，也都存在一些极为相似的模式，即：首页 -> 列表 -> 详情：

 - 首页，用于面向用户展示特定的数据或页面。这些数据通常是有限个数的，并且是多种模型的。
 - 列表，即数据模型的聚合，其典型特点是某一类数据的集合，可以看到尽可能多的**数据概要**（如 Google 只返回  100 页），典型见 Google、淘宝、京东的搜索结果页。
 - 详情，展示一个数据的尽可能多的内容。

如下是一个 Spring 框架，用于返回首页的示例：

```java
@RequestMapping(value="/")
public ModelAndView homePage(){
   return new ModelAndView("/WEB-INF/jsp/index.jsp");
}
```

对于某个详情页面来说，它可能是这样的：

```java
@RequestMapping(value="/detail/{detailId}")
public ModelAndView detail(HttpServletRequest request, ModelMap model){
   ....
   return new ModelAndView("/WEB-INF/jsp/detail.jsp", "detail", detail);
}
```

那么，在微服务的情况下，它则会变成这样子：

```
@RequestMapping("/name")
public String name(){
    String name = restTemplate.getForObject("http://account/name", String.class);
    return Name" + name;
}
```

而后端在这个过程中，多了一个服务发现的服务，来管理不同微服务的关系。

### 前端：组件调用 -> 应用调用

在形式上来说，单体前端框架的路由和单体后端应用，并没有太大的区别：**依据不同的路由，来返回不同页面的模板。**

```javascritp
const appRoutes: Routes = [
  { path: 'index', component: IndexComponent },
  { path: 'detail/:id', component: DetailComponent },
];
```

而当我们将之微服务化后，则可能变成应用 A 的路由：

```javascritp
const appRoutes: Routes = [
  { path: 'index', component: IndexComponent },
];
```

外加之应用 B 的路由：

```javascritp
const appRoutes: Routes = [
  { path: 'detail/:id', component: DetailComponent },
];
```

而问题的关键就在于：**怎么将路由分发到这些不同的应用中去**。与此同时，还要负责管理不同的前端应用。

路由分发式微前端
---

**路由分发式微前端**，即通过路由将不同的业务**分发到不同的、独立前端应用**上。其通常可以通过 HTTP 服务器的反向代理来实现，又或者是应用框架自带的路由来解决。

就当前而言，通过路由分发式的微前端架构应该是采用最多、最易采用的 “微前端” 方案。但是这种方式看上去更像是**多个前端应用的聚合**，即我们只是将这些不同的前端应用拼凑到一起，使他们看起来像是一个完整的整体。但是它们并不是，每次用户从 A 应用到 B 应用的时候，往往需要刷新一下页面。

在几年前的一个项目里，我们当时正在进行**遗留系统重写**。我们制定了一个迁移计划：

1. 首先，使用**静态网站生成**动态生成首页
2. 其次，使用 React 计划栈重构详情页
3. 最后，替换搜索结果页

整个系统并不是一次性迁移过去，而是一步步往下进行。因此在完成不同的步骤时，我们就需要上线这个功能，于是就需要使用 Nginx 来进行路由分发。

如下是一个基于路由分发的 Nginx 配置示例：

```
http {
  server {
    listen       80;
    server_name  www.phodal.com;
    location /api/ {
      proxy_pass http://http://172.31.25.15:8000/api;
    }
    location /web/admin {
      proxy_pass http://172.31.25.29/web/admin;
    }
    location /web/notifications {
      proxy_pass http://172.31.25.27/web/notifications;
    }
    location / {
      proxy_pass /;
    }
  }
}
```

在这个示例里，不同的页面的请求被分发到不同的服务器上。

随后，我们在别的项目上也使用了类似的方式，其主要原因是：**跨团队的协作**。当团队达到一定规模的时候，我们不得不面对这个问题。除此，还有 Angluar 跳崖式升级的问题。于是，在这种情况下，用户前台使用 Angular 重写，后台继续使用 Angular.js 等保持再有的技术栈。在不同的场景下，都有一些相似的技术决策。

因此在这种情况下，它适用于以下场景：

 - 不同技术栈之间差异比较大，难以兼容、迁移、改造
 - 项目不想花费大量的时间在这个系统的改造上
 - 现有的系统在未来将会被取代
 - 系统功能已经很完善，基本不会有新需求

而在满足上面场景的情况下，如果为了更好的用户体验，还可以采用 iframe 的方式来解决。
 
使用 iFrame 创建容器
---

iFrame 作为一个非常古老的，人人都觉得普通的技术，却一直很管用。

> **HTML 内联框架元素** ``<iframe>`` 表示嵌套的正在浏览的上下文，能有效地将另一个 HTML 页面嵌入到当前页面中。

iframe 可以创建一个全新的独立的宿主环境，这意味着我们的前端应用之间可以相互独立运行。采用 iframe 有几个重要的前提：

 - 网站不需要 SEO 支持
 - 拥有相应的**应用管理机制**。

如果我们做的是一个应用平台，会在我们的系统中集成第三方系统，或者多个不同部门团队下的系统，显然这是一个不错的方案。一些典型的场景，如传统的 Desktop 应用迁移到 Web 应用：

![Angular Tabs 示例](images/angular-tabs-example.png)

如果这一类应用过于复杂，那么它必然是要进行微服务化的拆分。因此，在采用 iframe 的时候，我们需要做这么两件事：

 - 设计**管理应用机制**
 - 设计**应用通讯机制**

**加载机制**。在什么情况下，我们会去加载、卸载这些应用；在这个过程中，采用怎样的动画过渡，让用户看起来更加自然。

**通讯机制**。直接在每个应用中创建 ``postMessage`` 事件并监听，并不是一个友好的事情。其本身对于应用的侵入性太强，因此通过 ``iframeEl.contentWindow`` 去获取 iFrame 元素的 Window 对象是一个更简化的做法。随后，就需要**定义一套通讯规范**：事件名采用什么格式、什么时候开始监听事件等等。

有兴趣的读者，可以看看笔者之前写的微前端框架：[Mooa](https://github.com/phodal/mooa)。

不管怎样，iframe 对于我们今年的 KPI 怕是带不来一丝的好处，那么我们就去造个轮子吧。

自制框架兼容应用
---

不论是基于 Web Components 的 Angular，或者是 VirtualDOM 的 React 等，现有的前端框架都离不开基本的 HTML 元素 DOM。

那么，我们只需要：

1. 在页面合适的地方引入或者创建 DOM
2. 用户操作时，加载对应的应用（触发应用的启动），并能卸载应用。

第一个问题，创建 DOM 是一个容易解决的问题。而第二个问题，则一点儿不容易，特别是移除 DOM 和相应应用的监听。当我们拥有一个不同的技术栈时，我们就需要有针对性设计出一套这样的逻辑。

尽管 [Single-SPA](https://github.com/CanopyTax/single-spa) 已经拥有了大部分框架（如 React、Angular、Vue 等框架）的启动和卸载处理，但是它仍然不是适合于生产用途。当我基于 Single-SPA 为 Angular 框架设计一个微前端架构的应用时，我最后选择重写一个自己的框架，即 [Mooa](https://github.com/phodal/mooa)。

虽然，这种方式的上手难度相对比较高，但是后期订制及可维护性比较方便。在不考虑每次加载应用带来的用户体验问题，其唯一存在的风险可能是：**第三方库不兼容**。

但是，不论怎样，与 iFrame 相比，其在技术上更具有**可吹牛逼性**，更有看点。同样的，与 iframe 类似，我们仍然面对着一系列的不大不小的问题：

 - 需要设计一套管理应用的机制。
 - 对于流量大的 toC 应用来说，会在首次加载的时候，会多出大量的请求

而我们即又要拆分应用，又想 blabla……，我们还能怎么做？

组合式集成：将应用微件化
---

**组合式集成**，即通过**软件工程**的方式在构建前、构建时、构建后等步骤中，对应用进行一步的拆分，并重新组合。

从这种定义上来看，它可能算不上并不是一种微前端——它可以满足了微前端的三个要素，即：**独立运行**、**独立开发**、**独立部署**。但是，配合上前端框架的组件 Lazyload 功能——即在需要的时候，才加载对应的业务组件或应用，它看上去就是一个微前端应用。

与此同时，由于所有的依赖、Pollyfill 已经尽可能地在首次加载了，CSS 样式也不需要重复加载。

常见的方式有：

 - 独立构建组件和应用，生成 chunk 文件，构建后再**归类**生成的 chunk 文件。（这种方式更类似于微服务，但是成本更高）
 - 开发时独立开发组件或应用，集成时合并组件和应用，最后生成单体的应用。
 - 在运行时，加载应用的 Runtime，随后加载对应的应用代码和模板。

应用间的关系如下图所示（其忽略图中的 “前端微服务化”）：

![组合式集成对比](images/angular-split-code-compare.jpg)

这种方式看上去相当的理想，即能满足多个团队并行开发，又能构建出适合的交付物。

但是，首先它有一个严重的限制：**必须使用同一个框架**。对于多数团队来说，这并不是问题。采用微服务的团队里，也不会因为微服务这一个前端，来使用不同的语言和技术来开发。当然了，如果要使用别的框架，也不是问题，我们只需要结合上一步中的**自制框架兼容应用**就可以满足我们的需求。

其次，采用这种方式还有一个限制，那就是：**规范！****规范！****规范！**。在采用这种方案时，我们需要：

 - 统一依赖。统一这些依赖的版本，引入新的依赖时都需要一一加入。
 - 规范应用的组件及路由。避免不同的应用之间，因为这些组件名称发生冲突。
 - 构建复杂。在有些方案里，我们需要修改构建系统，有些方案里则需要复杂的架构脚本。
 - 共享通用代码。这显然是一个要经常面对的问题。
 - 制定代码规范。

因此，这种方式看起来更像是一个软件工程问题。

现在，我们已经有了四种方案，每个方案都有自己的利弊。显然，结合起来会是一种更理想的做法。

考虑到现有及常用的技术的局限性问题，让我们再次将目光放得长远一些。

纯 Web Components 技术构建
---

在学习 Web Components 开发微前端架构的过程中，我尝试去写了我自己的 Web Components 框架：[oan](https://github.com/phodal/oan)。在添加了一些基本的 Web 前端框架的功能之后，我发现这项技术特别适合于**作为微前端的基石**。

> Web Components 是一套不同的技术，允许您创建可重用的定制元素（它们的功能封装在您的代码之外）并且在您的 Web 应用中使用它们。

它主要由四项技术组件：

 - Custom elements，允许开发者创建自定义的元素，诸如 <today-news></today-news>。
 - Shadow DOM，即影子 DOM，通常是将 Shadow DOM 附加到主文档 DOM 中，并可以控制其关联的功能。而这个 Shadow DOM 则是不能直接用其它主文档 DOM 来控制的。
 - HTML templates，即 ``<template>`` 和 ``<slot>`` 元素，用于编写不在页面中显示的标记模板。
 - HTML Imports，用于引入自定义组件。

每个组件由 ``link`` 标签引入：

```
<link rel="import" href="components/di-li.html">
<link rel="import" href="components/d-header.html">
```

随后，在各自的 HTML 文件里，创建相应的组件元素，编写相应的组件逻辑。一个典型的 Web Components 应用架构如下图所示：

![Web Components 架构](images/web-components-architecture.png)

可以看到这边方式与我们上面使用 iframe 的方式很相似，组件拥有自己独立的 ``Scripts`` 和 ``Styles``，以及对应的用于单独部署组件的域名。然而它并没有想象中的那么美好，要直接使用**纯** Web Components 来构建前端应用的难度有：

 - 重写现有的前端应用。是的，现在我们需要完成使用 Web Components 来完成整个系统的功能。
 - 上下游生态系统不完善。缺乏相应的一些第三方控件支持，这也是为什么 jQuery 相当流行的原因。
 - 系统架构复杂。当应用被拆分为一个又一个的组件时，组件间的通讯就成了一个特别大的麻烦。

Web Components 中的 ShadowDOM 更像是新一代的前端 DOM 容器。而遗憾的是并不是所有的浏览器，都可以完全支持 Web Components。

结合 Web Components 构建
---

Web Components 离现在的我们太远，可是结合 Web Components 来构建前端应用，则更是一种面向未来演进的架构。或者说在未来的时候，我们可以开始采用这种方式来构建我们的应用。好在，已经有框架在打造这种可能性。

就当前而言，有两种方式可以结合 Web Components 来构建微前端应用：

 - 使用 Web Components 构建独立于框架的组件，随后在对应的框架中引入这些组件
 - 在 Web Components 中引入现有的框架，类似于 iframe 的形式

前者是一种组件式的方式，或者则像是在迁移未来的 “遗留系统” 到未来的架构上。

### 在 Web Components 中集成现有框架

现有的 Web 框架已经有一些可以支持 Web Components 的形式，诸如 Angular 支持的 createCustomElement，就可以实现一个 Web Components 形式的组件：

```
platformBrowser()
	.bootstrapModuleFactory(MyPopupModuleNgFactory)
		.then(({injector}) => {
			const MyPopupElement = createCustomElement(MyPopup, {injector});
			customElements.define(‘my-popup’, MyPopupElement);
});
```

在未来，将有更多的框架可以使用类似这样的形式，集成到 Web Components 应用中。

### 集成在现有框架中的 Web Components

另外一种方式，则是类似于 [Stencil](https://github.com/ionic-team/stencil) 的形式，将组件直接构建成 Web Components 形式的组件，随后在对应的诸如，如 React 或者 Angular 中直接引用。

如下是一个在 React 中引用 Stencil 生成的 Web Components 的例子：

```javascriptA
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

import 'test-components/testcomponents';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```

在这种情况之下，我们就可以构建出独立于框架的组件。

同样的 Stencil 仍然也只是支持最近的一些浏览器，比如：Chrome、Safari、Firefox、Edge 和 IE11

复合型
---

**复合型**，对就是上面的几个类别中，随便挑几种组合到一起。

我就不废话了~~。

结论
---

那么，我们应该用哪种微前端方案呢？答案见下一篇《微前端快速选型指南》

相关资料：

 - [MDN 影子DOM（Shadow DOM）](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/%E5%BD%B1%E5%AD%90_DOM)
 - [Web Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)
 - [Shadow DOM v1：独立的网络组件](https://developers.google.com/web/fundamentals/web-components/shadowdom?hl=zh-cn)