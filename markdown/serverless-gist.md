 某天，我在微信群里发了一段代码，突然就有了一个想法——我应该做一个这样的小程序：它可以很方便的在微信群里分享代码。于是，就有了“今日代码”。

对于我来说，Serverless 已经相当的顺手，差不多花了一个星期天 + 一个晚上，就完成了小程序 + web + 服务端的功能。设计过程如下：

1. 使用前端页面做一个简单的页面原型（理论上应该使用 Sketch  之类的工具，但是最初我想也做一个网页版）

## 用户旅程

## Serverless API 列表

一般简单的 Serverless 服务都是以 CRUD 为主，在这里也是如此。

| URL             | HTTP Method  |   用途          |
|-----------------|--------------|----------------|
| /               | GET          | 返回首页        | 
| /	               | POST         | 创建代码        |
| /features       | GET			   | 返回首页推荐内容  |
| /login          | GET          | 获取 OpenID     |
| /user/{userId}  | GET		      | 获取对应用户的代码 |
| /code/{codeId}  | GET          | 获取特定代码      |
| /code/{codeId}  | POST         | 更新特定代码      |
| /code/{codeId}  | DELETE       | 删除特定代码      |

由于最初设计的时候，并没有预期到这么复杂的功能，所以创建代码的 API 的 URI 是 ``/``，而不是 ``/code/``，现在懒得改了。

### Login API

```javascript
request(`https://api.weixin.qq.com/sns/jscode2session?appid=${APPID}&secret=${SECRET}&js_code=${JSCODE}&grant_type=authorization_code`, {json: true}, (err, res, body) => {
  if (err) {
    console.log(err);
    ...
  }
  
  ...

  const response = {
    statusCode: 200,
    body: res,
  };
  callback(null, response);
});
```

## 代码

由于在上架过程中遇到 UGC 的问题，即只有企业认证过才能让用户自己创建内容。为了避免中途应用**被其它公司上架**，目前代码封存中，将在上架后开放源码。