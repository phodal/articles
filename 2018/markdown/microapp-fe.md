# 【架构拾集】 前端微应用化

【架构拾集】，是我用来记录项目实施过程中的优秀架构集。

微应用化即**在开发和运行时**，应用都是以单一、微小应用的形式存在。

微应用化与微前端架构相当的类似，它们在开发时都是独立应用，在构建时又可以按照需求单独加载。如果以微前端的单独开发、单独部署、运行时聚合的基本思想来看，微应用化就是微前端的一种实践。只是使用微应用化意味着：我们只能使用唯一的一种前端框架。如果从框架不限的角度来定义，怕是离微前端有些远，不过大团队怕是不会想同时支持多个前端框架。

## 适用场景

为了方便后期我的查阅，我还是简单地写个相应架构的电梯演进。

| 关键因素 | 描述 |
| --- | --- |
| 对于 | 想拆解单体前端应用的团队 |
| 我们的架构 | 微应用化 |
| 是一个 | 类微前端架构 |
| 它可以 | 在开发环境将应用拆分成一个个的模块化应用，在构建时以单体的形式构建 |
| 但他不同于 | 微前端架构 |
| 它的优势是 | 实施成本低、技术难度小、维护成本低 |

## 技术远景

**作为项目的技术负责人，我希望项目中的每个功能模块，都可以交由不同的团队独立开发。**

## 方案对比

再次的，让我们和之前的不同方案进行对比：

| 方式 | 开发成本 | 维护成本 | 可行性 | 同一框架要求 | 实现难度 | 潜在风险 |
| --- | --- | --- | --- | --- | --- | --- |
| 路由分发 | 低 | 低 | 高 | 否 | ★ | 这个方案太普通了 |
| iFrame | 低 | 低 | 高 | 否 | ★ | 这个方案太普通了 |
| 应用微服务化 | 高 | 低 | 中 | 否 | ★★★★ | 针对每个框架做定制及 Hook |
| 微件化 | 高 | 中 | 低 | 是 | ★★★★★ | 针对构建系统，如 webpack 进行 hack |
| 微应用化 | 中 | 中 | 高 | 是 | ★★★ | 统一不同应用的构建规范 |
| 纯 Web Components | 高 | 低 | 高 | 否 | ★★ | 新技术，浏览器的兼容问题 |
| 结合 Web Components | 高 | 低 | 高 | 否 | ★★ | 新技术，浏览器的兼容问题 |

**微服务化**，即每个前端应用一个独立的服务化前端应用，并配套一套统一的应用管理和启动机制，诸如微前端框架 Single-SPA 或者 [mooa](https://github.com/phodal/mooa) 。

**微件化**，即通过对构建系统的 hack，使不同的前端应用可以使用同一套依赖。它在**应用微服务化**的基本上，改进了重复加载依赖文件的问题。

## 架构设计方案

在刚结束的项目里，我们采用了这种架构方式来构建应用，我们将其称之为微应用。原因主要有两个，一个是每个应用都是以功能模块划分的，一个则是应用最后仍然是以单体应用的形式存在的。我们的方式就是在**开发环境将单体应用拆分成一个个的模块应用**，而在构建时是以单体应用的形式构建，而在运行时是以应用模块的形式存在。

状态        | 开发时        | 构建时  | 运行时
-------------|----------------|------------|-------
应用形式 | 微小应用    | 单体      | 微小模块

值得注意的是，我们能成功实施微应用化的一个关键因素是，前端框架本身是能支持**功能模块的 Lazyload**。不过，事实上支持 Lazyload 的另外一个关键因素是：webpack 对于 chunk 的使用。

由于我懒，所以我直接从 GitHub 上扒一个 [Lazyload Demo](https://github.com/toddmotto/angular-lazy-load-demo) 来作为示例。如下是一个 Lazyload 的路由示例：

```
export const ROUTES: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
  { path: 'dashboard', loadChildren: '../dashboard/dashboard.module#DashboardModule' },
  { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
  { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
];
```

其对应的目标结构如下所示：

```bash
├── app
│   ├── app.component.ts
│   └── app.module.ts
├── dashboard
│   ├── dashboard.component.ts
│   └── dashboard.module.ts
├── main.ts
├── reports
│   ├── reports.component.ts
│   └── reports.module.ts
└── settings
    ├── settings.component.ts
    └── settings.module.ts
```

上面的代码对应着对应的 module。只需要在使用的时候，Angular 构建的时候会将 module 独立构建成 `*.chunk.js`。假设现在我们有 dashboard、settings、reports 三个应用，那么现在的工程里的三个应用都是以空白 module 形式而存在的。它可以在其它 module 还未开发的时候，不影响系统的构建。那么再加上主的功能，一共会有四个代码仓库：

 - 主代码库。只包含一个空白的框架式代码，它是一个单独的应用可以独立构建，构建完是带 Lazyload 的工程。
 - dashboard、settings、reports 三个应用。它们都是各自独立的应用，在构建时复制对应模块的代码到主工程。

当系统开始构建时，我们会从独立的 dashboard 应用中拷贝相应的 module 代码及依赖，拷贝到上述的这个工程里，然后替换。而这个 dashboard 应用内，自己又是一个完整的 Angular 应用，它可以独立地开发运行。

## 持续集成设计

系统的持续集成的触发机制可以由这几部分集成：

 - 功能模块（features module) 代码更新，会触发对应的模块的持续构建
 - 主的应用代码更新，会触发整个系统的持续构建
 - 功能模块持续集成成功，触发整个系统的持续构建

如上一节中架构设计方案所述，主应用构建的工程中，我只需要复制对应的代码即可。

## 测试策略

考虑到微前端架构在实施上的一些特殊性，我们有必要在传统的测试金字塔的基础上添加一些额外的测试：
 
  - 依赖一致检测测试
  - 功能模块生成测试

### 依赖一致性测试

由于不同的功能模块，需要保持一致的依赖版本。因此有必要对依赖版本进行测试、对比，以避免在线上依赖并不一致的时候，出现一些意料之外的 Bug。

对于前端项目来说，这个依赖管理配置文件就是  `package.json` 。我们只需要从不同的项目中，读取这个文件，然后对比其中的版本即可。使得每个工程的依赖可以尽可能地保持一致。

### 功能模块生成测试

由于项目加载模块的方式，是通过前端框架自带的的 Lazyload 功能来实现的。理论上，我们就不需要测试 lazyload 的功能是否正确。如果需要的话，我们只需要以下三部分其中的**一个**：

 - 测试复制的模块能复制到对应的目录上
 - 测试生成的模块代码大小是否正常
 - E2E 测试

要对模块是否能正确复制进行测试，最简单的方式是编写脚本，在持续集成的过程中运行测试脚本，如果没有检测到则 ``exit(-1)``，持续集成构建失败。

测试模块代码大小是否合理的原因在于，我们可能没有正确的在对应的目测替换功能模块。如下是一个生成的 Lazyload 模块示例，正常情况下每个 chunk.js 文件应该是要大于空白的模块的大小：

```bash
Date: 2018-08-05T06:31:39.188Z
Hash: c1e57b16329e1ec9bb5e
Time: 44397ms
chunk {0} 0.bb599f286b4bd7a5671c.chunk.js (common) 22.7 kB  [rendered]
chunk {1} 1.0124f60f4b26e51b6eac.chunk.js () 22.9 kB  [rendered]
chunk {2} 2.563bd899f2d57f903f05.chunk.js ()  22.7kB  [rendered]
chunk {3} main.b812524b18403b7b0cc4.bundle.js (main) 1.54 MB [initial] [rendered]
chunk {4} polyfills.3b18b0b8f25d1038155d.bundle.js (polyfills) 87 kB [initial] [rendered]
chunk {5} styles.425af9d9b93b3ae95ff2.bundle.css (styles) 22 kB [initial] [rendered]
chunk {6} inline.a41bfd7c50df83afde20.bundle.js (inline) 2.54 kB [entry] [rendered]
```

但是上述这部分的测试，其依赖于在构建的时候测试日志。同样的需要在持续集成中编写脚本，并 ``exit(-1)``。

使用 E2E 测试对于微前端或者微服务化架构来说，是一种特别有效的方式。唯一的问题可能是，它运行起来比较慢。

## 结论

微应用化，又可以称之为组合式集成，即通过软件工程的方式，在开发环境对单体应用进行拆分，在构建环境将应用组合在一起构建成一个应用。
