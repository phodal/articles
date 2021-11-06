# Angular 修改测试注入问题：Async callback was not invoked within timeout

项目中采用了 Angular 作为前端框架，并采用的其自带的 Jasmine 作为测试框架。而在过去的一段时间里，一直在遭遇下面的问题：

> Async callback was not invoked within timeout specified by jasmine.DEFAULT_TIMEOUT_INTERVAL

由于，我们有差不多 600 个测试案例，而测试一直会因为这个原因挂掉。因此，我们不得不尝试去解决这个问题。

## 不可能的超时

既然超时，那就延长时间。

但是，我们并没有找到正确的方式。

直到有一天，我意味着可能是注入的问题——因为我们使用了 usecase 代替了 service，而 service 在 Angular 中很脆弱。

## Angular 测试注入


### 1. 传统的 Angular 测试的注入方式

如下所示：

```
let service: AuthService;

beforeEach(() => {
  service = new AuthService();
});

it('should return true from isAuthenticated when there is a token', () => {
  localStorage.setItem('token', '1234');
  expect(service.isAuthenticated()).toBeTruthy();
});
```

而这样的测试适合于，没有网络请求的测试，并且需要自己实例化 service，操作起来比较麻烦。

### 2. 采用 TestBed 的注入方式

正常情况下，我们采用的是如下的方式来测试的：

```
let authService: AuthService;

beforeEach(() => {
  ...
  TestBed.configureTestingModule({
    declarations: [LoginComponent],
    providers: [AuthService]
  });

  ...
  authService = TestBed.get(AuthService);
});
```

而这种方式也是不行的，于是又找到了新的方法。

### 3. 采用 Mock Service 的方式

为此，我们采用了更简单的方式，即去 Mock 对应的 Service

```typescript
TestBed.configureTestingModule({
  providers: [{ provide: SomeService, useValue: myMock }]
});
```

但是，这样也会出现随机挂掉的问题：

### 4. OverrideComponent

于是乎，我们又转向了新的方法：

```
TestBed.overrideComponent(
  MyComponentUnderTest,
  {set: {providers: [
    {provide: MyRealService, useClass: MyMockService}
  ]}}
);
```

但是仍然不行。

### 在单个 test case 中 inject

最后，我们似乎采用了以下的方式解决问题：

```typescript
it('should pass context to withModule helper - advanced',
  withModule(moduleConfig).inject([FancyService], function(service: FancyService) {
    expect(service.value).toBe('real value');
    this.contextModified = true;
}));
```

## 结论

坑仍然很多。

