# Go

过去的一两周里，被公司的大佬安利了 Go，用来写一个代码、架构分析和自动化重构的工具。经过这么一周对于 Go 语言的实战，我算是有底气来写一篇文章来介绍（吐槽）一下 Go 语言。

## 优点 1：跨平台支持（交叉编译）

对于习惯了创造轮子的我而言，起先我觉得使用 Go，又或者是其它语言并没有多大的区别。直到我看到了，大佬在 macOS 下构建出了 Windows、GNU/Linux 和 macOS 等系统的可执行文件的时候，我觉得 NBility现在，我已经把 Go 列为了**我的 CLI 工具的首选语言**。

如果你和我一样，尝试过开发一系列的跨平台应用，那么就会心累于跨平台开发。如：

1. **使用 Electron 开发跨平台应用**。除了开发机器，你还需要一个目标操作系统（如 Windows），以及对应的开发套件（Git、Node.js、Python 等）你才能完成目标操作系统上的构建。
2. **使用 Java 开发 GUI 应用**。我需要在目标机器上配置 Java 环境，才能运行 jar 包。虽然大部分操作系统已经安装有了 Java 环境，但是谁还会用它来开发 GUI 应用。
3. **使用JavaScript / TypeScript 开发 CLI 工具**。嗯，我需要先安装有一个 Node.js 或者是 Deno，才能运行起我的代码，还有对应的 Node.js 的版本管理工具。
4. **使用 Shell 来编写 CLI**。不好意思，我还没学会怎么编写 Windows 上的 BAT 工具。
5. ……

当然了，有了 Docker 上面的都是问题，不过你还得先安装一下 Docker。从这种角度上来说，它和我们使用的 Cordova、React Native、Flutter 相近，只是 Go 更高级一些。因为前三者目前都不能在 Windows 上编译出 iOS 应用。而对于 Go 来说，你只需要运行：

```
GOOS=windows GOARCH=amd64 go build fanta_cli.go
```

你就能得到一份在目标操作系统上的可执行文件。其中的 ``$GOARCH`` 是指目标平台（编译后的目标平台）的处理器架构（386、amd64、arm），``$GOOS`` 是指目标平台（编译后的目标平台）的操作系统（darwin、freebsd、linux、windows）

根据官方的 ``syslist.go`` 显示，它支持这么一些操作系统和处理器架构：

```go
const goosList = "aix android darwin dragonfly freebsd hurd illumos js linux nacl netbsd openbsd plan9 solaris windows zos "
const goarchList = "386 amd64 amd64p32 arm armbe arm64 arm64be ppc64 ppc64le mips mipsle mips64 mips64le mips64p32 mips64p32le ppc riscv riscv64 s390 s390x sparc sparc64 wasm "
```

是的，你可以尝试使用 Go 来玩转 WebAssembly 了。

## 优点 2：部署简单

使用 Go 语言编写应用时，你的目标是一个可执行的二进制文件，只需要直接 blabla 就可以运行了。

而，如果你想使用一个在 GitHub 运行的软件时，你也可以直接运行命令来获取，就可以直接构建了，如：

```
go get github.com/phodal/coca
```

理想情况下，就可以获得一个可执行的 ``coca`` 命令。不过，我这个项目由于使用的是相对路径，导入有一点问题。

## 优点 3：原生支持并发

还没体会到，我先去玩 WebAssembly。

## 缺点 1：依赖管理

Go 有依赖管理，哦，不 Go 没有依赖管理。

 - ``go dep`` 垃圾。
 - ``go vendor`` 垃圾。
 - ``go modules`` 垃圾，哦，终于不需要 vendor 和 GOPATH，把依赖下到了项目目录了。但是，相对路径引用……。

嗯，感觉一下，我要这么引用自己项目中的依赖：

```
import (
	. "github.com/phodal/coca/refactor/move_class"
	. "github.com/phodal/coca/refactor/rename"
	. "github.com/phodal/coca/refactor/unused"
)
```

Java 有包依赖冲突，JavaScript 有依赖地狱。但是 Python 和 Go 官方一直没能解决好依赖问题。

## 缺点 2：语法设计

好吧， 这个问题每个语言都有，Go 语言最麻烦的是去构建一个函数，因为 Go 没有构建函数。所以，你需要这么去写你的类：

```
type RemoveUnusedImportApp struct {
  ...
}

var nodes []JMoveStruct

func (j *RemoveUnusedImportApp) Analysis() { {
	...
}

```

但是用起来嘛，也不方便，我还是习惯 New 一下，然后直接调用：

```
func NewRemoveUnusedImportApp(config string, pPath string) *RemoveUnusedImportApp {
	return &RemoveUnusedImportApp{}
}
```

其它的语法还都好， 还好。

## 其它

好吧，我知道 Go 还有其它优点，如性能更好，但是这都是相对来说的。除非你的语言库里只有一种语言，这才会成为你所谓的优点。

没有银弹。

