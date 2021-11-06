# 【架构拾集】混合框架 CLI 设计

build
run
emulate


## Android 

1. 获取 Target 设备。通过执行 ``adb devices``，来返回设备列表。
2. 执行构建。
    1. 进行构建前的依赖项检查。如 JDK、Android SDK、目标设备、Gradle
    2. 清除旧的构建。执行 ``gradlew clean``，并通过 shell.js 删除构建目录。
    3. 执行 ``gradlew build`` 进行构建。
3. 安装应用
    1. 找到构建目录中的 adb 包。
    2. 执行 ``adb install`` 来安装对应的包。
4. 启动应用的 Activity
	  1. 通过 ``elementtree`` 解析 AndroidManifest 来获取包名。
	  2. 执行 ``adb`` 命令来启动应用的 Activity。

## Cordova 代码示例

