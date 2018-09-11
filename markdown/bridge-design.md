# 【架构拾集】WebView 的 JavaScript Bridge 设计
最近的一个项目，是一个关于在移动应用中嵌入 WebView，并在其中实现 JavaScript 能调用原生代码。虽然已经有 Cordova 这样的成熟方案，但是它太 “重” 了——直接在一个拥有大量代码的项目上，修改起来并不容易。

原先，我并不打算花费笔墨来记录相关的实现。毕竟，JavaScript Bridge 相关的方案相当的成熟——到处可风，但是考虑到其中的一些设计思想相当不错。我还是想存个档下来，方便自己以后查阅。它之所以成为了我的第 701 篇博客，也不是没有理由的，毕竟年纪一大记忆力就不好了。

广告时间：作为一个前端开发人员，我更推荐在底层可以使用 React Native，可以节省开发成本。也可以使用我之前写的基于 React Native 实现的 WebView 容器 [Dore](https://github.com/phodal/dore)，这样能节省大量的开发成本。

## 技术远景

作为项目的开发人员，我们希望能提供一个 WebView 的 APP 壳，它可以提供 WebView 访问原生组件的能力。

## 适用场景

| 关键因素 | 描述 |
| --- | --- |
| 对于 | 想通过 JavaScript 调用原生代码的开发人员|
| 我们的方案 | JS Bridge 接口 |
| 是一个 | 轻量级 JavaScript Bridge |
| 它可以 | 在 APP 内提供原生代码调用  JavaScript、JavaScript 调用原生代码的接口 |
| 但他不同于 | Cordova |
| 它的优势是 | 代码量低、维护成本低 |

## 方案对比

对于这个项目来说，在最开始的时候，我们设计了两个方案：

 - 同步数据流，使用常规的 HTTP 来请求数据，在请求的 promise 里返回相应的结果。
 - 异步回调，通常 iframe 的方式创建 HTTP 请求，在原生代码执行成功后，调用相应的 JavaScript 来返回对应的结果。

之所以有两种方案的原因，也是因为两种方案各有优缺点

### 同步数据流

第一版里，我们截获 HTTP 数据流，而后调用相应的响应结果：

```javascript
this.http.get("bridge://picture").then((res: any) => {
  console.log(res.json());
});
```

这样做的过程中，遇到了一个问题，即 HTTP 请求的 Timeout。如果我们的请求超时了，那么我们的方法调用就失败了。在 APP 中有诸多这样的场景，如用户选择相册的时候，或者用户拍照的时候。

### 异步调用

第二个版本里，我们则使用在原生代码里直接调用  JavaScript 。这是最常见的一种方式，知名的混合应用框架 Cordova 就是使用这种方式来实现的。

其对应的逻辑便是：

1. 从 JavaScript 代码中发起对原生代码的调用
2. 生成对应的 callbackId 和函数的映射，并绑定到全局的 window 变量中
3. 原生代码执行完后，调用 JavaScript 代码
4. JavaScript 代码通过传回的 Callback ID  找到、调用对应的函数
5. 完成后，按需要删除相应的 Callback ID

（PS：Callback ID 用于识别不同的函数）

这个时候，我们就需要手动提供 JavaScript 的接口，相应的调用逻辑也就变成了：

```javascript
jsbridge.get("bridge://picture").then((res: any) => {
  console.log(res.json());
});
```

两种方式的区别并不是太大。不过这样做有一个好处是，避免与 HTTP 请求之间的混淆。

## 方案调研

由于 WebView 的 JavaScript ，因此我们不妨先大致了解一下现有的几种设计。

 - 覆写原生 WebView 的方法来捕获请求，如 WebView 中的 prompt 弹出对话框方法
 - 在 WebView 中创建相应调用的数组，而后通过轮询数组来获取方法
 - 通过 iframe 来创建 HTTP 请求，在 APP 壳对对应的请求进行处理

下面则是关于 Cordova 一些方案的设计。

### JavaScript 调用原生

对于  JavaScript 调用原生方面的实现，做得比较成功的莫过于 Cordova 框架了，于是我们可以先简单地了解一下其中的实现。

#### Cordova 版本： Android 实现

由于早期 Android 版本（即 JellyBean，4.1 之前）不支持使用 addJavascriptInterface，所以提供了两种方式实现：

```javascript
jsToNativeModes = {
  PROMPT: 0,
  JS_OBJECT: 1
}
```

``PROMPT`` 即通过覆写 DOM 中的对话框方法来接收参数，``JS_OBJECT`` 的方式是通过 ``addJavascriptInterface``  来实现。在 addJavascriptInterface 不支持的情况下，自动转换为使用 prompt 方式实现。

对应的调用原生代码逻辑，如下所示：

```javascript
function androidExec(success, fail, service, action, args) {
	  ...
		var msgs = nativeApiProvider.get().exec(bridgeSecret, service, action, callbackId, argsJson);
   	... 
}
```

如果其通过 ``JS_OBJECT`` 调用失败，则会使用 ``PROMPT`` 方法实现，其调用  prompt 的实现如下所示：

```javascript
module.exports = {
    exec: function(bridgeSecret, service, action, callbackId, argsJson) {
        return prompt(argsJson, 'gap:'+JSON.stringify([bridgeSecret, service, action, callbackId]));
    },
    setNativeToJsBridgeMode: function(bridgeSecret, value) {
        prompt(value, 'gap_bridge_mode:' + bridgeSecret);
    },
    retrieveJsMessages: function(bridgeSecret, fromOnlineEvent) {
        return prompt(+fromOnlineEvent, 'gap_poll:' + bridgeSecret);
    }
};
```

而后在 Android 的 Java 代码中覆写 ``onJsPrompt`` 方法，

```java
@Override
public boolean onJsPrompt(WebView view, String origin, String message, String defaultValue, final JsPromptResult result) {
    // Unlike the @JavascriptInterface bridge, this method is always called on the UI thread.
    String handledRet = parentEngine.bridge.promptOnJsPrompt(origin, message, defaultValue);
    ...
    return true;
}
```

逻辑大体上就是这样。

#### Cordova 版本：iOS 实现

iOS 端在实现上，就没有 Android 这么复杂，则是先往 commandQueue 中 push 数据：

```
var command = [callbackId, service, action, actionArgs];
commandQueue.push(JSON.stringify(command));
```

然后通过创建 ``iframe``，告知 iOS 原生代码可以处理数据了：

```
if (execIframe && execIframe.contentWindow) {
    execIframe.contentWindow.location = 'gap://ready';
} else {
    execIframe = document.createElement('iframe');
    execIframe.style.display = 'none';
    execIframe.src = 'gap://ready';
    document.body.appendChild(execIframe);
}
```

原理大致是这样的：

###  Callback ID 生成与绑定

对应的生成 ID 的逻辑比较简单：

```javascript
var cordova = {
	...
  callbackId: Math.floor(Math.random() * 2000000000),
  callbacks: {}
}
```

代码绑定相应的逻辑便是：

```javascript
function androidExec(success, fail, service, action, args) {
    ...
    if (successCallback || failCallback) {
        callbackId = service + cordova.callbackId++;
        cordova.callbacks[callbackId] =
            {success:successCallback, fail:failCallback};
    }
	  ...
}
```

### 原生代码触发 JavaScript

不论是 iOS 还是 Android，最后都可以直接调用相应的 JavaScript 函数。于是，在被调用之后我们只需要进行对应的处理即可：

```javascript
callbackFromNative: function (callbackId, isSuccess, status, args, keepCallback) {
    try {
        var callback = cordova.callbacks[callbackId];
        if (callback) {
            if (isSuccess && status === cordova.callbackStatus.OK) {
                callback.success && callback.success.apply(null, args);
            } else if (!isSuccess) {
                callback.fail && callback.fail.apply(null, args);
            }
            if (!keepCallback) {
                delete cordova.callbacks[callbackId];
            }
        }
    } catch (err) {
        ...
    }
}
```

相应的逻辑便是，根据 callbackId 获取对应的 callback 方法。如果存在 callback 方法，则调用相应的成功或者失败回调，再根据需要决定是否删除对应的回调方法。

## 架构设计方案

为了便于 Android 和 iOS 统一，并且能快速实现，我们直接使用了 ``iframe`` 来创建请求。目前市面上的一些混合应用框架，采用的都是类似的形式，如豆瓣的 rexxar-web，相关的代码如下：

```javascript
export function callUri(uri) {
  let iframe = document.createElement('iframe');
  iframe.src = uri;
  iframe.style.display = 'none';
  document.documentElement.appendChild(iframe);
  setTimeout(() => document.documentElement.removeChild(iframe), 0);
}
```

即创建了一个 iframe，随后添加到 WebView 中就会创建对应的 HTTP 请求，然后立马删除了这个 iframe。过程中，原生 APP 就能捕获到对应的请求，并进行处理。由于在某些版本的 Android 系统中，timeout = 0 太短，需要改长一些。

对于 callbackId 的处理，则相似的，创建一个数组，然后添加值进去。

```javascript
if (typeof callback == 'function') {
    callbackId = 'cb_' + (uniqueId++) + '_' + new Date().getTime();
    callbackArray[callbackId] = callback;
}
```

再把 callbackId 添加到 URL 中即可。对应的，我们要调用的方式也比较简单：

```javascript
...
callbackArray[callbackId](result.result);
...
```

由于 callback 在 Angular 框架上，不能被 Zone.js 监测到。我们添加了 ``Promise`` 的支持，即当：

```javascript
if (typeof callback == 'function') {
  ...
} else {
	return new Promise(resolve: any, reject: any) => {
	   ...
	});
}
```

## 结论

对于一个多方协作，跨域不同平台的系统设计来说，这并不是一件容易的事。在这个方案里，是 Android  和 iOS 对于不同请求方式的处理，是一个比较复杂的地方。

