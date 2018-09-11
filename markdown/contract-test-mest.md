前后端分离：使用 TypeScript 的 interface 跟踪 API 接口变更
===

> 还在担心后端 API 变更对前端的影响？快来使用 mest

在实施前后端分离架构的过程中，最让人苦恼的莫过于：**API 发生了变化**。API 发生变化的原因那可是相当的丰富：业务变化、字段名出错、第三方接口不匹配等等，但是这些都不重要——一个字段从 string 变成了 number，对于前端的影响并不大。重要的是，后台 API 发生了一些变化，而前端却不知道或者没有人对此作出相应——比如说，某一天你下班了，后台的同事告诉了你，这个 API 变了。结果，第二天这事被忘记了，因此就轻而易举地就创造了一堆 bug。

契约与契约测试
---

过去，前后端约定好的 API 可能以文档的形式定义，当发生 API 变更的时候，可能以邮件、聊天工具等等不同的形式来通知 API 的消费方。但是，这种方式容易出现问题，比如说：不能通知到团队的所有人、或者相关的消息埋没在消息流中。为此，人们发明了一种以 JSON 作为契约的方式来作为契约好的 API 接口。

> 契约，是以双方当事人互相对立合致的意思表示所构成的，其中包括要约及承诺两个基本的意思表示。[^contract]

[^contract]: https://zh.wikipedia.org/zh/%E5%A5%91%E7%BA%A6

即，对于前端和后台来说，这个契约就是最后以 JSON 形式展现的 API。它可以用于前端开发时使用的接口，也是后端最后要提供的线上 API 格式：

```javascript
{
    "id": 9,
    "title": "搭建指南",
    "make": {
        "id": 8,
        "category": 9,
        "featured_image": "uploads/make/.thumbnails/guide-map.jpg/guide-map-600x360.jpg"
    },
    "slug": "搭建指南"
}
```

有了契约，前端不会被前端 Block，可以在后台接口还没开发好的情况下，继续开发工作——只需要在后期做一些集成工作即可。一般来说，对于这样一个契约（接口），它会包含大部分的 API 可能响应的情况，如 200 返回的是正常情况，404 返回 Not Found，401 返回权限异常。这个时候，我们只需要有一个 Mock Server 就可以了，在这个 Mock Server 里，只要能处理 参数、授权等等的内容即可。

然而，契约只是一种承诺，它并**不能保证**接口提供方能完成遵循这个契约开发。对于接口的提供来说，需要一个有效的机制来验证后台的接口是与契约对齐、一致的。

于是，就有了**契约测试**的概念。

### 契约测试

契约测试（Contract Test），又称为消费者驱动的契约测试（Consumer-Driven Contracts，简称CDC），顾名思义就是对契约进行测试。

由于后端是 API 的提供者，因而在维护契约上，后端也拥有了更多的责任——主要用于验证接口是否稳定。因而在选型的时候，多数以后端的技术栈为主，如 Java 系列，如 Spring Cloud Contract 就是一个不错的方式。

无论采用怎样的方式，它都在一定的程度上保证后端 API 与契约的一致性。

那么，对于前端呢？

前端契约测试：Mest 方式
---

对于前端来说，要追踪这样的一个契约并不是一件容易的事。过去，要对契约对待测试，无非就是将其写到正常的测试中，验证字段是否一致。直到，我看到了 interface 的时候有了一些想法。

### 引子：使用 interface 进行 API 类型检查

TypeScript 提供了一个 interface 这样的类型。在写 Angular 代码的时候，为了进行类型检测，我们会写一个 interface 来获取相应的值：

```
this.http.get<IUserResponse>('https://api.github.com/users/phodal')
  .subscribe(
    data => this.githubUser = data,
    err => console.log(err)
  );
```

使用的时候，只需要：

```
<div *ngIf="this.githubUser">
  <p><img [src]="this.githubUser.avatar_url" alt="" width="80" height="80"></p>
  <p>{{this.githubUser.company}}</p>
  <p>{{this.githubUser.bio}}</p>
</div>
```

即这里的 IUserResponse 是一个 interface，然而这个类型检测在构建完后，并不能自动展示出来。

可能你也想到了，这里的 interface 可以直接用来校验后台 API 是否与 interface 一致。

于是，我们需要一个更高级的手段来做这样的事，这就是 mest 框架要做的，哈~。

对，我撸了一个新的框架，[mest](https://github.com/phodal/mest)。先来看看用法：

### Mest 前端测试契约： CLI 模式

CLI 模式，其实是我早期验证是否可以工作的时候，留下来的接口。它可以验证一些公开的 API 是否是可以访问的，不需要编写任何代码。

1. 首先只需要安装 mest

```
npm install -g mest
```

2. 然后编写一个简单的 csv 文件，里面映射有 URL 和接口路径：

```
url,interface
https://phodal.github.io/mest-test/error.json,mock/IError.ts
https://phodal.github.io/mest-test/moreerror.json,mock/IMoreIError.ts
https://phodal.github.io/mest-test/user.json,mock/IUser.ts
```

然后，就可以运行命令进行测试了：

```
mest -i data/url.csv
```

完了，将会显示对应的结果：

```
-> API https://phodal.github.io/mest-test/error.json .
-> API https://phodal.github.io/mest-test/moreerror.json .
-> API https://phodal.github.io/mest-test/user.json .
same key: login,id,avatar_url,url,html_url,followers_url,following_url,gists_url,starred_url,subscriptions_url,organizations_url,repos_url,events_url,received_events_url,type,site_admin,name,company,blog,location,email,hireable,bio,public_repos,public_gists,followers,following,created_at,updated_at
local diff key: avatar_id, remote diff: gravatar_id
```

###  Mest 前端测试契约：API 调用

当然，除了对于大部分的契约来说，它们都是需要一系列复杂的交互，才能进行下一步的对比。这个时候，一个比较简单的方式，就是直接调用这些 API：

```
yarn add mest
```

然后直接对响应的结果进行处理即可，就可以直接进行断言。

```typescript
let mest = new Mest()

let results: IDiff = mest.localCompareInterface('mock/IError.ts', {
  id: 233333,
  key: 'ssfd',
  messages: '2010-11-08T11:46:51Z',
  documentation_url: 2
})

expect(results).toEqual({
  diff: {
    local: ['message'],
    remote: ['key', 'messages']
  },
  diffTypes: [],
  same: ['id', 'documentation_url']
})
```

对，就是这么简单，只需要一个 Interface 就可以验证 API 的 **key 是否一致，以及 value 的类型是否一致**。

结论
---

原理其实很简单，就是获取 interface 的 key-value，以及 API 的 key-value 进行对比。

TypeScript 大法好，Mest 大法好

