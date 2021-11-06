# Angular Router Reuse 的那些坑

RouteReuseStrategy（路由复用策略）是 Angular 框架提供的一种路由复用机制。它可以在用户切换页面的时候，暂存路由的状态，在返回路由的时候，恢复这些快照。

**使用场景**

我们在做一个后台管理系统，它使用的是 Tab 页 + 无限滚动的方式实现。一旦我们从这个列表页进入详情页，再返回列表页时。不仅会回到列表的最上面，而且还会重置 Tab 的状态。为此，我们的业务需求是：记错这个页面的状态。

于是乎，我使用了之前的：《[适用于 Angular Lazyload 下的 RouteReuseStrategy](https://www.phodal.com/blog/angular-2-lazyload-reuse-stragery/)》，然后出了一堆的 bug。

### 1. queryParams

我遇到的第一个坑是 queryParams 失效了——对于跳转过来的 URL 来说，原先的代码是：

```typescript
this.route.queryParams.subscribe(params => {
    this.param1 = params['param1'];
    this.param2 = params['param2'];
});
```

后来，临时改成了通过 ``queryString`` 来解决 URL 上的参数。

### 2. 全局 cache 问题

先前配置的 ``shouldAttach`` 是通过判断缓存中，是否存在对应的路由，即：

```typescript
  /** 若 path 在缓存中有的都认为允许还原路由 */
  public shouldAttach(route: ActivatedRouteSnapshot): boolean {
    if (!!route.data.reusePath && !!SimpleReuseStrategy.handlers[route.data.reusePath]) {
      return true;
    }

    return false;
  }
```

这样做导致了一个问题，不管从哪个页面过来，它会被 cahce。

**解决方式 1**：在组件中判断上一页是否是详情页，如果是则使用缓存，如果不是则重新加载数据。这是我在上上一个项目中采用的方式，但是这种方式太麻烦了，需要在每个组件中写上对应的逻辑。

**解决方式 2**：配置需要缓存的页面，结合 reusePath 使用。

相关的步骤如下：

1. 标记详情页。在对应的 componet 中添加一个全局可访问的静态变量，诸如于 ``componetName = 'BlogDetailComponent'``.
2. 在 ``shouldAttach`` 的时候，先判断是否是 reuse，再判断上一个页面是否是需要缓存的页面。

这样一来，我们就可以将主要的逻辑放置在 ``SimpleReuseStrategy``，而非每个 component 中。

```typesciprt
  public shouldAttach(route: ActivatedRouteSnapshot): boolean {
    let cachablePage = false;

    ...遍历 this.cachedPages，判断是否等于 lastPageName

    if (!!route.data.reusePath && !!SimpleReuseStrategy.handlers[route.data.reusePath] && cachablePage) {
      return true;
    }

    this.lastPageName = route.routeConfig.component('componetName');
    return false;
  }
```

于是，又遇到新的坑。

### 3. 元素不销毁

采用 RouteReuseStrategy 时，页面只是被缓存，而没有被销毁。因为 Component 对就的 ngDestory 方法不会被销毁。

我们遇到的第一个坑是：无限滚动的 ``infiniteScroll`` 不会被 desotry。于是当我们在详情页的时候，仍然会触发 ``scrolled`` 方法，而解决的办法也很简单。在我们二次封装的组件里，在 ``ngOnInit`` 时，监听路由是否变化，然后调用 ``infiniteScroll`` 的 ``destoryScroller()``。

```typescript
this.router.events.subscribe((event) => {
  if (event instanceof NavigationStart && this.scrollEl) {
	this.scrollEl.destoryScroller();
  }
})
```

除此，我们还有其它的组件也有同样的问题，解决方式是相似的。

### 其它方式

```typescript
public static deleteRouteSnapshot(name: string): void {
    if (AppRouteReuseStrategy.handlers[name]) {
        delete SimpleReuseStrategy.handlers[name];
    } else {
        SimpleReuseStrategy.waitDelete = name;
    }
}
```
