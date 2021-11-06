
Bazel build fails with "" errors https://issuetracker.google.com/issues/126764883

## Clone

```
$ mkdir studio-master-dev
$ cd studio-master-dev
$ repo init -u https://android.googlesource.com/platform/manifest -b studio-master-dev
$ repo sync -c -j4 -q
```

## 编译

从代码库中删除所有的：tools/vendor/google

示例：https://android.googlesource.com/platform/tools/idea/+/refs/heads/studio-master-dev/RELEASE.md

```
--- a/bazel/toplevel.WORKSPACE
+++ b/bazel/toplevel.WORKSPACE
@@ -1,13 +1,6 @@
 load("//tools/base/bazel:repositories.bzl", "setup_external_repositories")
 setup_external_repositories()

-local_repository(
-      name = "blaze",
-      path = "tools/vendor/google3/blaze",
-)
-load("@blaze//:binds.bzl", "blaze_binds")
-blaze_binds()
-
 http_archive(
   name = "bazel_toolchains",
   urls = [
```

## NDK

```
git clone https://android.googlesource.com/platform/prebuilts/ndk tools/vendor/google/android-ndk
 ```
 
## FAG

### SDK 或者 NDK 找不到

```
ln -s ~/Library/Android/sdk/ /Users/fdhuang/jvm/studio-master-dev/prebuilts/studio/sdk/darwin
ln -s ~/sdk/android-ndk-r20/ /Users/fdhuang/jvm/studio-master-dev/prebuilts/studio/sdk/darwin/ndk-bundle
```

### armeabi 问题

```
     [exec] ERROR: /private/var/tmp/_bazel_fdhuang/0711c902f818b6dd779b715988db0de0/external/androidndk/BUILD.bazel:41:1: in cc_toolchain_suite rule @androidndk//:toolchain-libcpp: cc_toolchain_suite '@androidndk//:toolchain-libcpp' does not contain a toolchain for cpu 'armeabi'
```

打开，复制粘贴 

```
      'armeabi': ':arm-linux-androideabi-clang8.0.7-v7a-libcpp',
```

### 删除 DS_Store

```
    ref_id = fd.readline()
  File "/usr/local/Cellar/python/3.7.5/Frameworks/Python.framework/Versions/3.7/lib/python3.7/codecs.py", line 322, in decode
    (result, consumed) = self._buffer_decode(data, self.errors, final)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 3131: invalid start byte
```

find . -name ".DS_Store" -delete
