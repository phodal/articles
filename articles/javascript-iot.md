> 凡是能用JavaScript写出来的，最终都会用JavaScript写出来。

 —— Atwood定律
 
  在那篇《[最流行的编程语言JavaScript能做什么？](http://mp.weixin.qq.com/s?__biz=MjM5Mjg4NDMwMA==&mid=405412226&idx=1&sn=3bc7a9c6afd166591a90723a1802ed99&scene=21#wechat_redirect)》里，我们列举了JavaScript在不同领域的使用情况，今天让我们来详解一下JavaScript在物联网中的应用。
 
基础：物联网的三个层级
---
 
开始之前， 先让我们简单地介绍点物联网的基础知识。如果你有点Web开发经验的话，都知道下图是CS架构：

![Client-Server架构](http://articles.phodal.com/js-iot/cs.png)

相比于一个物联网系统，无非就是多了一层硬件层以及可选的协调层。

![源自《自己动手设计物联网》](http://articles.phodal.com/js-iot/struct-action.png)

这个硬件层决定了物联网应用比Web应用更加复杂。对于大部分的Web应用来说 ，客户端都是手机、电脑、平板这些设备，都有着强大的处理能力，不需要考虑一些额外的因素。

 对于物联网应用来说，我们需要考虑设备上的MCU的处理能力，根据其处理能力和使用环境使用不同的通信协议，如我们在一些设备上需要使用CoAP协议。在一些设备上不具备网络功能，需要考虑借助于可以联网的协助层，并且还需要使用一些短距离的无线传输协议，如低功耗蓝牙、红外、Zigbee等等。


一个物联网系统：六种语言
---
 
两年半以前，大四，电子信息工程，我选定的毕业论文是一篇关于物联网的论文——《基于REST服务的最小物联网系统设计》。这是一篇入门级的物联网论文，如果大部分学习CS的人有一点硬件基础，都能写出这样的论文。

这篇论文是之前参加比赛的作品论文的“最小化”，里面使用到的主要就是创建RESTful服务，而它甚至称不上是一种技术。在这个作品里：

 - 我们使用Python语言里的Django框架作为Web服务框架，使用Django REST Framework来创建RESTful服务。
 - 为了使用手机当控制器，我们还要用Java写一个Android应用。
 - 我们使用Raspberry Pi作为硬件端的协调层，用于连接网络，并传输控制信号给硬件。
 - 我们在硬件端使用Arduino作为控制器，写起代码特别简单，可以让我们关注于业务。
 - 最后，我们还需要在网页上做一个图表来显示实时数据。

所有的这些，我们需要使用Python、Java、JavaScript、C、Arduino五种语言。而如果我们要写相应的iOS应用，我们还需要Objective-C。
 
![你是在逗我吗？](http://articles.phodal.com/js-iot/6359758744428735171956612167_are-you-serious-wtf-meme-baby-face.jpg	)

JavaScript在物联网领域的发展
--- 

同样的，两年多以前，刚实习，在我们的项目里，我们的新项目里我们使用Backbone作为单页面应用框架的核心来打造Web应用。这时，我开始关注Node.js实现物联网应用的可能性。

![Node.js Express Mongodb](http://articles.phodal.com/js-iot/enm.jpg)

当时，已经有了物联网协议MQTT和CoAP协议的库，于是我照猫画虎地写了一个支持HTTP、CoAP、WebSocket和MQTT的物联网。由于，当时缺乏一些大型应用的开发经典，所以做得并不是很好，但是已经可以看到JavaScript在这方面的远景。

![Ionic Cordova](http://articles.phodal.com/js-iot/cordova-ng-ionic.png)

一年多以前，Ionic还没推出正式版的时候，我发现到了这个框架真的很棒——它自带了一系列的UI，还用NgCordova集成了Cordova的一系列插件。我便开始使用Ionic写了一些移动应用，发现还挺顺手的。接着，我就开始拿这个框架尝试写物联网应用，这需要一些原生的插件，如BLE、MQTT。后来，我也写了一个简单的CoAP插件。

![Iot](http://articles.phodal.com/js-iot/0c1d958622ada18_w960_h557.jpg)

后来我们不再需要编译Node.js，就可以在ARM处理器上运行Node.js。并且我们已经有Tessel、Espruino、Kinoma Create、Ruff这些可以直接运行JavaScript的开发板。三星还推出iot.js，可以让更多的嵌入式设备可以使用JavaScript语言作为开发语言。

![Node.js Future](http://articles.phodal.com/js-iot/jobgraph_node_php_others.png)

人们开始在硬件上使用JavaScript的原因有很多，如Web的开发人员是最多的、JavaScript很容易上手。

现在，这次我们在这三个层级上都可以使用JavaScript，只需要一种语言。

使用一种语言开发物联网应用：JavaScript
---
 
在我写的那本《自己动手设计物联网》中，我就试图去展示JavaScript在这方面的威力。使用Node.js + Node-CoAP + MQTT.js + MongoDB + Express搭建了一个支持多协议的物联网：

![Lan IoT](http://articles.phodal.com/js-iot/iot.jpg)

不过，上图是完善版的物联网，代码自然是在GitHub上啦：[Lan](https://github.com/phodal/lan)。作为服务端来说，Node.js的能力已经是经过验证的。而在混合应用上，仍然也可以经受住考验，混合应用在手机上做个图表是轻轻松松的事（只需要获取数据，然后显示）：

![混合应用图表](http://articles.phodal.com/js-iot/ios-charts.png)

作一个控制端也是轻轻松松的事（我们只需要发个POST请求，更具逻辑一点的就是先获取状态）：

![Led控制](http://articles.phodal.com/js-iot/led-control.png)

而在硬件端，我并没有在书中以JavaScript作为例子来展示JavaScript的用法，因为这会局限了用户的硬件设备。

不过，我们仍然可以使用类似于Johnny-Five这样的库来做硬件方面的编程，只是它没有那么好玩~~。

既然我们可以JavaScript来实现，为什么我们还要喝杯咖啡等它用C编译完呢？

你想知道的答案都在这本书里，已在亚马逊、京东、当当上架：

![自己动手设计物联网](http://articles.phodal.com/js-iot/iot-demo.jpg)

亚马逊：https://www.amazon.cn/dp/B01IBZWTWW

京东：http://item.jd.com/11946585.html

毕竟： 
 
 > 凡是能用JavaScript写出来的，最终都会用JavaScript写出来。
