---
layout: post
title: 2019 iOS面试题集合(一)
categories: 面试题
description: iOS面试题集合(一)
keywords: iOS, 面试题
---


2019 iOS面试题集合(一)

#### 1. **iOS有哪几种锁？比较各种锁的优缺点？并给出实例场景判断用哪种锁**
推荐博客--[深入理解 iOS 开发中的锁](https://bestswifter.com/ios-lock/#osspinlock)
#### 2. **内核态和用户态？写的代码在哪上面？**
**内核态**：CPU可以访问内存的所有数据，包括外围设备，例如硬盘，网卡，CPU也可以将自己从一个程序切换到另一个程序。
**用户态**：只能受限的访问内存，且不允许访问外围设备，占用CPU的能力被剥夺，CPU资源可以被其他程序获取。

**为什么要有用户态和内核态？**

由于需要限制不同的程序之间的访问能力, 防止他们获取别的程序的内存数据, 或者获取外围设备的数据, 并发送到网络, CPU划分出两个权限等级 -- 用户态和内核态。

**用户态与内核态的切换**

所有用户程序都是运行在用户态的, 但是有时候程序确实需要做一些内核态的事情, 例如从硬盘读取数据, 或者从键盘获取输入等. 而唯一可以做这些事情的就是操作系统, 所以此时程序就需要先操作系统请求以程序的名义来执行这些操作.

这时需要一个这样的机制: 用户态程序切换到内核态, 但是不能控制在内核态中执行的指令

这种机制叫**系统调用**, 在CPU中的实现称之为陷阱指令(`Trap Instruction`)

他们的工作流程如下:

1. 用户态程序将一些数据值放在寄存器中, 或者使用参数创建一个堆栈(`stack frame`), 以此表明需要操作系统提供的服务.
2. 用户态程序执行陷阱指令
3. CPU切换到内核态, 并跳到位于内存指定位置的指令, 这些指令是操作系统的一部分, 他们具有内存保护, 不可被用户态程序访问
4. 这些指令称之为陷阱(trap)或者系统调用处理器(`system call handler`). 他们会读取程序放入内存的数据参数, 并执行程序请求的服务
5. 系统调用完成后, 操作系统会重置CPU为用户态并返回系统调用的结果

当一个任务（进程）执行系统调用而陷入内核代码中执行时，我们就称进程处于内核运行态（或简称为内核态）。此时处理器处于特权级最高的（0级）内核代码中执行。当进程处于内核态时，执行的内核代码会使用当前进程的内核栈。每个进程都有自己的内核栈。当进程在执行用户自己的代码时，则称其处于用户运行态（用户态）。即此时处理器在特权级最低的（3级）用户代码中运行。当正在执行用户程序而突然被中断程序中断时，此时用户程序也可以象征性地称为处于进程的内核态。因为中断处理程序将使用当前进程的内核栈。这与处于内核态的进程的状态有些类似。 

内核态与用户态是操作系统的两种运行级别,跟intel cpu没有必然的联系, intel cpu提供Ring0-Ring3三种级别的运行模式，Ring0级别最高，Ring3最低。Linux使用了Ring3级别运行用户态，Ring0作为 内核态，没有使用Ring1和Ring2。Ring3状态不能访问Ring0的地址空间，包括代码和数据。Linux进程的4GB地址空间，3G-4G部 分大家是共享的，是内核态的地址空间，这里存放在整个内核的代码和所有的内核模块，以及内核所维护的数据。用户运行一个程序，该程序所创建的进程开始是运 行在用户态的，如果要执行文件操作，网络数据发送等操作，必须通过write，send等系统调用，这些系统调用会调用内核中的代码来完成操作，这时，必 须切换到Ring0，然后进入3GB-4GB中的内核地址空间去执行这些代码完成操作，完成后，切换回Ring3，回到用户态。这样，用户态的程序就不能 随意操作内核地址空间，具有一定的安全保护作用。
至于说保护模式，是说通过内存页表操作等机制，保证进程间的地址空间不会互相冲突，一个进程的操作不会修改另一个进程的地址空间中的数据。

**特权级**

熟悉Unix/Linux系统的人都知道，fork的工作实际上是以系统调用的方式完成相应功能的，具体的工作是由sys_fork负责实施。其实无论是不是Unix或者Linux，对于任何操作系统来说，创建一个新的进程都是属于核心功能，因为它要做很多底层细致地工作，消耗系统的物理资源，比如分配物理内存，从父进程拷贝相关信息，拷贝设置页目录页表等等，这些显然不能随便让哪个程序就能去做，于是就自然引出特权级别的概念，显然，最关键性的权力必须由高特权级的程序来执行，这样才可以做到集中管理，减少有限资源的访问和使用冲突。

特权级显然是非常有效的管理和控制程序执行的手段，因此在硬件上对特权级做了很多支持，就Intel x86架构的CPU来说一共有0~3四个特权级，0级最高，3级最低，硬件上在执行每条指令时都会对指令所具有的特权级做相应的检查，相关的概念有CPL、DPL和RPL，这里不再过多阐述。硬件已经提供了一套特权级使用的相关机制，软件自然就是好好利用的问题，这属于操作系统要做的事情，对于Unix/Linux来说，只使用了0级特权级和3级特权级。也就是说在Unix/Linux系统中，一条工作在0级特权级的指令具有了CPU能提供的最高权力，而一条工作在3级特权级的指令具有CPU提供的最低或者说最基本权力。

**用户态和内核态的转换**

- 用户态切换到内核态的3种方式

1. 系统调用
这是用户态进程主动要求切换到内核态的一种方式，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作，比如前例中fork()实际上就是执行了一个创建新进程的系统调用。而系统调用的机制其核心还是使用了操作系统为用户特别开放的一个中断来实现，例如Linux的int 80h中断。

2. 异常
当CPU在执行运行在用户态下的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中，也就转到了内核态，比如缺页异常。
3. 外围设备的中断
当外围设备完成用户请求的操作后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条即将要执行的指令转而去执行与中断信号对应的处理程序，如果先前执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了由用户态到内核态的切换。比如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。

这3种方式是系统在运行时由用户态转到内核态的最主要方式，其中系统调用可以认为是用户进程主动发起的，异常和外围设备中断则是被动的。

- 具体的切换操作
从触发方式上看，可以认为存在前述3种不同的类型，但是从最终实际完成由用户态到内核态的切换操作上来说，涉及的关键步骤是完全一致的，没有任何区别，都相当于执行了一个中断响应的过程，因为系统调用实际上最终是中断机制实现的，而异常和中断的处理机制基本上也是一致的，关于它们的具体区别这里不再赘述。关于中断处理机制的细节和步骤这里也不做过多分析，涉及到由用户态切换到内核态的步骤主要包括：

1. 从当前进程的描述符中提取其内核栈的ss0及esp0信息。

2. 使用ss0和esp0指向的内核栈将当前进程的cs,eip,eflags,ss,esp信息保存起来，这个过程也完成了由用户栈到内核栈的切换过程，同时保存了被暂停执行的程序的下一条指令。

3. 将先前由中断向量检索得到的中断处理程序的cs,eip信息装入相应的寄存器，开始执行中断处理程序，这时就转到了内核态的程序执行了。

#### 3.**内存管理机制**
推荐唐巧博客--[理解iOS的内存管理](https://blog.devtang.com/2016/07/30/ios-memory-management/)
#### 4.**block有几种？为什么__block修饰的值在内部可以改变？**
**block有三种类型：**
- 全局块(_NSConcreteGlobalBlock)--不使用外部变量的block是全局block
比如：
```
NSLog(@"%@",[^{
NSLog(@"blobalBlock");
} class]);
```
输出：
```
__NSGlobalBlock__
```
- 栈块(_NSConcreteStackBlock)--使用外部变量并且未进行copy操作的block是栈block
比如：
```
NSInteger num = 10;
NSLog(@"%@",[^{
NSLog(@"stackBlock:%zd",num);
} class]);
```
输出：
```
__NSStackBlock__
```
日常开发常用于这种情况：
```
[self testWithBlock:^{
NSLog(@"%@",self);
}];
```
```
- (void)testWithBlock:(dispatch_block_t)block{
block();
NSLog(@"%@",[block class]);
}
```
- 堆块(_NSConcreteMallocBlock)--对栈block进行copy操作，就是堆block，而对全局block进行copy，仍是全局block

```
void(^globalBlock)(void) = ^{
NSLog(@"globalBlock");
};
NSLog(@"%@",[globalBlock class]);
```
输出
```
__NSGlobalBlock__
```

**三种block在内存中的体现**
* 全局块存在于全局内存中, 相当于单例.
* 栈块存在于栈内存中, 超出其作用域则马上被销毁
* 堆块存在于堆内存中, 是一个带引用计数的对象, 需要自行管理其内存

#### 5.**copy与strong修饰符**
copy修饰的NSString,在初始化时,如果来源是NSMutableString的话,会对来源进行一次深拷贝,将来源的内存地址复制一份,这样,两个对象就一点关系就没有了,无论你怎么操作来源,都不会对自己的NSString有任何影响。

比如:
你有一个@property(nonatomic,copy) NSString *str;
然后有一个NSMutableString *sourceStr;
当你进行str = sourceStr操作之后,紧接着你又改变了sourceStr的内容sourceStr = @"change";那么str的内容并不会改变. 如果你的str不是copy修饰的,而是strong修饰的,那么str的值也会变成@"change";因为strong是浅拷贝的,并不会对来源的内存地址进行拷贝

**那么问题来了,既然copy安全,那为什么不都用copy?**

这里我们需要了解一点,copy修饰的NSString在进行set操作时,底层是这样实现的:
我们还是举上面那个例子,进行str = sourceStr操作时,内部会执行一个操作:
str = [sourceStr copy];
那么这个copy里面做了什么呢?
if ([str isMemberOfClass:[str class]])
没错,就是进行一次判断,判断来源是可变的还是不可变的,如果是不可变,那么好,接下来的操作就跟strong修饰的没有区别,进行浅拷贝;如果是可变的,那么会进行一次深拷贝。
所以,copy操作内部会进行判断,你别小看了这个if操作所消耗的内存,一次不重要,十次可能也可以忽略不计,但当你的项目十分庞大时,有成百上千个个NSString对象,多多少少会对你的app的性能造成一定的影响.

**那么回到最初的问题,什么时候用copy,什么时候用strong**
你只需要记住一点,当你给你的的NSString对象赋值时,如果来源是NSMutableString,那么这种情况就必须要用copy;如果你确定来源是不可变类型的,比如@"abc_url"这种固定的字符串,那么用strong比较好。
#### 6.**多线程有哪几种？GCD和NSOperation有什么区别？信号量了解过吗，dispatch_once的原理？**
- **多线程的原理**
同一时间，CPU只能处理一条线程，只有一条线程在工作(执行)，多线程并发(同时)执行，其实是CPU快速的在多条线程之间调度(切换),如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象。思考：如果线程足够多，CPU会在N多条线程之间调度，CPU会累死，消耗大量的CPU资源每条线程被调度的频率会降低(线程的执行效率降低)。
- **多线程的优点**
能适当提高程序的执行效率；
能适当提高资源利用率(CPU、内存利用率)
- **多线程的缺点**
线程需要占用一定的内存空间(默认情况下主线程占用1M子线程占用512k)，如果开启大量的线程，会占用大量的内存空间，降低程序的性能，线程越多CPU在调度线程上的开销就越大程序设计更加复杂：比如线程之间的通信、多线程的数据共享。

- **多线程的分类**
**pthread**
1、一套通用的多线程API
2、适用于Unix/Linux/Windows等系统
3、跨平台、可移植
4、使用难度大
5、使用语言：C语言
6、使用频率：几乎不适用
7、线程的生命周期：由开发者自己进行管理
**NSThread**
1、面向对象
2、简单易用，可直接操作线程
3、使用语言:OC语言
4、使用频率:偶尔使用
5、线程的生命周期:由开发者自己管理
**GCD**
1、替换NSThread等线程技术
2、充分利用了设备多核(自动)
3、使用语言:C语言
4、使用频率:经常使用
5、线程的生命周期:自动管理
**NSOperation**
1、基于GCD(底层是GCD)
2、比GCD多了一些简单实用的功能
3、使用更加面向对象
4、使用语言:OC语言
5、线程的生命周期:自动管理
**GCD优点:**
GCD是一个轻量级的数据结构，以底层实现隐藏的神奇技术，我们可以通过GCD和block轻松实现多线程编程，有时候，GCD相比其他系统提供的多线程方法更加有效。
- **dispatch_semaphore保持线程同步**
```
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
```

```
__block int j = 0;
dispatch_async(queue, ^{
j = 100;
dispatch_semaphore_signal(semaphore);
});

dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"finish j = %zd", j);
```
结果输出 j ＝ 100；
如果注掉dispatch_semaphore_wait这一行，则 j ＝ 0；
>**注释：** 由于是将block异步添加到一个并行队列里面，所以程序在主线程跃过block直接到dispatch_semaphore_wait这一行，因为semaphore信号量为0，时间值为DISPATCH_TIME_FOREVER，所以当前线程会一直阻塞，直到block在子线程执行到dispatch_semaphore_signal，使信号量+1，此时semaphore信号量为1了，所以程序继续往下执行。这就保证了线程间同步了。

- **dispatch_semaphore为线程加锁**
```
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);

for (int i = 0; i < 100; i++) {
dispatch_async(queue, ^{
// 相当于加锁
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"i = %zd semaphore = %@", i, semaphore);
// 相当于解锁
dispatch_semaphore_signal(semaphore);
});
}
```
> **注释：** 当线程1执行到dispatch_semaphore_wait这一行时，semaphore的信号量为1，所以使信号量-1变为0，并且线程1继续往下执行；如果当在线程1NSLog这一行代码还没执行完的时候，又有线程2来访问，执行dispatch_semaphore_wait时由于此时信号量为0，且时间为DISPATCH_TIME_FOREVER,所以会一直阻塞线程2（此时线程2处于等待状态），直到线程1执行完NSLog并执行完dispatch_semaphore_signal使信号量为1后，线程2才能解除阻塞继续住下执行。以上可以保证同时只有一个线程执行NSLog这一行代码。

#### 8.**runLoop机制？source0是什么？source1是什么？追问 事件响应时怎么通知runLoop的**
RunLoop是一个do-while 循环，又不是一个do-while 循环。他的工作模式是一个循环，但是他基于mach_port和mach_msg的 休眠\唤醒 机制确保了他可以在无任务的时候休眠，有任务的时候及时唤醒，相比于一个普通循环，不会空转，不会浪费系统资源。RunLoop又通过不同的工作mode隔离了不同的事件源，使他们的工作互不影响。这才是RunLoop实现省电，流畅，响应速度快，用户体验好的根本原因；进而基于RunLoop的组件如计时器、GCD、界面更新、自动释放池能高效运转的根本原因。
* **为什么引入Runloop机制，有什么作用或者好处？**
引入Runloop机制的目的是利用RunLoop机制的特点实现整体省电的效果，并且让系统和应用可以流畅的运行，提高响应速度，达到极致的用户体验。
* **为什么省电？**
主要有两点：一、因为不做任何操作的时候主线程Runloop会处于退出状态，不会执行任何空转逻辑，不执行代码自然不消耗CPU资源，自然省电。二、Runloop提供一种班车机制，限制如页面刷新等任务的执行频率，一次Runloop只执行一次，防止多次重复执行代码带来的性能损耗。

* **为什么可以流程运行？**
一个app流畅与否的决定性因素是主线程的阻塞率，在iOS系统中runloop每秒执行60次，理论上主线程runloop达到55帧以上的刷新频率用户就感觉不到卡顿。
Mode机制，同一时间只执行一个Mode内的Source或者Timer，比如拖动的时候只指定拖动Mode，其他Mode 如Default Mode中的源不会被执行，确保了高速滑动的时候不会有其他逻辑阻碍主线程刷新。
Runloop做的是管理视图刷新频率，防止重复运算。由于视图更新必须在主线程，视图的重布局和重绘都会占用主线程的执行时间，一次Runloop循环只执行一次可以最大限度的防止重复运算导致的计算浪费。
管理核心动画。核心动画有三个树，其中render tree 是私有的，应用开发无法访问到。render tree在专用的render server 进程中执行，是真正用来渲染动画的地方，线程优先级高于主线程。所以即使app主线程阻塞，也不会影响到动画的绘制工作。既节省了主线程的计算资源，又使动画可以流畅的执行。
支持异步方法调用，将耗时操作分发到子线程中进行。RunLoop是performSelector的基础设施。我们使用 performSelector:onThread: 或者 performSelecter:afterDelay: 时，实际上系统会创建一个Timer并添加到当前线程的RunLoop中。
还有其他的点，这里不展开，详情可阅读下文应用实践。
当然Runloop不是万能的，如果代码质量差，在一次Runloop循环中执行的时间过久一样会导致卡顿，所以解决卡顿问题也是程序员能力的体现。

* **如何提高响应速度？**
当发生系统事件时，如触碰事件，系统通过Mach Port 发送 Mach消息主动唤醒Runloop。Mach是抢占式操作系统内核，Mach系统IPC机制就是依靠消息机制实现的，所以效率非常高。

#### 10.**KVO的实现原理**

经过查阅资料我们可以了解到。 NSKVONotifyin_Person中的setage方法中其实调用了 Fundation框架中C语言函数 _NSsetIntValueAndNotify，_NSsetIntValueAndNotify内部做的操作相当于，首先调用willChangeValueForKey 将要改变方法，之后调用父类的setage方法对成员变量赋值，最后调用didChangeValueForKey已经改变方法。didChangeValueForKey中会调用监听器的监听方法，最终来到监听者的observeValueForKeyPath方法中。

#### 11.**字典的原理？追问重复的key是怎么排列的？取的时候是怎么取的?**
一言以蔽之： 在OC中NSDictionary是使用hash表来实现key和value的映射和存储的。
那么问题来了什么是hash表呢？
哈希表（hash表）：又叫做散列表，是根据关键码值（key value）而直接访问的数据结构。也就是说它通过关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射叫做函数，存放记录的数组叫做哈希表。

#### 12.**layoutIfNeed 和 setNeedlayout有什么区别?**
- **setNeedsLayout**
标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，在下一轮runloop结束前刷新，对于这一轮runloop之内的所有布局和UI上的更新只会刷新一次，layoutSubviews一定会被调用。
- **layoutIfNeeded**
如果有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）。
- layoutIfNeeded不一定会调用layoutSubviews方法。
* setNeedsLayout一定会调用layoutSubviews方法（有延迟，在下一轮runloop结束前）。
* 如果想在当前runloop中立即刷新，调用顺序应该是
[self setNeedsLayout];
[self layoutIfNeeded];
setNeedsDisplay会调用自动调用drawRect方法














