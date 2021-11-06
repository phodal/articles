# 组件化架构的坑与经验

坑多了，想吐槽一下。

## 应对组件化架构的那些经验

在日常的项目开发中，关于组件有这么一些常见的使用方式：

1. 选定一个**基础的组件库**
2. 当我们需要在基础的组件上添加功能时，需要封装出自己的组件
3. 当我们找不到合适的组件，则需要自己编写组件
4. 基础组件库不存在的组件，但是存在第三方组件库，则需要进行二次封装
5. 自己写的组件有 bug，于是修改，引出新的 bug
6. 写的组件太臃肿了，拆分成两个组件；两个组件功能重复，于是合并成一个组件。
7. 反复重复 2~6 步，或者只重复第 5 步，又或者是只重复第 6 步。

是的，我们主要在处理的是：难以预料的 bug。

### 组件的二次封装：装饰器模式

首先，我们不可能会花时间去二次封装所有的组件——除非，我们打算封装自己的组件库。然后，对于那些我们需要修改的组件来说，直接进行二次封装，倒也没有什么可说的。剩下的，我们要讨论的便是：**第三方组件**。

在大部分的项目里，我们进行组件库选型的时候，往往选择的是开源、免费、可商用的组件。而一些基础的组件库，往往不包含一些复杂的组件，诸如于表格、富文本编辑器等组件。又或者是，某个组件的行为不符合的预期，于是乎便需要寻找一个第三方组件。

对于这些第三方组件来说，也没有什么好说的，一律进行二次封装：

 - 按需封装输入和输出
 - 按需修改成自己的样式
 - 尽可能通用化，而不是依赖于三方组件封装

这样做的目标是，降低因为三方组件的修改带来的风险——它们看上去没有基础组件库，那么强大，有强力的组织支持，笑~。万一哪天出问题了，我们还可以自己接盘来维护。

从某种意义上来说，组件的二次封装也算是组件的组合，只是呢，它适用的场景不太一样。从名称上来说，二次封装第三方组件会比组合第三方组件要适用得多。

### 业务组件的拆与合

> 天下大势，分久必合，合久必分。

组件的拆与合，仍然是我很痛苦的一个话题。哪怕是做了很多的项目，趟了很多的坑，我都没有想到最佳实践。于是乎：

 - 我们觉得 A 组件和 B 组件是同样的功能，只是参数不一样。便写成了同一个组件，后来我们便开始吐槽设计了。
 - 我们觉得 A 组件和 B 组件是两个不同的组件，于是就变成了两个组件。可是，业务一番修改之后，它们可能只存在 head 上的差异。

能大概总结一下的也只有：

 - 如果只有两三个基础属性，如样式或者显示，那么合成一个组件的话，问题并不是那么大。
 - 如果存在大量的差异，而且表现和行为又有比较大的差异，那么拆分成两个组件可能是一个不错的选择。

除此，还要看使用的地方 ，比如详情页和列表页的某些部分，在设计的初期是同样的，但是它们可能早晚会不一样——不过也可能是一样的，哈哈哈。

**过度设计**和**设计不足**，是我们经常会遇到的坑。见招拆招，大抵是我们所能做的。除此，最有意思的是，一旦我们拆成了两个组件之后， 我们就不太起合回去了，哈哈哈哈。

### 组合或继承

组件的组合，即在引用子组件的基础上，添加一些新的行为——可能还需要在父组件上添加输入和输出。

组件的继续，即通过继续父组件，来添加一些新的行为。

**继承**

对于组件来说，我们关注于两部分的内容：**输入** 和 **输出**。因此对于使用 Angular 的组件，只需要父组件暴露出 ``inputs`` 和 ``outputs``，便可以很方便地继承输入和输出。

```typescript
@Component({
  selector: 'app-clean-navbar',
  inputs: [...NavBarMeta.inputs],
  outputs: [...NavBarMeta.outputs],
  templateUrl: './clean-navbar.component.html',
  styleUrls: ['./clean-navbar.component.scss']
})
```

除此，由于大部分的 Angular 项目，使用的是 TypeScript。因此 inputs 和 outptus 等参数，也可以通过注解来拿到相应的值。

鉴于过去维护某个 10 年遗留系统的痛苦经历，我对于继承并不是那么热衷——当我们使用过多的继承时，会遇到一个天然的坑：父类层层调试。不过，如果只有一两层的继承关系，那到底是没有多大问题的。

于是乎，一个简单的方法就是使用组合——在需要的部分，加上定制的属性。

```javascript
function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

看上去两者好像都没有多大区别，不过按 React 官方的文档所说：

> 在 Facebook ，我们在千万的组件中使用 React，我们还没有发现任何用例，值得我们建议你用继承层次结构来创建组件。

## 吐槽一下组件化的坑

在实施组件化架构时，常常会有两种开发模式：

1. 组件与业务功能分离。即组件的开发者与业务功能的开发者，并非是同一批人。
2. 组件与业务功能绑定。即实现组件的开发者，也是实现业务功能的人。

而组件的开发则依赖于编程经验。因此功能分离，可能会导致组件的消费者需要修改组件；功能绑定，则可能会导致组件与业务功能绑定。

于是乎，我们经常会遇到各种问题。

### 未复用的组件：散弹式修改

开发人员最讨厌的就是**散弹式修改**，即：每遇到某种变化，你都必须在许多不同的 classes 内做出许多小修改以响应之（《重构》）。

如果我们不复用组件时，就会遇到这样的坑。一一修改可能并不痛苦，痛苦的是：**你可能会改漏了**。因为不是所有的人，都会对代码库特别熟悉。

### 过度复用的组件：一波未平，一波又起

过度复用，它就是反复用模式，基本上我是最常做的和遇到的情形。但是如果大家都有封装的意识，那么这个开局还是不错的。

业务变更，会导致组件功能出现差异。起先，我们只是传入一个字段，后来我们传入了一个模型，里面有十几个字段。

我们所能做的便是，在添加新功能之前，讨论一下，判断一下。

### 晚于进度的组件

如果开发组件和开发业务是两批人，我们会就会遇到这个情况：被所需要的组件拉后腿。于是乎，我们采用的模式是：

定义一个**组件接口契约**，传入相应的参数：``<component [data]="users" [type]="'detail'"></component>``。

尽管采用装饰器模式，能在一定程度上缓解这个问题——开发人员不需要修改代码， 或者只需要少量的修改。但是呢，对于测试人员来说，他/她们需要重新测试一遍。

所以吧，最好还是同一批人，可能呢，大公司分工明确，难免会遇到这样的情况。

不过，要是遇到生产者修改了组件，却没有通知消费者，那就是个大坑了。

### 未预期的复杂组件

当我们开始一个项目的时候，按照设计文档和需求，……。评估了一下，没有出现什么复杂的功能组件：

而开发者容易遇到的一个坑是，对于某一组件如 AutoComplete，便认为它的行为和其它 AutoComponent 的组件行为是相似的，而不多加考虑。结果，随着业务的持续完善，往往就会暴雷。这个时候便需要：

1. 降低交互需求
2. 待后期完善——对，可能就是不做
3. 寻找合适的替代方案

应对这种情形，最好的方式是明确好相关的需求。

### 无法完成的风格指南

当我们选定了一个组件库，不可避免的会遇到这个问题：

1. 图标无法修改
2. 交互行为无法修改
3. 样式难以覆盖

与此同时，它会导致另外一个问题：

1. 过多的全局 Override
2. 修改成本过高

也因此，应对的方式有：

1. 在选型的时候，多加考虑相关的因素，选择适合的组件库。
2. 在设计的时候，结合组件库的需求，设计出适合于模式库的设计。

### 测试无法 100% 保障

哪怕我们有 100% 的测试覆盖率，我们也无法保障没有 bug。单元测试虽然能发现 bug，但是并不能减少 bug。

 - 加入 Snapshot 测试。
 - 加入 E2E 测试。

这些并不能真真正正地保证功能无误，尤其是 snapshot 测试，它只是看上去更加好看而已。

所以，**如何保证组件的质量**？便是我们要解决的另外一个问题。

## 小结

没有银弹，我就是吐槽一下。
