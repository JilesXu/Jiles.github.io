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
- 任务：就是我想要完成的功能，我想要做的事情。在GCD中就是一个Block，所以添加任务十分方便。任务有两种执行方式：**同步执行**和**异步执行**，他们之间的区别是**是否会阻塞当前线程，直到 Block 中的任务执行完毕**。

**同步执行（sync）**：它会阻塞当前线程并等待Block中的任务执行完毕，然后当前线程才会继续往下运行。

**异步执行（async）**：当前线程会直接往下执行，它不会阻塞当前线程。

- 队列：用于存放任务。一共有两种队列，**串行队列**和**并行队列**。
**串行队列的任务**：GCD会FIFO（先进先出）地取出来一个，执行一个，然后取下一个，这样一个一个的执行。

**并行队列的任务**：GCD会FIFO的取出来，但不同的是，它取出来一个就会放到别的线程，然后再取出来一个又放到另一个的线程。这样由于取的动作很快，忽略不计，看起来，所有的任务都是一起执行的。不过需要注意，GCD会根据系统资源控制并行的数量，所以如果任务很多，它并不会让所有任务同时执行。

 | 同步执行 | 异步执行
------------ | ------------ | -------------
串行队列 | 当前线程，一个一个执行 | 其他线程，一个一个执行
并行队列 | 当前线程，一个一个执行 | 开很多线程，一起执行

###创建队列
- 主队列：这是一个特殊的**串行队列**。
它用于刷新UI，任何需要刷新UI的工作都要在主队列执行，所以一般耗时的任务都要放到别的线程执行。
```
let queue = DispatchQueue.main
```
- 全局队列
在iOS < 8之前我们有四种优先级的全局队列，他们分别是：
```
#define DISPATCH_QUEUE_PRIORITY_HIGH         2  
#define DISPATCH_QUEUE_PRIORITY_DEFAULT      0  
#define DISPATCH_QUEUE_PRIORITY_LOW          (-2)  
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND   INT16_MIN
```
在iOS >= 8 之后，优先级的概念被苹果使用QoS替代了，Swift3中也一样。我们不再使用优先级，而是使用QoS来描述全局队列。简单地说，这两者之间的对应关系是这样的：

Priority(优先级) | DispatchQoS
------------ | ------------
DISPATCH_QUEUE_PRIORITY_HIGH | .userInitiated
DISPATCH_QUEUE_PRIORITY_DEFAULT | .default
DISPATCH_QUEUE_PRIORITY_LOW | .utility
DISPATCH_QUEUE_PRIORITY_BACKGROUND | .background

在 Swift 3 中，获取全局队列需要使用这个方法：
```
let queueGlobal = DispatchQueue.global(qos: .userInitiated)
```
我们将 QoS 传入 global() 方法，实际上就像指定它的优先级。当然也可以不指定，默认就是default。
```
let queueGlobal = DispatchQueue.global() //即DispatchQueue.global(qos: .default)
```
- 自己创建队列：既可以是串型队列也可以是并行队列
串型队列直接使用`init`函数
```
let queue1 = DispatchQueue.init(label: "queue1"）
```
`init`函数中还可以添加其他的参数，如果想要创建并行队列，就可以在`attributes`中添加`.concurrent`
```
let queue1 = DispatchQueue.init(label: "queue1", qos: .default, attributes:.concurrent)
```
###任务
在Swift3中指派任务只需要在所指定的队列后使用相应的方法（.sync、.async），然后使用闭包传入任务即可。
- 同步任务:会阻塞当前线程 (SYNC)
```
let queue2 = DispatchQueue.init(label: "queue2")
queue2.sync {
    print("Hello World")
}
print("Hello")        
```
打印：
Hello World
Hello

- 异步任务：不会阻塞当前线程 (ASYNC)
```        
let queue3 = DispatchQueue(label: "queue3")
queue3.async {
    print("Hello World")
}
print("Hello")
```
打印：
Hello
Hello World

**为了更好的理解同步和异步，和各种队列的使用，下面看两个示例**：

- 示例一：
```
        print("之前**********/\(Thread.current)")
        DispatchQueue.main.sync {
            print("sync**********/\(Thread.current)")
        }
        print("之后**********/\(Thread.current)")
```
以上代码在主线程调用，结果是什么？

**结果**：只会打印第一句：`之前**********/<NSThread: 0x600000075740>{number = 1, name = main}`，然后主线程就卡死了，在界面上放一个按钮，就会发现点不了了。

**分析**：
同步任务会阻塞当前线程，然后把闭包中的任务放到指定的队列中执行，只有等到闭包中的任务完成后才会让当前线程继续往下运行。
那么这里的步骤就是：打印完第一句后，`.sync`立即阻塞当前的主线程，然后把闭包中的任务放到`.main`的队列中，可是`.main`中的任务会被取出来放到主线程中执行，但主线程这个时候已经被阻塞了，所以闭包中的任务就不能完成，它不完成，`.sync`就会一直阻塞主线程，这就是死锁现象，导致主线程一直卡死。

简而言之：`.sync`阻塞主线程->闭包中的任务放入队列`.main`->队列`.main`放入主线程运行->主线程已阻塞等待闭包中的任务完成->死循环

- 示例二
```
        let queue4 = DispatchQueue.init(label: "queue4")
        print("之前**********/\(Thread.current)")
        queue4.async {
            print("sync之前**********/\(Thread.current)")
            queue4.sync {
                print("sync**********/\(Thread.current)")
            }
            print("sync之后**********/\(Thread.current)")
        }
        print("之后**********/\(Thread.current)")
```
**结果**：
```
之前**********/<NSThread: 0x60800007c500>{number = 1, name = main}
之后**********/<NSThread: 0x60800007c500>{number = 1, name = main}
sync之前**********/<NSThread: 0x600000277400>{number = 3, name = (null)}

```
很明显`sync`和`sync之后`没有被打印出来，这是为什么呢？

**分析**
1. `let queue4 = DispatchQueue.init(label: "queue4")`是一个串型队列
2. 打印`之前**********/<NSThread: 0x60800007c500>{number = 1, name = main}`
3. `queue4.async`所以当前线程不会被阻塞，于是有了两条线程，一条当前线程A继续往下打印`之后**********/<NSThread: 0x60800007c500>{number = 1, name = main}`, 另一条线程B执行闭包中的内容打印`sync之前**********/<NSThread: 0x600000277400>{number = 3, name = (null)}`。
4. 接着`.sync`同步执行，于是它所在的线程B会被阻塞，一直等到`sync`里的任务执行完才会继续往下。于是`sync`就高兴的把自己闭包中的任务放到`queue4`中，可谁想`queue4`是一个串行队列，一次执行一个任务，所以`.sync`的闭包必须等到前一个任务执行完毕，可是`queue4`正在执行的任务就是被`.sync`阻塞了的那个。于是又发生了死锁。所以`.sync`所在的线程被卡死了。剩下的两句代码自然不会打印。

4. 接着`.sync`同步执行，于是它所在的线程B会被阻塞，一直等到`sync`里的任务执行完才会继续往下。于是`sync`就把自己闭包中的任务放到`queue4`中，可`queue4`是一个串行队列，一次执行一个任务，所以`.sync`的闭包必须等到前一个任务执行完毕，可是`queue4`正在执行的任务就是被`.sync`阻塞了的那个。于是又发生了死锁。所以`.sync`所在的线程被卡死了。剩下的两句代码自然不会打印。
```
        let queue4 = DispatchQueue.init(label: "queue4", attributes: .concurrent)
        print("之前**********/\(Thread.current)")
        queue4.async {
            print("sync之前**********/\(Thread.current)")
            queue4.sync {
                print("sync**********/\(Thread.current)")
            }
            print("sync之后**********/\(Thread.current)")
        }
        print("之后**********/\(Thread.current)")
```
如上将`queue4`改成并行队列后，结果如下：
```
之前**********/<NSThread: 0x608000077840>{number = 1, name = main}
sync之前**********/<NSThread: 0x60800026db80>{number = 3, name = (null)}
之后**********/<NSThread: 0x608000077840>{number = 1, name = main}
sync**********/<NSThread: 0x60800026db80>{number = 3, name = (null)}
sync之后**********/<NSThread: 0x60800026db80>{number = 3, name = (null)}
```
###延时执行
之前在GCD中，想要指派一个任务延时执行，需要写的代码十分复杂。在swift3中很简洁。
```
        let delay = DispatchTime.now() + 3
    
        DispatchQueue.main.asyncAfter(deadline: delay) {
            print(Thread.current)
            print("hello world")
        }
```
下面是常用的GCD模板在Swift 3中的写法，仅供参考。
```
全局队列异步
DispatchQueue.global().async {
    //想做的事
    DispatchQueue.main.async {
    //返回主线程
    }
}
```
```
延时操作，注意这里的单位是秒
DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 3.5) {
    // 你想做啥
}
```
```
创建队列同步
let queue = DispatchQueue(label: "queue")
queue.sync {
    print("Hello World")
}
```
