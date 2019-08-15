---
layout: post
title: iOS APP的生命流程
categories: Objective-C
description: iOS APP的生命流程
keywords: Objective-C, OC, iOS
---


#### iOS APP的生命流程

当自己对技术对APP的性能达到一定的追求时，就需要对APP有较深的了解，越是深入的了解和理解才能从各个点上去优化性能。深入的理解还可以使我们在对iOS进行逆向工程时更加了解从哪一个时间段什么方式入侵最合适。
我把APP的生命流程分为三大部分:
1.APP的启动流程(pre-main)
2.APP的初始化流程(main)
3.APP的运行时生命周期
APP的启动流程
* 1.iOS系统首先会加载解析该APP的Info.plist文件，因为Info.plist文件中包含了支持APP加载运行所需要的众多Key，value配置信息，例如APP的运行条件(Required device capabilities)，是否全屏，APP启动图信息等。
* 2.创建沙盒(iOS8后，每次启动APP都会生成一个新的沙盒路径)
* 3.根据Info.plist的配置检查相应权限状态
* 4.加载Mach-O文件读取dyld路径并运行dyld动态连接器(内核加载了主程序，dyld只会负责动态库的加载)
* 4.1 首先dyld会寻找合适的CPU运行环境
* 4.2 然后加载程序运行所需的依赖库和我们自己写的.h.m文件编译成的.o可执行文件，并对这些库进行链接。
* 4.3 加载所有方法(runtime就是在这个时候被初始化的)
* 4.4 加载C函数
* 4.5 加载category的扩展(此时runtime会对所有类结构进行初始化)
* 4.6 加载C++静态函数，加载OC+load
* 4.7 最后dyld返回main函数地址，main函数被调用
* 
从整个APP启动流程中，我们可以做优化的点主要有
1.是减少系统依赖库
2.减少自己需要加入的各种三方库(库越少dyld加载的速度越快，就能越早的返回程序入口main函数的地址)
3.有一些自己加入的库，能选择静态库就选择静态库，少用动态库，因为动态库的加载方式比静态库慢。如果必须依赖动态库，则把多个非系统的动态库合并成一个动态库。
4.自己加入的各种framework库根据情况设为optional和required，如果该framework在当前App支持的所有iOS系统版本中都存在，那么就设为required，否则就设为optional，因为optional会有些额外的检查会导致加载变慢。
5.将不必须在+load方法中做的事情延迟到+initialize中，尽量不要用C++虚函数(创建虚函数表有开销)。
6.减少项目文件中Category，静态变量等的使用数量
7.使用appCode检查项目中，那些类和方法没有使用到。 把没有使用到的删除
8.让UI大佬尽量把给的资源压缩到最小，因为在启动加载时会加载资源图片进行IO操作。所以图片小加载速度也会响应提升。
9.内存上优化：类和方法名不要太长：iOS每个类和方法名都在__cstring段里都存了相应的字符串值，所以类和方法名的长短也是对可执行文件大小是有影响，及影响加载速度也消耗内存；因为OC的动态特性，都是加载后通过类/方法名反射找到这个类/方法进行调用，OC的对象模型会把类/方法名字符串都保存下来(压缩算法TinyPNG)。
冷启动、热启动
如果程序刚被运行过一次，那么程序的代码会被dyld缓存起来，因此即使杀掉进程再次重启加载时间也会相对快一点，如果长时间没有启动或者当前dyld的缓存已经被其他应用占据，那么这次启动所花费的时间就要长一点，这就分别是热启动和冷启动的概念。
启动时间测试方法
在 Xcode 中 Edit scheme -> Run -> Auguments 将环境变量 DYLD_PRINT_STATISTICS设为 1。随后启动APP时控制台会输出的启动耗时内容。
APP的初始化流程
* 1.main 函数
* 2.执行UIApplicationMain
* 2.1 创建UIApplication对象
* 2.2 创建UIApplication的delegate对象
* 2.3 创建MainRunloop
* 2.4 delegate对象开始处理(监听)系统事件(没有storyboard)
* 3.根据Info.plist获得最主要storyboard的文件名,加载最主要的storyboard(有storyboard)
4. 程序启动完毕的时候, 就会调用代理的application:didFinishLaunchingWithOptions:方法
在application:didFinishLaunchingWithOptions:中创建UIWindow
创建和设置UIWindow的rootViewController
5. 最终显示第一个窗口
* 3.+load的调用时机与规则
当类被程序引用的时候就会调用类的+load方法，当程序启动时，会加载相关Mach-O文件，这个时候就会查找项目中那些文件被引用，这个时候就会调用+load。
1.当父类和子类都实现+load方法时,父类+load方法的执行顺序要优先于子类执行
2.当子类未实现+load方法时,不会调用父类+load方法
3.类中的+load方法执行顺序要优先于类别(Category)中的+load方法，虽然在APP启动流程中，Category的加载顺序在OC的+load方法之前，但是Category中的+load方法的执行顺序却在OC类的+load方法之后。
4.当有多个类别(Category)都实现了+load方法时,这几个+load方法都会执行,其执行顺序与类别文件在Build Phases里的Compile Sources中出现顺序一样。
5.由4可以得知，当有多个不同的类或者不同类的Category的时候,每个类+load 执行顺序与其在Compile Sources出现的顺序一致。
* 4.+initialize的调用实际与规则
不同于+load，类中的+initialize是在引用类被首次使用时被调用。也就是说也许你工程里import了一个类，但是你并没有调用这个类的任何方法，那么这个类的+initialize方法就不会被调用。而+initialize方法也只会调用一次，那就是首次调用这个类的任意方法时。
1.父类的+initialize方法会比子类的先执行
2.当子类未实现+initialize方法时,会调用父类+initialize方法,子类实现+initialize方法时,会覆盖父类+initialize方法.
3.当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法，因为覆盖的原因，所以最后一个覆盖了前面的)
