Angular 动画的两种方式及添加购物车动画
===

在前端应用中，动画是一个常见的场景。在使用了一系列的动画库之后，终于需要自己来实现一个动画了。这次的动画则是基于 Angular 框架。我的场景是一个类似于添加购物车的动画。在这个场景里，需要两个动画，一个是购物车数量的增加动画，一个则是折叠页面元素的动画。

在实现的过程上，我采用了两种不同的 Angular 动画的方式：

 - 使用 TypeScript 控制动画
 - 使用 @Component 中的 animations

Angular 动画基础
---

如 Angular 官网中的示例那样，要在 Angular 应用中添加动画是比较简单的一件事——前提是我们懂得添加的法则。如下是官网的示例：

```
@Component({
  selector: 'app-hero-list-basic',
  template: `
    <ul>
      <li *ngFor="let hero of heroes"
          [@heroState]="hero.state"
          (click)="hero.toggleState()">
        {{hero.name}}
      </li>
    </ul>
  `,
  styleUrls: ['./hero-list.component.css'],
  animations: [
    trigger('heroState', [
      state('inactive', style({
        backgroundColor: '#eee',
        transform: 'scale(1)'
      })),
      state('active',   style({
        backgroundColor: '#cfd8dc',
        transform: 'scale(1.1)'
      })),
      transition('inactive => active', animate('100ms ease-in')),
      transition('active => inactive', animate('100ms ease-out'))
    ])
  ]
})
```

要使用动画，需要在模板中使用 ``[@heroState]``语法，这里的 ``heroState`` 对应着 ``@Component`` 中的 ``heroState`` 相关的动画。

 - 在这个 ``trigger`` 中，我们定义了 ``inactive`` 和 ``active`` 两个不同的 ``state``。即当模板中的 ``hero.state`` 发生变化的时候，我们就会找到对应的 ``state`` 的样式等等的内容。
 - 在这个 ``trigger`` 中，我们还定义了两个 ``transition``，即当我们的 ``state`` 从 ``inactive => active`` 或者 ``active => inactive`` 时，我们就会执行后面的动画。

原理上，大概就是这么多了。然后，我就开始了我的动画之旅。

购物车数量增加动画
---

对于我的场景来说，要添加这个动画并不难。无非就是上一个值淡出，新的值淡入：

```
  trigger('count', [
    transition('void => current', [
      animate(
        '400ms 150ms',
        keyframes([
          style({ opacity: 0.6, transform: 'translateY(0)', offset: 0 }),
          style({ opacity: 0.3, transform: 'translateY(-15px)', offset: 0.5 }),
          style({ opacity: 0, transform: 'translateY(-30px)', offset: 1 })
        ])
      )
    ]),
    transition('void => last', [
      animate(
        250,
        keyframes([
          style({ opacity: 0, transform: 'translateY(100%)', offset: 0 }),
          style({ opacity: 0.3, transform: 'translateY(15px)', offset: 0.5 }),
          style({ opacity: 0.8, transform: 'translateY(0)', offset: 1.0 })
        ])
      )
    ])
  ])
```

代码就是这么简单，这里用到了关键帧 ``keyframes``，来进行一些简单的动画转换。

页面缩放动画
---

随后，我需要做的就是对页面的元素进行缩放等效果，这个时候就需要用到 AnimationBuilder 来实现了：

```

    const myAnimation = this.animationBuilder.build([
      animate(
        1000,
        keyframes([
          style({ opacity: 0.8, transform: 'scale(0.8)', offset: 0.3 }),
          style({ opacity: 0.3, transform: 'scale(0.3)', offset: 0.5 }),
          style({ opacity: 0.2, transform: 'scale(0.2) translate(12000px, 8000px)', offset: 1 })
        ])
      )
    ]);

    const player = myAnimation.create(forkFormComponent);
    player.play();
    player.onDone(() => {
      const nativeElement = this.cartContainer.nativeElement;
      nativeElement.removeChild(nativeElement.childNodes[0]);
      this.renderer.setStyle(nativeElement, 'display', 'none');
    });
```

在那之前，我先复制了页面元素：

```
const formElement = this.formElement.nativeElement;

const forkFormComponent = this.cartContainer.nativeElement;
forkFormComponent.appendChild(formElement.cloneNode(true));

this.renderer.setStyle(forkFormComponent, 'display', 'block');
this.renderer.setStyle(forkFormComponent, 'position', 'absolute');
this.renderer.setStyle(forkFormComponent, 'top', '-300px');
this.renderer.setStyle(forkFormComponent, 'left', '0');
```

这样一来，就能复制页面的 DOM，然后实现缩放效果了。