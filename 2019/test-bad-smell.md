# 测试代码的坏味道
测试代码才能真正体现开发人员的水平。

> 追求技术卓越是采用敏捷的第一成功要素。 —— Jeff Sutherland 敏捷宣言创始人之一

 - Phodal:   “你为什么写测试？”
 - 开发人员 A：“为了测试覆盖率”。
 - Phodal：  “咦，这个测试没有断言”
 - 开发人员 A 笑了笑。

某次代码重构中，我发现代码的测试覆盖率很高，过程中出了一些错误，重构手法不正确是一个问题。但是在重构的过程中，发现有些测试都是没有意义的，所以我变转向开始研究测试坏味道，顺便在 Coca 中写了个识别代码测试坏味道的工具。

## 测试反应开发人员的水平

与编写业务代码相比，测试代码才能真正体现开发人员的水平。你可以用测试来判断开发人员的水平：

 - 有没有为自己的代码编写测试？
 - 测试中有没有断言？
 - 测试中有没有包含有效的断言？
 - 测试的长度是否正常？
 - 测试中的断言是否合理？

没有断言的测试意味着原本的代码写得又臭又长；测试中只包含无效断言表明开发人员在划水；测试方法的长度过长，表明原有的方法可以进一步抽象……

顺便一提，我们推荐的 TDD（测试驱动开发），它并非是银弹。但是，当你来面对一个复杂的场景时，它可以驱动出可测试的代码，辅助以重构，能帮助你写出短小的函数。借此整体上降低整一部分代码的开发 + 维护成本。

我知道你想说有人的很聪明，可以写出的代码足够的健壮。但是呢，这样的人存在吗？即使存在的话，需求是善变的，下一次接手代码的人能保证原有的功能是好的吗？

### 正视测试同正视 bug 一样

> 软件测试（英语：Software Testing），描述一种用来促进鉴定软件的正确性、完整性、安全性和质量的过程。换句话说，软件测试是一种实际输出与预期输出间的审核或者比较过程。

项目代码是日常接触最多的部门，我们会直面代码中的问题，也因此会重视它们。对于测试嘛，就呵呵了。但是，你们就这么忽略了测试的重要性。

我们编写测试是为了提升软件开发质量，一旦代码改出了问题，那么测试就会帮我们找出破坏了的原有功能。而不是在长长的软件测试反馈链之后，才发现：原来我们改出了 bug。

不过呢，当你的业务进度压力大的时候，没有时间编写测试，反而 bug 就更多了。

## 测试代码坏味道

> 代码坏味道是对应于系统中的更深层问题的表面指示。

我们一般谈论代码坏味道的时候，主体是项目代码，而测试代码坏味道则往往被人忽略了。测试代码能直观地反应出代码的设计问题，它们是 API 的使用方，它们是 API 的第一等使用方。

> 测试代码坏味道，是指单元测试代码中的不良编程实践（例如，测试用例的组织方式，实现方式以及彼此之间的交互方式），它们表明测试源代码中潜在的设计问题。

如 Robert C. Martin 在《代码整洁之道》所说的那样，好的测试应该是：

 - 快速（Fast），测试应该够快。
 - 独立（Indendent），测试应该相互独立。
 - 可重复（Repeatable），测试应当可在任何环境中通过。
 - 自足验证（Self-Validating），测试应该有布尔值输出。
 - 及时（Timely），测试应该及时编写。

要我说的话，它应该还有：

 - **同一人编写**，测试应该由开发业务代码的编写。这样他/他们才知道自己代码写得烂。
 - **边界**，测试直接不影响业务代码。这里指的主要是 private -> public 的行为，又或者是业务代码中包含测试代码，而非因为测试对原有代码重构。
 - **有效命名**。测试信息应该体现在方法名上，表达某一个特定的需求。

测试代码应该遵循生产代码的质量标准。

命名在测试中也是一大难题，我们如可以采用 Roy Osherove（《单元测试的艺术》作者） 推荐的 `UnitOfWork_StateUnderTest_ExpectedBehavior` [命名法则](https://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html)。 几个示例如下：

```
Public void Sum_NegativeNumberAs1stParam_ExceptionThrown()
Public void Sum_NegativeNumberAs2ndParam_ExceptionThrown ()
Public void Sum_simpleValues_Calculated ()
```

## 测试代码坏味道示例

先让我们来看看有哪些常见的测试坏味道：

 - 空的测试。测试是生成的，但是没有内容。
 - 忽略的测试。即测试被 Ignore
 - 没有断言的测试。为了测试覆盖率而出现的测试
 - 多余的 Println。调试时留下的讯息。
 - 多重断言。每个测试函数只应该测试一个概念。
 - ……。

然后，再来个 Examples。

这是 Arduino 代码中的 `I18NTest.java` 文件，先看看文件，再看看问题：

```
  @Test
  public void testAllLocales() {
    for (Language language : Languages.languages) {
      if (!language.getIsoCode().equals("")) {
        Locale locale = toLocale(language);
        ResourceBundle bundle = ResourceBundle.getBundle("processing.app.i18n.Resources", locale);
        if (locale.equals(bundle.getLocale())) {
          Collections.list(bundle.getKeys()).stream().map(bundle::getString).filter(key -> !key.contains("<html")).forEach(key -> {
            try {
              I18n.format(key);
            } catch (IllegalArgumentException e) {
              System.out.println(language);
              System.out.println(key);
              throw e;
            }
          });
        } else {
          System.out.println("Missing locale: " + locale);
        }
      }
    }
  }
```

问题有很多：

 - 没有断言
 - 多余的 print 函数
 - try...catch...
 - 糟糕的测试命名

这个测试的正确作法之一应该是：使用容器 collection 来过滤出正确的语言，最后对比长度是否正确。

再举个例子：

```
@Test
public void testXmlSanitizer() {
    boolean valid = XmlSanitizer.isValid("Fritzbox");
    assertEquals("Fritzbox is valid", true, valid);
    System.out.println("Pure ASCII test - passed");

    valid = XmlSanitizer.isValid("Fritz Box");
    assertEquals("Spaces are valid", true, valid);
    System.out.println("Spaces test - passed");

		...
}
```

这个测试用例违反了每个测试一个用例的原则。

## 坏味道检测工具

欢迎成为 Coca 的忠实用户，只需要运行 `coca tbs`，就可以识别出你的 Java 代码中的测试味道。如下是 Arduino 源码中的测试坏味道：

|        TYPE         |                           FILENAME                            | LINE |
|---------------------|---------------------------------------------------------------|------|
| DuplicateAssertTest | app/test/cc/arduino/i18n/ExternalProcessOutputParserTest.java |  107 |
| DuplicateAssertTest | app/test/cc/arduino/i18n/ExternalProcessOutputParserTest.java |   41 |
| DuplicateAssertTest | app/test/cc/arduino/i18n/ExternalProcessOutputParserTest.java |   63 |
| RedundantPrintTest  | app/test/cc/arduino/i18n/I18NTest.java                        |   71 |
| RedundantPrintTest  | app/test/cc/arduino/i18n/I18NTest.java                        |   72 |
| RedundantPrintTest  | app/test/cc/arduino/i18n/I18NTest.java                        |   77 |
| DuplicateAssertTest | app/test/cc/arduino/net/PACSupportMethodsTest.java            |   19 |
| DuplicateAssertTest | app/test/processing/app/macosx/SystemProfilerParserTest.java  |   51 |
| DuplicateAssertTest | app/test/processing/app/syntax/PdeKeywordsTest.java           |   41 |
| DuplicateAssertTest | app/test/processing/app/tools/ZipDeflaterTest.java            |   57 |
| DuplicateAssertTest | app/test/processing/app/tools/ZipDeflaterTest.java            |   83 |
| DuplicateAssertTest | app/test/processing/app/tools/ZipDeflaterTest.java            |  109 |

Coca 开源，不要钱，不要钱。对 Coca Pro 有兴趣的，可以和我们联系，哈哈哈哈。

## 结论

回到开头：《敏捷宣言》上的原文是，『坚持不懈地追求技术卓越和良好设计，敏捷能力由此增强』。只是对于工具、手法和模式的理解，何时去使用各种各样的技术，以及考虑产品、需求的意图，大抵就是技术卓越的体现。

相关资料：https://testsmells.github.io/index.html ，我从这个网站上获得了 Coca 项目所需要的大量用例代码。
