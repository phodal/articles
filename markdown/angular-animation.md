Angular 动画
===


```
export const AddCartAnimation = [
  animate(
    1000,
    keyframes([
      style({ opacity: 0.8, transform: 'scale(0.8)', offset: 0.3 }),
      style({ opacity: 0.3, transform: 'scale(0.3)', offset: 0.5 }),
      style({ opacity: 0.2, transform: 'scale(0.2) translate(12000px, 8000px)', offset: 1 })
    ])
  )
];
```

```
const formElement = this.formElement.nativeElement;

const forkFormComponent = this.cartContainer.nativeElement;
forkFormComponent.appendChild(formElement.cloneNode(true));

this.renderer.setStyle(forkFormComponent, 'display', 'block');
this.renderer.setStyle(forkFormComponent, 'position', 'absolute');
this.renderer.setStyle(forkFormComponent, 'top', '-300px');
this.renderer.setStyle(forkFormComponent, 'left', '0');

const myAnimation = this.animationBuilder.build(AddCartAnimation);

const player = myAnimation.create(forkFormComponent);
player.play();
player.onDone(() => {
  const nativeElement = this.cartContainer.nativeElement;
  nativeElement.removeChild(nativeElement.childNodes[0]);
  this.renderer.setStyle(nativeElement, 'display', 'none');
});
```

```
trigger('countUp', [
state('in', style({ transform: 'translateY(0)' })),
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