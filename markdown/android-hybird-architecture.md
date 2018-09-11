# （草稿） 【架构拾集】简易的 Android 混合应用框架的插件架构设计

在上一篇文章《[【架构拾集】WebView 的 JavaScript Bridge 设计](https://www.phodal.com/blog/architecture-books-build-hybird-framework/)》里，我们介绍了 Android、iOS JavaScript Bridge 的一些 相关设计。

这篇文章，则针对上一篇文章，做一些相应的插件管理机制相关的补充。该方案基于 Cordova 框架的插件管理方案，做了尽可能的精简。

## 技术远景

作为项目的开发人员，我们希望拥有一个模块化的方案，及对手的插件管理机制，它在解耦应用代码的同时，可以提供更好的插件化管理方案。

## 架构设计

参考了 Cordova 的设计——通过类名（即插件名）来创建对应的实例（newInstance），对应的插件机制加载主要的代码如下所示：

```java
private CordovaPlugin instantiatePlugin(String className) {
    CordovaPlugin ret = null;
    try {
        Class<?> c = null;
        if ((className != null) && !("".equals(className))) {
            c = Class.forName(className);
        }
        if (c != null & CordovaPlugin.class.isAssignableFrom(c)) {
            ret = (CordovaPlugin) c.newInstance();
        }
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println("Error adding plugin " + className + ".");
    }
    return ret;
}
```

对应的，我们只需要创建自己的 ``CordovaPlugin`` 类即可，使用时调用对应的  ``execute``  方法：

```javascript
import java.util.Map;

public interface BridgePlugin {
    void execute(Map parameters, BridgePlugin bridgePluginCallback);
}
```

在完成执行结束后，则调用相应的回调方法即可：

```javascript
import com.alibaba.fastjson.JSONObject;

public interface PluginCallback {
    void sendPluginResult(JSONObject message);
}
```

即对应于如下的代码：

```java
locationPlugin.execute(pluginParams, message -> {
    buildData(message, callbackId);
    offer("success");
});
```

接着我们面临的挑战是，如何下载这些插件？于是，便编写了一个简单的 gradle 脚本来测试：

```gradle
ext.addPlugins = { plugins ->
    delete "${buildDir}/bridge-plugins/"
    def hasCorePlugins = false
    def sidePlugins = []
    def corePlugins = []
    def pluginsSource = "https://github.com/phodal"
    def CORE_STRING = "core."

    for (plugin in plugins) {
        if(!plugin.startsWith(CORE_STRING)) {
            sidePlugins.push(plugin)
        } else {
            hasCorePlugins = true
            corePlugins.push(plugin)
        }
    }

    if (hasCorePlugins) {
        def git = Git.cloneRepository()
                .setURI(pluginsSource + "/core-plugins")
                .setDirectory(new File("${buildDir}/bridge-plugins/core"))
                .call()

        for (corePlugin in corePlugins) {
            def pluginName = corePlugin.substring(CORE_STRING.size(), corePlugin.size())
            println "handle plugin:" + pluginName
            copy {
                from "${buildDir}/bridge-plugins/core/" + pluginName
                into "${projectDir}/app/src/main/java/com/phodal/plugins/core/" + pluginName
            }
        }
    }

    if (sidePlugins.size > 0) {
        for (sidePlugin in sidePlugins) {
            println "handle plugin:" + sidePlugin
            def git = Git.cloneRepository()
                    .setURI(pluginsSource + "/" + sidePlugin)
                    .setDirectory(new File("${buildDir}/bridge-plugins/side/" + sidePlugin))
                    .call()

            copy {
                from "${buildDir}/bridge-plugins/side/" + sidePlugin
                into "${projectDir}/app/src/main/java/com/phodal/plugins/side/" + sidePlugin
            }
        }
    }
}
```

对应的插件下载逻辑是：

1. 读取插件列表，按照是否以 ``core.`` 开头来判断是不中是模块化组件。
2. 如果是模块化的组件，他们是单独的一个项目
3. 如果不是模块组件，则是一堆 API 在同一个项目里
4. 下载完后，将代码复制到项目里


