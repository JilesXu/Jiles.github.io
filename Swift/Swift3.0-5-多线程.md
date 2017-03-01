#多线程
- NSThread
- GCD
- NSOperation & NSOperationQueue
##NSThread
NSThread是苹果封装过的面向对象的多线程方案，但是仍然需要我们自己管理线程的生命周期。
###创建并启动
例子中还为当前线程命名方便使用`Thread.current`查看当前所在线程
```
  let thread = Thread.init(target: self, selector: #selector(run), object: nil)
  thread.name = "thread1"
  thread.start()
```
###创建并自动启动
```
  Thread.detachNewThreadSelector(#selector(run), toTarget: self, with: nil)
```
##GCD
**Grand Central Dispatch**它是苹果为多核的并行运算提出的解决方案，所以会自动合理地利用更多的CPU内核（比如双核、四核），最重要的是它会自动管理线程的生命周期（创建线程、调度任务、销毁线程），完全不需要我们管理，我们只需要告诉干什么就行。由于使用了Block即闭包，使得使用起来更加方便，而且灵活。所以基本上大家都使用GCD这套方案，老少咸宜，实在是居家旅行、杀人灭口，必备良药。
###任务和队列
在GCD中有两个很重要的概念**任务**和**队列**
- 任务：就是我想要完成的功能，我想要做的事情。在GCD中就是一个Block，所以添加任务十分方便。任务有两种执行方式：**同步执行**和**异步执行**，他们之间的区别是**是否会创建新的线程**。
