此生无悔入 TypeScript
===

想来，我已经用 TypeScript 已经有一段时间了，它可以算得上是前端领域的一门 “平淡生活” 的语言。

平淡生活，我的意思是：生活可以从此多一点乐趣——毕竟 bug 少了一些。

最近的一年多里，我造轮子的时候，基本上选用的是 TypeScript：

 - [Mooa](https://github.com/phodal/mooa)，用于 Angular 的微前端框架
 - [ADR](https://github.com/phodal/adr)，轻量级架构决策记录工具
 - [mest](https://github.com/phodal/mest)，interface 契约测试工具
 - [Solla](https://github.com/phodal/solla)，封面矢量图片生成库
 - [Stepping](https://github.com/phodal/stepping)，EventStorming 记录工具

这一点也可以在我的 GitHub 首页上体现：

![Phodal GitHub](phodal_githun_typescript.png)


我是从 Ionic 2 beta （混合应用开发框架）开始使用 TypeScript，大概是在 16 年。Ionic 1 是基于 Angular.js，Ionic 2 也就基于 Angular 2，因此 Ionic 使用的便是 TypeScript。

协作性
---

起先，由于多数是用在个人的 side project 里，我便觉得 TypeScript 是一门繁琐的语言——每次创建一个函数的时候，我总得想一下这个对象的类型。

如果只是 ``const apps: any[] = []``，倒还算是不错的。可是，在多数时候，它是 ``any``，又或者是：

```javascript
registerApplication(appConfig?: any, customProps: object = {}) {

}
```

这样的代码，TSLint 或者 ``ng build --prod`` 的话，也是能通过的。可当 Angular 框架开始流行开来时，便发现事情不一样了。

当别人调用你的函数时，如果你返回的 type 是 ``any``，那么调用方可能需要这么去声明返回的变量：

```
const blabla: any = somethings()
```

又需要这么去写代码：

```
(something as any).callMethod()
```

而如果使用的语言不是 ``typescript`` 的话，那么可能是这样子的：

```
if (somethings && somethings. hasOwnProperty('callMethod') {}
```

这样一来，总算是体现出了 TypeScript 的优势。对，这个东西就是智能感知：

![](typescript-intellisense.png)

与 JavaScript 相比，TypeScript 的强类型，可以由 IDE/编辑器 赋予开发者更智能的开发提示，如某个对象拥有哪些可以调用的方法，获取的值。

类型
---

**Type**Script 顾名思义是一门以 Type 为核心的语言，它可以做很多类型相关的事情。

在 ES20xx 到来之前，它显然是最完善的 JavaScript 方言之一。尽管 ES6/ES7 不断的在尝试推出一些新的特性，但是对于旧的浏览器而言，显然是需要 Babel 之类的转义为 ES5。与此同时一些高级的语言特性，ES6/ES7 还只在草案中，等到被 Node.js、各有大浏览器支持，不知道要到何年何月。

因而在这个时候，与其一点点跟进语言特性，还不如拥抱 TypeScript。

### 类型检查

类型检，测除了用于正常的变量声名：

```
private authService: AuthService;
private role: any;
public githubUser: UserResponse;
private http: HttpClient;
```

在 Angular 的 HTTPClient 中还可以对 HTTP 返回结果进行类型检查：

```
this.http.get<UserResponse>('https://api.github.com/users/phodal')
  .subscribe(
    data => this.githubUser = data,
    err => console.log(err)
  );
```

对，这样就可以 GET 到你想要的**对象**了。

### OOP

TypeScript 是一门对 OOP（object-oriented programming，面向对象编程） 特别友好的语言——实现接口，可以强制一个类去符合 interface：

```
export interface AbstractBuilder {
  setStart (startSting: any)
  setBody (handleBody: any)
  setEnd (endString: any)
  build ()
}
```

对应的实现类：

```
export class GenerateBuilder implements AbstractBuilder {
 ...
}
```

以及常见的继承：

```
export default class SearchListGenerateBuilder extends GenerateBuilder {
 ...
}
```

除此还有各种组合式用法，如：接口继承类。

当然了，FRP（functional reactive programming，函数式响应式编程） 与 OOP（object-oriented programming，面向对象编程）并不是完全对立的，而是互相结合的。这一点在 JavaScript（ES6+) 及 TypeScript 上的体现更加明显。

未来
---

我不是说 TypeScript 有多好，而是在说 JavaScript （ES5）真的不行。诸如 CoffeeScript 是之前最受欢迎的 JavaScript 方言，Angular 默认使用的是 TypeScript，并且 Vue 和 React 也提供了 TypeScript 支持。

未来，ES20xx 足够完善的时候，我们就可以继续用 EcmaScript。在那之前，不妨试试 TypeScript。



