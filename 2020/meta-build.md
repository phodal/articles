构建要素：

1. 软件源
2. 代码
3. 构建工具

因素：

1. 构建速度
2. 增量构建

## Bazel

如 JDK

Bazel 是类似 于Make，Maven 和 Gradle 的开源构建和测试工具。它使用人类可读的高级构建语言。 Bazel 支持多种语言的项目并为多种平台构建输出。 Bazel 支持跨多个代码库的大型代码库以及大量用户。
为什么要使用Bazel？

Bazel具有以下优点：

 - 高级构建语言。 Bazel 使用一种抽象的，人类可读的语言以较高的语义级别描述了项目的构建属性。与其他工具不同，Bazel 在库、二进制文件，脚本和数据集的概念上进行操作，使您免于编写对工具（例如编译器和链接器）的单独调用的复杂性。
 - 快速可靠。 Bazel 缓存所有以前完成的工作，并跟踪对文件内容和构建命令的更改。这样，Baze知道何时需要重建某些内容，并且仅重建该内容。为了进一步加快构建速度，您可以将项目设置为以高度并行和增量的方式构建。
 - 多平台。 Bazel 在 Linux，macOS 和 Windows 上运行。 Bazel 可以从同一项目为多个平台（包括台式机，服务器和移动设备）构建二进制文件和可部署的程序包。
 - 可伸缩。 Bazel 在处理包含 10 万多个源文件的构建时可保持敏捷性。它可以与数以万计的多个存储库和用户群一起使用。
 - 可扩展的。支持多种语言，您可以扩展Bazel以支持任何其他语言或框架。

## 过程

[https://mirror.bazel.build/](https://mirror.bazel.build/) 会返回官方定制的源

```bash
Current running Bazel is ahead of bazel-toolchains repo. Please update your pin to bazel-toolchains repo in your WORKSPACE file.
DEBUG: /private/var/tmp/_bazel_fdhuang/94cec4e4014a3e8b14681128f90e6d71/external/bazel_toolchains/rules/rbe_repo/checked_in.bzl:103:14: rbe_ubuntu1604_java8 not using checked in configs as detect_java_home was set to True
Analyzing: target //src:bazel (303 packages loaded, 8567 targets configured)
    Fetching @openjdk_macos_minimal; fetching 42s
    Fetching https://mirror.bazel.build/...ulu11.37.17-ca-jdk11.0.6-macosx_x64-minimal-b23d4e05466f2aa1fdcd72d3d3a8e962206b64bf-1581689063.tar.gz; 10,107,697B 41s
```

开始打包：

```bash
INFO: Analyzed target //src:bazel (304 packages loaded, 8579 targets configured).
INFO: Found 1 target...
[201 / 2,153] 8 actions running
    Building third_party/aws-sdk-auth-lite/libaws-sdk-auth-lite.jar (75 source files); 5s worker
    Compiling Java headers .../com/google/devtools/build/lib/skyframe/serialization/autocodec/libautocodec-annotation-hjar.jar (1 source file); 2s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/windows/jni/libfile-hjar.jar (1 source file); 1s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/windows/jni/libprocesses-hjar.jar (1 source file); 1s darwin-sandbox
[213 / 2,153] 8 actions, 7 running
    Compiling external/com_google_protobuf/src/google/protobuf/compiler/java/java_enum_field_lite.cc [for host]; 4s darwin-sandbox
    Compiling Java headers src/java_tools/singlejar/java/com/google/devtools/build/zip/libzip-hjar.jar (16 source files); 3s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/skyframe/libmutable_supplier-hjar.jar (1 source file); 2s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/remote/libReferenceCountedChannel-hjar.jar (1 source file); 1s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/bazel/rules/ninja/file/libfile-hjar.jar (9 source files); 1s darwin-sandbox
    Compiling Java headers src/main/java/com/google/devtools/build/lib/windows/libwindows_short_path-hjar.jar (1 source file); 1s darwin-sandbox
    Compiling external/com_google_protobuf/src/google/protobuf/extension_set_heavy.cc; 0s darwin-sandbox
```

## 定义

### WORKSPACE

### BUILD





