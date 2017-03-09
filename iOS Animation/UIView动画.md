#UIView动画
>感谢Sindri的小巢的[博客](http://www.jianshu.com/p/6e326068edeb)对我学习动画的帮助

iOS的动画效果一直都很棒很，给人的感觉就是很炫酷很流畅，起到增强用户体验的作用。在APP开发中实现动画效果有很多种方式，对于简单的应用场景，我们可以使用UIKit提供的动画来实现。

UIView动画实质上是对Core Animation的封装，提供简洁的动画接口。

UIView动画可以设置的动画属性有:

1. 大小变化(frame)

2. 拉伸变化(bounds)

3. 中心位置(center)

4. 旋转(transform)

5. 透明度(alpha)

6. 背景颜色(backgroundColor)

7. 拉伸内容(contentStretch)

##UIView 类方法动画

先来看一个例子，动画的效果基本上可以用下面的代码来概括：
```
self.userName.center.x += offset;    //userName进入
self.password.center.x += offset;    //password进入
```
所以可以写出以下的代码
初始化
```
- (UITextField *)userName {
    if (!_userName) {
        _userName = [[UITextField alloc] initWithFrame:CGRectMake(100, 100, 100, 30)];
        _userName.backgroundColor = [UIColor redColor];
        
        CGPoint accountCenter = self.userName.center;
        accountCenter.x -= 200;
        self.userName.center = accountCenter;
    }
    return _userName;
}

- (UITextField *)passWord {
    if (!_passWord) {
        _passWord = [[UITextField alloc] initWithFrame:CGRectMake(100, 200, 100, 30)];
        _passWord.backgroundColor = [UIColor redColor];
        
        CGPoint pswCenter = self.passWord.center;
        pswCenter.x -= 200;
        self.passWord.center = pswCenter;
    }
    return _passWord;
}
```
动画
```
    [UIView animateWithDuration:0.5 delay:1 options:UIViewAnimationOptionCurveEaseInOut animations:^{
        self.userName.center = accountCenter;
    } completion:^(BOOL finished) {
        [UIView animateWithDuration:0.5 delay:0.5 options:UIViewAnimationOptionCurveEaseInOut animations:^{
            self.passWord.center = pswCenter;
        } completion:^(BOOL finished) {
            
        }];
    }];
```
之所以嵌套一个`animateWithDuration`，是因为这样可以使得`userName`和`passWord`两个分前后进入视野。

在UIKit中，系统提供了animate标题打头的属于UIView的类方法让我们可以轻松的制作动画效果，每一个这样的类方法提供了名为animations的block代码块，这些代码会在方法调用后立刻或者延迟一段时间以动画的方式执行。此外，所有这些API的第一个参数都是用来设置动画时长的。

几个参数详细说明一下
- duration：动画时长
- delay：决定了动画在延迟多久之后执行
- options：用来决定动画的展示方式
- animations：转化成动画表示的代码
- completion：动画结束后执行的代码块

##动画参数
上面我们使用到的动画方法中有一个重要的参数options，它能让你高度的自定义动画效果。下面展示这个参数类型的值集合，你可以通过结合不同的参数来实现自己的动画：
- Repeating
```
 UIViewAnimationOptionRepeat       //动画循环执行
 UIViewAnimationOptionAutoreverse  //动画在执行完毕后会反方向再执行一次
```
```
    [UIView animateWithDuration:0.5 delay:1 options:UIViewAnimationOptionCurveEaseInOut animations:^{
        self.userName.center = accountCenter;
    } completion:^(BOOL finished) {
        [UIView animateWithDuration:0.5 delay:0.5 options:UIViewAnimationOptionAutoreverse | UIViewAnimationOptionRepeat  animations:^{
            self.passWord.center = pswCenter;
        } completion:^(BOOL finished) {
            
        }];
    }];
```
我们可以看到密码框在不断的循环进入屏幕，反方向退出屏幕这个操作，并且登录按钮也始终没有渐变出现。由此可以知道`UIViewAnimationOptionRepeat`参数不仅是让动画循环播放，并且还导致了`completion`的回调永远无法执行。
- Easing
我们都知道，一个好的动画应该更符合我们认知的规则。比如，任何事物都不能突然间的开始移动和停下，像车辆启动和停止都有一个加速和减速的过程。

为了让动画更具符合我们的认知，系统同样提供了类似的效果的参数给我们使用.
```
  UIViewAnimationOptionCurveEaseInOut   //先加速后减速，默认
  UIViewAnimationOptionCurveEaseIn      //由慢到快
  UIViewAnimationOptionCurveEaseOut     //由快到慢
  UIViewAnimationOptionCurveLinear      //匀速
```
- Transitioning
除了上面提到的这些效果，在视图、图片切换的时候，我们还能通过传入下面的这些参数来实现一些特殊的动画效果。
```
  UIViewAnimationOptionTransitionNone            //没有效果，默认
  UIViewAnimationOptionTransitionFlipFromLeft    //从左翻转效果
  UIViewAnimationOptionTransitionFlipFromRight   //从右翻转效果
  UIViewAnimationOptionTransitionCurlUp          //从上往下翻页
  UIViewAnimationOptionTransitionCurlDown        //从下往上翻页
  UIViewAnimationOptionTransitionCrossDissolve   //旧视图溶解过渡到下一个视图
  UIViewAnimationOptionTransitionFlipFromTop     //从上翻转效果
  UIViewAnimationOptionTransitionFlipFromBottom  //从上翻转效果
```
##弹簧动画
通过组合不同的options参数你可以制作真实的动画。但是，我们总是能做的更多，比如一个弹簧被用力压扁，当松开手的时候会反复弹动。使用上面的方式纵然可以实现这样的动画，但代码量复杂，也基本无复用性可言，可想而知会是糟糕的代码。因此，我们需要其他的动画方式，系统也正好提供了这样的一种动画供我们使用：
```
+ (void)animateWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay usingSpringWithDamping:(CGFloat)dampingRatio initialSpringVelocity:(CGFloat)velocity options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^ __nullable)(BOOL finished))completion
```
- dampingRatio：速度衰减比例。取值范围0 ~ 1，值越低震动越强
- velocity：初始化速度，值越高则物品的速度越快
```
    [UIView animateWithDuration:0.5 delay:1 usingSpringWithDamping:0.6 initialSpringVelocity:0 options:UIViewAnimationOptionCurveEaseOut animations:^{
        self.verify.center = verifyCenter;
    } completion:^(BOOL finished) {
        
    }];
```
