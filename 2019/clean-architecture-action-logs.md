# Clean Architecture 实施指南

在之前的那篇《[整洁前端架构](https://phodal.github.io/clean-frontend/)》的文章里， 我们介绍了如何在前端领域里使用 Clean Architecture。在过去的几个月里，我们实践了 Clean Architecture 架构，并且实践证明了 Clean Architecture 也可以在前端工作得非常好。

## Clean Architecture + MVP + 组件化架构

开始之前，让我们先看一下使用 Clean Architecture 的 Angular 应用的最终架构：

![Clean MVP + Componet-based + MVP](clean-mvp-component-based.jpg)

图中，我们将架构拆为了这部分来考虑：

 - 数据层。即前端与后端所有交互的处理层，从请求到返回结果，只返回前端**需要的值与字段**。
 - MVP 层。MVP 不是本文的重点，不过有意思的是，在 Angular 应用中，module 层可以与后端 service 对应。而 module 层下的 page，也可以按此来拆分。
 - MVP 中的组件层。对于组件层的合理规划，会使我们的 componet 层也变得 clean，而不仅仅是 domain 层整洁。组件，难的地方是满足场景，而不是拆分、组合与封装。

在这里，我们少了样式层的部分。一来，使用各种 CSS 预处理器来组织代码已经很成熟了；二来，在基础设施完善的今天，CSS 已经没有那么痛了。

与我们旧的架构图相比，我们加入了更多的实施细节：

![Clean Architecture](old-clean-frontend-components.jpg)

不过，前者是限定条件的，而后者是通用条件下的架构。

## 有利于实施的上下文

Clean Architecture 并不是银弹，它适合于我们，并不代表它就适合于你们。尤其是如果你习惯自由、自主地项目开发，那么强规范化的 Clean Architecture + Angular 并不一定适合你们。不过，与此同时，如果你的团队规模比较大，并且初级开发者比较多，那么我想规范化能帮助你们减少问题——易于维护。

所以，我先大介绍一些有利于我们的上下文环境：

 - 实施 DDD 的微服务后台架构。
 - 有志于实现全功能团队的成员。

还有其它诸如于初级开发者较多，采用适合于企业（追求规范化）的 Angular 框架——所以规范更多一点，反而更好维护。不过，剩下的这些因素，以于我们的架构来说：有帮助，但并不会太大。

### 实施 DDD 的微服务后台架构

DDD 只是一套软件开发方法，不同的人理解下的 DDD 自有差异。在特定的情形下，使用 DDD 进行微服务拆分时，每个子域都是一个特定的微服务。在这种模式之下，我们需要有一种命名模式来区分每一种服务，而其中的一种体现方式则是 URL。每个服务有自己特有的 URL 前缀/路径，对应的当这些服务暴露出来时，便可以产出对应的前端领域层——哪怕是没有参加过事件风暴，又或者是领域划分。

诸如于：``/api/payment/``，``/api/manage/`` 都可以清晰地拆分出前端的 ``domain data`` 层。

与此同时，若是后端再根据资源路径命名 Controller，诸如于 ``/api/blog/blog/:id``，``/api/blog/blog/category/:id``，那么前端便可以清晰地把它们划分到同一个 ``repository`` 之中。当然了，一旦后端设计有问题，前端也能明显地察觉出来。

### 全功能团队

过去，我在 ThoughtWorks 的某个团队里，采用的是全功能团队的模式——团队成员擅长某一领域，但是会其它领域，比如擅长前端，后点后端。它可以在某种程度上，降低沟通成本。而这意味着，我们有大量的 knowledge transfer 的成本。因此，我们采用了结对编程、让新人讲项目相关的 session。

所以，当你们决定成为一个全功能团队（前后端 + 业务分析都做），那么你们就会遇到这样的问题：

 - 找到应用的前端代码，怎么快速找？
 - 找到对应的后端代码，
 - 前后端模型对应
 - ……

让前后端尽量保持一致，便成为了一种新的挑战。

## 目录即分层

从某种意义上来说，Clean Architectute 是一种**规范化**和**模板化**的实施方案。与此同时，它在数据层的三层机制，使得它存在两层**防腐层**，usecase 可以作为业务的缓冲层，repository 层可以隔离后端服务与模型。

在众多的架构设计之种，分层架构是最容易实施的，因为目录即分层。目录便是一种规范，一看就能看出哪放什么。一旦放错了，也能一眼看出来。

### MVP 分层（目录划分）

从目录结果上来说，我们的划分方式和一般的 Angular 应用，并不会有太大的区别。

```javascript
├── core			    // 核心代码，包含基本服务和基础代码
├── domain				// 业务层代码，包含每个业务的单独 Clean 架构内容
│  └── elephant		// 某一具体的业务
├── features		   	 // 公共的业务组件
├── presentation  // 业务逻辑页面
├── pages 				   // 公共页面
└── shared				  // 共享目录
```

我们将：

1. 业务页面都放到了 ``presentation`` 目录
2. 公共的页面（如 404）都放到了 ``pages`` 目录
3. 业务页面共用的业务组件放到了 ``features`` 目录
4. 剩余的通用部分都放到了 ``shared`` 目录，诸如于 ``pipes``、``utils``、``services``、``components``、``modules``

示例代码见：https://github.com/phodal/clean-frontend

### domain  + data 层：垂直  + 水平 分层

上述的目录中的 domain 层，示例结构如下所示：

```javascript
├── model
│   ├── elephant.entity.ts                         // 数据实体，简单的数据模型，用来表示核心的业务逻辑
│   └── elephant.model.ts                         // 核心业务模型
├── repository
│   ├── elephant.mapper.ts                      // 映射层，用于核心实体层映射，或映射到核心实体层。即进行模型转换
│   └── elephant.repository.ts                  // Repository，用于读取和存储数据。
└── usecases
    └── get-elephant-by-id-usecase.usecase.ts     // 用例，构建在核心实体之上，并实现应用程序的整个业务逻辑。
```

相关的解释如上，这里就不 Ctrl + V / Ctrl + C 了。

值得注意的是，我们采用的是垂直  + 水平双分层的模式，垂直应对的是领域服务。它适用于没有 BFF 的微服务架构，尤其是采用 DDD 的微服务后端应用。

## 映射领域服务

在上一部分中，前端的 domain + layer 层，实际上已经映射了后端的服务。当前端发起一个请求时，它的流程一般是这样的：Component / Controller（前端） -> Usecase -> Repository -> Controller（后端）

对应的返回顺序便是：Controller（后端） -> Repository -> Usecase -> Component / Controller（前端）

为此，我们将 Repository 与后端的 Controller 对应。并且由于服务的简单化，我们的大部分 usecase 也与 repository 中的命名对应。

### repository 命名：URL 命名

为了不看后端的代码就可以命名，我们使用 URL 来命名 repository 和 repository 中的方法。如有一个 

| URL  | 解释 | 抽象  |
|------|-----|----------|
| ``/api/blog/blog/:id`` | /API/微服务名/资源名称/资源 ID | HTTP 动词 + 资源 + 行为 |

于是乎，对应的 ``repository`` 的名字应该是 ``blog.repository.ts``。对应的 repository 的名字也是 ``get-blog-by-id``。相似的，还有 URL ``/api/blog/blog/:id/comment`` 对应的 repository 便是 ``get-comment-by-blog-id``。

嗯，是的，和数据库的存取保持一致。

### usecase 命名

由于，我们还不涉及复杂的 API，所以常见的行为如下：

 - 常规动词：get / create / update / delete / patch
 - 非常规：search, submit

哈哈，是不是和 repository 相似了。

## Clean Architecture 的 MVP 层实践

实际上这里的 MVP 层， 主要内容便是组件化架构。这部分的内容已经在之前的文章（《[【架构拾集】组件设计原则](https://www.phodal.com/blog/architecture-in-realworld-design-component-based-architecture/)》）介绍过了，这里就不详细介绍了。简单的介绍一下就是：

 - 基础组件库，如 Material Design
 - 二次封装组件。额外的三方组件库，如 Infinite Scroller，务必在封装后使用。
 - 自制基础组件。
 - 领域特定组件。

上述的四部分，构建了整个系统的通用组件库模块。随后，便是业务相关组件和页面级组件。

 - 业务相关组件。在不同的模块、页面之间，共享逻辑的组件。
 - 页面级组件。在不同的模块、路由之间，共享页面。

嗯，这部分的东西就这么多了。

## Clean Architecture 的 Domain + Data 层实践

嗯，其它部分和正常的项目开发，并没有太大的区别。于是，我们便可以把注意力集中在 Domain + Data 层上。

### DDD ApplicationService vs 多个 Usecases

> 在 DDD 实践中，自然应该采用自顶向下的实现方式。ApplicationService 的实现遵循一个很简单的原则，即一个业务用例对应ApplicationService上的一个业务方法。[^ddd_backend]

[^ddd_backend]: https://insights.thoughtworks.cn/backend-development-ddd/

稍微有点不同的是，我们采用的 Clean Architecture 推荐的方式是：**一个业务用例（usecase）对应于一个业务类**。即，同样的业务场景下，前端是一堆 usecase 文件，而后端是一个applicationService。所以，在这种场景之下，前端有：

 - change-production-count.usecase.ts
 - delete-product.usecase.ts

后端便是 ``OrderApplicationService.java``，其中有多个方法。

### usecases + repository vs services

如果我们的 usecases  + repository 做的功能，和一个 services 是一样的，那么只使用 serivce 不好吗？只使用 service 会有这样的问题：

 - 存在 API 重复调用的问题
 - 调用划分不清晰

那么，使用 usecase 呢：

 - 更多的模板化代码
 - 更多的分层

Usecases 的复用率极低，项目会因而急剧的增加类和重复代码。因此，我们试图以更多的代码量，来提升架构的可维护性。PS：更多的代码，还可能降低代码的可维护性，不过在 IDE 智能化的今天，这应该不是问题。

### Usecases 作为逻辑层/防腐层

不过，Usecases 在带来更多代码的同时，也带来了防腐层。它负责了以下的职责：

 - 业务逻辑处理。在数据传给后端之前，对一些必要的内容进行处理。
 - 返回数据管理。从后端返回的数据里，构建出前端所需要的结果。当需要调用多个 API 时，可以在 usecase 里做这样的工作。
 - 输入参数管理。

也因此，当前端被赋予过重的业务逻辑时，Usecases 层就非常有用。反之，如果逻辑被放置在 BFF 层时，那么 Usecases 层就变得有些鸡肋。但是它仍然是一个非常不错的防腐层。

## 模型管理

在我们处理 Usecase 的同时，我们就需要解决前端的模型问题。后端，有多个微服务，有多个工程，每个工程有自己的模型。而如果前端只有一个工程，那么前端的模型管理就变成一个痛点。因为在不同的限界上下文里，后端的模型是不一样的。即在不同的 API 里，其模型是不一样的，而这些根据业务定制的模型，最后在前端都聚合到一起。

在这个时候，同个资源有可能有多种不同的模型。因此，要么：

 - 前端拥有一一对应的模型。管理起来比较麻烦。
 - 使用同一个模型。不能使用类型检查来减少 bug。

当前，我也想不到一个更好的解决方法。我们采用的主要是第二种方式，毕竟管理起来方便一些。

以下是我们的几种类型的分类和管理方式：

 - **Request Model / Response Model**。即请求参数和返回模型（修改过，适用于前端展示），都放在服务的对应的 .model.ts 目录下。
 - **Response Entity**。直接使用后端返回结果时，名字上带 ``entity``，否则就使用 ``model``。
 - **View Model / Component Model**。适用于业务组件的封装时，传入参数用 model 传入。

你们呢，是否有更好的实践？

## 相关问题

真的 Clean 吗？还没有

### 框架依赖的表单验证

由于 Angular 框架本身提供了强大的 Reactive Form 功能，我们在大部分的表单设计时，采用了  Reactive Form，而不是通过 Entity 来验证的方式。这使得我们在这部分的 UI 交互，依赖于 Angular 框架，而非自己实现。

如果采用了诸如 DDD 的 Entity 模式，又或者是采用 validator 的方式。随后，我们还需要开发自己的表单验证模式，类似于此：

```javascript
{
    validator: RegExp,
		errorMessage: string
}
```


而它意味着大量的开发成本。不过，好在我们可以尽量将它通用化。

## 下一步

 - **Clean 表单**。如上所述。
 - **代码生成**。尽管，我们已经在项目中，采用了 Angular Schematics 来生成模板代码。但是相信，下一步我们可以使用工具来生成页面。
 - **架构守护**。有了分层结构之后， 要判定层级关系变得更加简单。
 - **其它框架尝试**。



