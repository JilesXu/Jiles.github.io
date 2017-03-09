#UIView动画
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

