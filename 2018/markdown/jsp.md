# 前后端架构趋同：你写的前端代码，类似于我的 JSP

写完《整洁架构》之后，经历一次简单的讨论，我从同事那里得到一个有意思的结论：『你写的前端代码，类似于我的 JSP』。

开始之前，让我们先强调一遍：

 - ** 本文的观点，仅适用于复杂前端应用。**
 - ** 本文的观点，仅适用于复杂前端应用。**
 - ** 本文的观点，仅适用于复杂前端应用。**

如果你的应用相当的简单，就没有理由采用复杂的架构模式。又或者是你们拥有一个强大的 BFF，或者业务交互不复杂，或以致于前端只展示，那么这篇文章的结论对于你们来说，也是不生效。

所以……，你懂我的意思，这个场景多少是不适合你的。

## 缘由

最开始的时候，我觉得有像是对的，又好像有哪里不对。直至，我打开了一个公司的 DDD 项目，看了看相应的代码。再将它与我在 GitHub 上编写的 Clean Architecture 目录结构和思维方式，我便陷入深深的思考：我究竟是在写前端代码，还是在写后端代码——Interface、Model、Entity、Services、Repository……。

我仿佛回到了 5 年前，我来到 ThoughtWorks 实习，在写 Spring MVC 的 ModelAndView 和 JSP 代码。作为一个菜鸟，我听到最多的一句话是：**不要把业务逻辑这到 JSP 里**。而在今天，我们对于前端新人的话是：『**不要把业务逻辑放到模板里**』。一来，它不可测试；二来，它容易出现 Bug——哪怕是今天的 IDE，对于模板的计算也没有那么美好。

于是，就有了这篇文章了——hiahia。

## 示例 1：MVC/MVP

无论你是写前端代码，还是写后端代码，对于 MV* 都有基本的认识，我就不强调了。

最近，手疼——职业病。

后端：

 - model
 - services.
 - repository - 连接数据库
 - controller.

前端：

 - model。
 - presenter。controller，Angular 里的组件
 - view。双向绑定，模板
 - repository/services - 连接后台服务

## 示例 2： Clean Architecture

最近，手疼——职业病。

后端示例：

因为前后端分离的缘故，后端的 Controller 的逻辑基本上已经很干净了—— 当然了，你可以把 JSON View 视为 Presenter 的逻辑，只是它已经被框架取代。所以，最好的例子就是参考 Android Clean Architecture 的示例：

 - model 层。
 - view 层。即 activity/fragment，对于 view 的直接处理
 - presenter 层。处理真正的业务显示逻辑（间接处理），如是否显示
 - domain 层。业务层
     - usecase/interfactor。业务逻辑实现层
     - repository/dao。数据层
 - 其它无关本话题的部分

代码示例：[Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)

前端相关的部分，详细见：《整洁架构》。相应的例子有一个 Angular 的，哈哈：

 - mode 层。
 - presenter 层。即页面级组件，逻辑由 Component + Template 共同完成
 - components 层。通用组件
 - domain 层。
     - 某一具体业务
       - usecase/interfactor。业务逻辑实现层
       - repository/dao。数据层

## 示例 3： DDD



## 组件化 + DDD + MVP + Clean Architecture



