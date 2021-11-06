我的密码之旅：从统一密码，到云同步的随机密码器。。。
===

取一个变量很纠结，取一个密码很头痛。于是，我开始使用自己的随机密码器，并且它还是“云同步”的。

取一个变量很纠结，取一个密码很头痛，为此我们并不能取一个简单的密码。

![硅谷](silicon-valley.jpg)

而出于以下的背景：

1. 10 年前我使用一个固定的、统一的密码，直到一系列的明文密码泄露事件，我在不同的平台采用了不同的密码。
2. 在经历了一系列的忘记密码之后，我开始采用平台限定的密码，即不同的平台，密码是半动态的。
3. 现今的大部分重要网站都采用了二次验证，或是 Google Authenticator，或是短信验证。
4. 在 ThoughtWorks，我们的密码需要每三个月换一次，它足够的长，并且有一系列的复杂规则。

于是，我终于编写了自己的随机密码器，然后它也是“云同步”的。

统一密码
---

大家都懂的。。。

如果你的所有账号只有一个密码，那么你完了。。。

最好不要拍些艳照什么的。

快速密码：解锁电脑
---

当我还在使用 Windows 时，从来就没有想过，可以拥有这样的一个密码：

```
1qaz2wsx
```

它在输入上特别快速，并且解锁起来又特别的快。

因为，对应于键盘上的：

![1qaz2wsx](1qaz2wsx.png)

它有一些变体，比如说：

```
1234qwer
```

又或者是：

```
1qaz@WSX
```

也会有一些其它高级的变体。

特定平台 + 固定密码
---

10 年前我使用一个固定的、统一的密码，直到一系列的明文密码泄露事件，我在不同的平台采用了不同的密码。在经历了一系列的忘记密码之后，我开始采用平台限定的密码，即不同的平台，密码是半动态的。如下就是一个简单的，对应于不同平台的代码体系，如：

 - 京东：``tb-1qaz2wsx``
 - 淘宝：``jd-1qaz2wsx``

而这样的密码本身也是不安全的，如果有人知道密码规则的话。那么，这无异于一场灾难。于是，我在设计出规则到现在，仅会**在一些重要的网站上**，采用这种规则。

使用密码管理器
---

在几年前，我初次接触到了密码管理：LastPass、1Password——对于普通用户来说，它们是一个相当不错的工具。1Password 可以使用 iCloud 同步，可惜我是个 Android 用户。因此，一旦需要我输入密码到手机的时候，就变得有点可怕。并且 1Password 也有对应的 Chrome 插件，可是 Chrome 的自动填充是另外一个问题——**所以，尽量不要把电脑借给别人**。

然而，1Password 和 LastPass 的密码策略，让我觉得它也是不可靠的——**Master Password**，也就是一个密码可以解锁其它密码。

特定平台 + 生成密码
---

考虑到上面的密码在固定部分是不安全的，我便想到了一种更优雅的方式：基于 Git 的 SHA-1 算法生成的 commit 号，来实现：特定平台 + 生成密码。如下是一个 ``git log`` 的示例：

```
commit b40c83cb772d4b496d1f9ecec136e21d46f6a765
Author: Phodal HUANG <h@phodal.com>
Date:   Tue Mar 13 17:00:11 2018 +0800

    [mooa] log: add basic logic

commit 3c8badf0bd6671368910fb22eada5a850a3fcf88
Author: Phodal HUANG <h@phodal.com>
Date:   Tue Mar 13 16:46:53 2018 +0800

    [mooa] rename staus filter to status helper
```

其中的：``commit b40c83cb772d4b496d1f9ecec136e21d46f6a765``，即对应了一个随机密码。于是我们的密码就变成了：

 - 京东：``tb-b40c83cb``
 - 淘宝：``jd-3c8badf0``

当然了 ``btoa('jingdong')``，也是一个不错的实践：``amluZ2Rvbmc=``。

作为一个程序员，最后我采用的策略是，半动态的密码 + 生成的密码。

代码生成密码 + GitHub 存储密码
---

虽然，我不知道怎么样的密码是真的安全的，但是我觉得一个二十位左右的密码就是安全的。于是我编写了一个七行的 Python 脚本来生成一个 12 位的密码：

```python
from random import choice
from string import hexdigits, punctuation
import time

base_time = str(time.time())[12:16]
base_string = ''.join(choice(hexdigits) for i in range(6))
base_punctuation = ''.join(choice(punctuation) for j in range(2))

print(base_string + base_punctuation + base_time)
```

首先，生成器将从时间戳里，取第 12~16 位的字符，如下：

```
1520944501.967865
```

接着，生成器将从十六进制数字中，随机取 6 位：

```
0123456789abcdefABCDEF
```

最后，再从所有的特殊字符中取出几个字符：

```
!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
```

就有了这部分的密码：

```
09a86a{#4242
```

PS：我使用的是 Python 3，你们这些 Python 2.6 的异教徒。

最后，需要有一个地方来同步我的密码，于是，我将密码存储在了 GitHub 上了——我的 GitHub 需要二次验证。

结论
---

最后，请不到你的密码贴在笔记本上：

![Password in Latop](password-in-latop.jpg)
