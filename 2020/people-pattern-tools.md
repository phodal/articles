# 精炼：如何打造有用的工具？

过去的几年里，我一直在打造各式各样的编程相关的工具。这些工具有的是用于指导软件开发工作，有的是用来进行编程学习，还有的纯粹是为了提升技术而写的。在我写了越来越多的工具，接触了越来越多的工具思路之后。 我便想写一篇文章，用于记录一下过程中发生的一些变化。

## 如何打造工具？

如果你拥有广泛的技术栈知识，还有相对充裕的时间，那么加上一些激情，你就能写出一个不是那么差的工具。

### 工具的技术栈

在我短短十几年的编程生涯中，我尝试了不同的层级技术栈，大抵也是了解怎么从底层到顶层做各种工具。连接物理世界的工具：

 - 纯嵌入式系统编程。只需要一个 Arduino IDE + 一个 Arduino 开发板，配合各式各样的输入设备（如传感器），配合一些输出，这就是一个简单的计算机原型。
 - 嵌入式操作系统编程。对于复杂的场景来说，你还需要一个简单的操作系统，以帮助你进行任务调度。你可以简单的写一个任务调度，又或者是根据你的能力基于 uCOS、FreeRTOS、Contiki、Zephyr 等系统开发应用。
 - 基于 GNU/Linux 的嵌入式系统编程。对于大部分人来说，只需要一个 Raspberry Pi + Python 就可以编写一个高性价比的机器学习相关的应用；又或者是在路由器上运行 OpenWRT 这样的操作系统，这种性价比更更高的方案。
 - 编译 GNU/Linux 操作系统。如果时间充裕，可以用诸如于 Linux From Scratch 这样的工具，自己编译一个自用的操作系统。
 - 物联网时代的嵌入式编程。我喜欢使用 ESP32 （ESP8266 的继任者）来搞搞智能家居，它们自带 WiFi 和蓝牙，有各种模拟现有设备的方案，配置上 HomeKit。考虑到成本原因，对于没有硬件基础的开发者来说，采用 Android Things 又或者是 Windows IoT 是一个更简单的选择。我没有玩过 Fuchsia，它可能也是不错的，哈哈。

对于大部分开发者来说，连接物理世界是一项昂贵的事，毕竟硬件太贵了（考虑一下 Raspberry Pi，它也相当的不错）。于是乎，我们所能做的就是在操作系统之后，开发一些工具。

 - 桌面应用。过去我尝试使用 QT 来开发一些桌面应用，后来我改为了 PyQT / PyGtk + Python，而现在我都转向了 Electron，用 Web 技术来开发桌面应用就是这么简单。
 - 移动应用。尽管 React Native、Flutter 是一个非常不错的移动应用框架，我也用它们开发了一系列的应用。但是，从架构上来说，我偏向于使用混合式架构的应用，Flutter + Ionic / Angular，或者是 RN + Ionic / Angular。
 - 小程序。我讨论小程序，它们都有各种审查。
 - 命令行界面应用（CLI）。对于日常工作来说，我们只需要一个简单的命令行，对于前端来说，也许 Node.js 就够了，对于后端来说，也许 Python 就够用了；不过，现在我更喜欢用 Go 来开发 CLI 应用。
 - Web 应用。对于我们而言，考虑到跨平台特性，我往往使用后端 Serverless + 前端 Angular + 微前端架构来开发 Web 应用的工具。
 - 浏览器插件。偶尔我也会开发一些浏览器插件，但是当我切换到 Firefox 浏览器之后，我正在考虑怎么迁移旧的 Chrome 插件。

愿意这些花式的介绍能给你的新工具带来一些想法。然后你就可以把这些知识串起来，开发一些有意思的应用：

 - 基于 Arduino + Raspberry Pi 的持续集成告警灯。
 - 通过 ESP8266 模拟各种硬件，部署个服务器，来实现远程。
 - ……

现在，我们已经离题很远很远了，回到正题上。

### 找一个想法

配合上上述的技术栈，你就可以轻松地开发一个工具。

完了？

还没有

还有一半的内容

## 工具的开发模式

对于开发工具来说，存在一些特别固定的开发模式。我大概也经历了三个阶段，它们大概是三种不同的模式：

 - 『随心所欲』造轮子模式。即，我爱怎么做，我就怎么做，我缺什么，我就加什么。
 - 『转化原则与模式』的模式。我从某些地方，寻找一些原则与模式，并将它们沉淀到工具里。
 - 『标准化特定流程』模式。我将过程、流程转变为工具，以弱化人在过程中的作用。

### 『随心所欲』造轮子

没啥可说的，我爱怎么做就这么做。但是，它有这么一些点，你可以去玩：

 - 积累技术和素材。要随心所欲地造轮子并不是一件容易的事情，你要有强大的学习能力，以实现自由自在的目的，上要会焊电阻，下要会拿锤子。
 - 寻找现成、学习现有的工具。你可以在它们身上学习到一些优点。
 - 构建、并持续完善常用的工具。通过持续的使用，你就可以完善这个工具，直到它顺手。突然间，我有了一个想法，自己写一个浏览器（基于 Electron），笑~。如果提升技术对于你来说很重要，那么这是一个非常不错的提升点。

然后， 你就可以和我一样，开开心心地天天看着自己的 bug 在自己的电脑上复现，然后觉得在其它电脑上应该是好的。等我有空的时候，我再来修这个 bug 吧。

### 转化原则与模式

这个大抵是去年在造工具的一大收获。当时和公司的同事一起讨论造工具的时候，讨论出了沉淀出原则与模式，然后将使用工具来承载它们。

原则与模式这种东西，本身就是我们对于日常工作的一些沉淀。所以，它们特别容易被转换到工具上。我们也可以写一个工具，它用于介绍各种原则与模式，狗头。

转化原则与模式的另外一大意义是，告诉你：**你可以通过学习别人，来打造出自己的工具**。就这么来说，去年我在写 Coca 的时候，我做了这么一件事：

 - 寻找各种 paper。
 - 下载各种 paper。
 - 阅读各种 paper。

然后，我就把各种 paper 转换到我的工具中了。虽然，大部分地 paper 都写得特别水（我觉得读完 100 篇之后，写出一个工具之后，我能写出一篇比 99% 的都强），但是我们就轻松地 copy 了别人几个月的研究经验。

从书中读也是一个不错的主意，但是大部分技术书偏向于实践为主，比较难转换。

### 标准化特定流程

如果说，前两者是造轮子的话，那么标准化流程做的是平台。我最近在研究各种成熟度模型，它们有五个阶段。我更喜欢 GitHub 官方写的一个开源成熟度模型的定义：

 - 临时（Ad-hoc） —— 新的或未记录的过程是不受控制、反应性的和不可预测的，通常是由个人驱动而没有协调或沟通。成功取决于个人英雄主义。
 - 管理（Managed）—— 流程已部分记录在案，有可能导致一致的结果。成功取决于纪律。
 - 已定义（Defined）—— 记录，标准化流程并将其集成到其他流程中。成功取决于自动化。
 - 度量（Measured）—— 对过程进行定量管理。成功取决于根据业务目标衡量指标。
 - 已优化（Optimized）—— 通过增量和创新更改，该过程正在持续可靠地得到改善。 成功取决于减少变革的风险。

抽象完这段话，我们就可以提到这样的过程：

> 个人的实践 -> 团队的实践流程 -> 标准化流程为工具 -> 集成到其它流程，作为平台的一部分 -> 持续完善平台

再一次抽象就是：

> 实践 -> 模式 -> 工具 -> 流程 -> 平台


对，就是这么简单。所以开发这一种模式的工具，只需要寻找现有的成熟度模式，然后就成平台了。

## 结论

没有轮子，哪来的 KPI？

没有技术，哪来的轮子？

没有兴趣，哪来的技术？

KPI 是什么？
