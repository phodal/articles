#  混合应用：从 file 协议到本地 HTTP 服务器

过去的几个月里，我们一直在开发一个混合应用。前端框架使用的是 Angular，但是在某些 Android 机型上运行的时候，报了以下的错误：

```
main.aca1d67….bundle.js:1235 EXCEPTION: Uncaught (in promise): SecurityError: Failed to execute 
'replaceState' on 'History': A history state object with URL 'file:///#/' cannot be created in a document with 
origin 'file://' and URL 'file:///android_asset/www/index.html'.t.handleError @
```

错误的主要原因是：**History API 在某些 Chrome 内核的 WebView 上不支持 file:// 协议**。据不完成测试统计，版本在 45~49 之间，都有这个问题。

随后找到了官方文档中的 issue：[Cannot run angular 2+ from file:/// - looks like 'base href="/"' is the issue](https://github.com/angular/angular/issues/13948)

官方 issue 中的一个高赞答案，一共两步，并是不 work。

1.将 Router 配置为 Hash

```javascript
CommonModule,RouterModule.forRoot(routes,{useHash:true})
```

2.修改 base href:

```javascript
<script>document.write('<base href="' + document.location + '" />');</script>
```

于是乎，这时我们有两个选择：

1. 重写 HashLocationStrategy，如下 issue 12341 所说：[The router HashLocationStrategy should not use the History API](https://github.com/angular/angular/issues/12341) 
2. 使用 HTTP 服务器来运行本地的 WebView 资源文件。

方式一，面对的主要挑战是，不只 Angular 使用 History API 来处理 hash 路由，其它框架也使用了相似的东西，如 React Router、Vue Router。

方式二，面对的主要问题是，我们需要在 Android 设备上启动一个 HTTP 服务器，并能让它支持跨域请求和访问本地文件。

在这个过程中，想到了 Ionic 框架也是使用 Angular 来编写的。于是，先运行了个官方的 Hello, world，发现它是运行在 ``http://localhost:8100`` 上的。

紧接着，复制了官方的 WebView 插件代码：[Ionic Web View for Cordova](https://github.com/ionic-team/cordova-plugin-ionic-webview)。随之，我们就遇到了跨域的问题，原因是这些请求都是通过 WebView 发出去。

而这个问题，又近乎无解，我们无法获取 WebView post 请求中的参数。于是乎，我们只能通过插件来向 WebView 提供一个跨域请求的能力：[Cordova Advanced HTTP](https://github.com/silkimen/cordova-plugin-advanced-http)。
