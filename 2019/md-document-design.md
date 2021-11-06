#【架构拾集】基于 Markdown 文档展示系统设计

> Markdown是一种轻量级标记语言，它允许人们 “使用易读易写的纯文本格式编写文档，然后转换成有效的 XHTML（或者HTML）文档”。 

Markdown 的这种特性，非常适合于作为文档编写和展示的系统。于是乎，我们便在最近的项目中，采用了 Markdown 作为标记语言，作为我们的文档部分的核心。


## 系统设计思路

在我们的系统中，采用了多种模式来展示 Markdown 相关的内容：

 - 普通 Markdown 文档。即单个 Markdown 文档，它可以读取支持目录生成，以及目录与内容之间的联动。
 - Markdown 合集。Markdown + gzip 的方式，用于支持诸如图片、视频、文档等内容。支持多个/批量的 Markdown 文件展示，其支持方式是由 zip + index.md——在 index.md 中创建目录及与对应 Markdown 文件的关系。
 - Markdown  + Iframe 展示。即在 Markdown 中嵌入特定的链接，链接用 iframe 的形式来嵌入内容

当然了，这个需求是最后的产出结果，而非一开始的设计需求。

## 实现思路

**单个 Markdown 文件**

1. 解析 Markdown 文件生成目录
2. 添加对应的点击目录支持，和文档滚动支持。

**Markdown 命令**

用户需要：

1. 使用相对目录的方式，编写 markdown，并可以放置对应的资料
2. 用户编写对应的 index.md 作为目录，并打包文件夹成 gzip 文档
3. 上传文档

代码逻辑：

1. 先判断资源的类型是否是 zip 文件。
2. 读取 zip 文件中的 index.md，修改 render 方法，根据解析的 Markdown 生成目录结构数据，并渲染目录。
3. 获取目录数据中的第一值，并渲染。
4. 当用户点击某一目录时，根据目录数据，加载对应的 Markdown 文件，并渲染。

**Markdown + Iframe**

1. 解析生成的 Markdown 文件，判断目录中是否只有一个链接 ，且这个链接是一个定制的链接，添加对应的标志位
2. 当是特定的 iframe 链接时，使用 iframe 的方式加载这个链接

## 实现

在这里我们采用的是 ``ngx-markdown``，它可以支持自定义的渲染方式。即，可以有目标性地对标记进行修改，如自定义 heading 生成的 HTML：

```typescript
  constructor(private markdownService: MarkdownService) { }

  ngOnInit() {
    this.markdownService.renderer.heading = (text: string, level: number) => {
      const escapedText = text.toLowerCase().replace(/[^\w]+/g, '-');
      return '<h' + level + '>' +
               '<a name="' + escapedText + '" class="anchor" href="#' + escapedText + '">' +
                 '<span class="header-link"></span>' +
               '</a>' + text +
             '</h' + level + '>';
    };
  }
```

### 静态资源访问

由于，下载的文档需要权限管理，也因此文档内的静态资源也需要权限。而这些资源都是通过 HTTP 直接访问的，并非使用前端中的 HTTP Interceptor，因此我们有了 Token 管理需求。为此，我们采用的方式是：在资源的链接后面加上 ``?token=xxx`` 的形式，以实现这个功能。

1. 前端在发送请求时，在 URL 中加入  ``?token=xxx`` 。
2. 后端在 gateway 转换 token，再返回给对应的服务。

### htmlparser2 解析 HTML

Markdown 本身是支持 HTML 标签，而我们需要支持视频播放，因此需要解析 HTML，并在标记上添加 token。

起先我尝试了 cheerio 来解决，但是在 Angular 中没有集成成功。我便采用了其依赖中的  htmlparser 2 来解析。最后，采用了 htmlparser 2 来解析：

```
let lastNode = null;
let html = '';
let handler = {
  onopentag: function(name, attribs) {
    if (name === 'source' && lastNode === 'video') {
      let src = attribs['src'] + '?token=233';
      html = `<video controls width="250"><source src="${src}" type="video/webm"></video>`
    }
    lastNode = name;
  }
};
let parser = new htmlparser.Parser(handler, {decodeEntities: true});
parser.write(`<video controls width="250"><source src="/media/examples/flower.webm" type="video/webm"></video>`);
parser.end();
```

### 排版

项目由于功能需要，需要特殊的排版方式：内容在左，代码在右

1. list 和 table 将 float: left，代码 float: right
2. 剩下的部分都是 block 显示 



