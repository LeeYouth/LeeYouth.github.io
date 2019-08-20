---
layout: post
title: iOS APP启动流程与生命周期
categories: iOS
description:iOS APP启动流程与生命周期
keywords: iOS, APP启动
---


iOS APP启动流程与生命周期



>前言
当自己对技术对APP的性能达到一定的追求时，就需要对APP有较深的了解，越是深入的了解和理解才能从各个点上去优化性能。深入的理解还可以使我们在对iOS进行逆向工程时更加了解从哪一个时间段什么方式入侵最合适。
我把APP的生命流程分为三大部分:
1. APP的启动流程(`pre-main`)
2. APP的初始化流程(`main`)
3. APP的运行时生命周期

#### APP的启动流程

1. iOS系统首先会加载解析该APP的Info.plist文件，因为Info.plist文件中包含了支持APP加载运行所需要的众多Key，value配置信息，例如APP的运行条件(Required device capabilities)，是否全屏，APP启动图信息等。
2. 创建沙盒(iOS8后，每次启动APP都会生成一个新的沙盒路径)
3. 根据Info.plist的配置检查相应权限状态
4. 加载Mach-O文件读取dyld路径并运行dyld动态连接器(内核加载了主程序，dyld只会负责动态库的加载)
1.1 首先dyld会寻找合适的CPU运行环境
1.2  然后加载程序运行所需的依赖库和我们自己写的.h.m文件编译成的.o可执行文件，并对这些库进行链接。
1.3 加载所有方法(runtime就是在这个时候被初始化的)
1.4 加载C函数
1.5 加载category的扩展(此时runtime会对所有类结构进行初始化)
1.6 加载C++静态函数，加载OC+load
1.7 最后dyld返回main函数地址，main函数被调用

**Mach-O文件说明:**
Mach-O文件格式是 OS X 与 iOS 系统上的可执行文件格式，类似于windows的 PE 文件。想我们编译产生的.o文件、程序可执行文件和各种库等都是Mach-O文件。
Mach-O文件主要有3部分组成：

1. Header：保存了一些基本信息，包括了该文件运行的平台、文件类型、LoadCommands的个数等等。Headers的主要作用就是帮助系统迅速的定位Mach-O文件的运行环境，文件类型。保存了一些dyld重要的加载参数
2. LoadCommands：可以理解为加载命令，在加载Mach-O文件时会使用这里的数据来确定内存的分布以及相关的加载命令。比如我们的main函数的加载地址，程序所需的dyld的文件路径，以及相关依赖库的文件路径。
3. Data： 每一个segment的具体数据都保存在这里，这里包含了具体的代码、数据等等。

**安全**
ASLR（Address Space Layout Randomization）：地址空间布局随机化，镜像会在随机的地址上加载。
代码签名：为了在运行时验证 Mach-O 文件的签名，并不是每次重复的去读入整个文件，而是把文件每页内容都生成一个单独的加密散列值，并把值存储在 __LINKEDIT 中。这使得文件每页的内容都能及时被校验确并保不被篡改。而不是每个文件都做hash加密并做数字签名。
如果要查看Mach-O文件可以用Mac OSX自带的otool工具,下面是一些常用的查看Mach-O文件命令

**dyld说明:**
`dyld`叫做动态链接器，主要的职责是完成各种库的连接。dyld是苹果用C++写的一个开源库，可以在苹果的git上直接查看源代码。
当系统从xnu内核态把控制权转交给dyld变成用户态后dyld首先初始化程序环境，将可执行文件以及相应的系统依赖库与我们自己加入的库加载进内存中，生成对应的ImageLoader类对应的image对象(镜像文件)，对这些image进行链接，调用各image的初始化方法等等(注:这里多数情况都是采用的递归，从底向上的方法调用)，其中runtime就是在这个过程中被初始化的，这些事情大多数在dyld:_mian方法中被发生。

```flow
op0=>operation: xnu加载Mach-O文件
op1=>operation: 从xnu内核态 控制权转到dyld用户态
op2=>operation: _dyld_sart
op3=>operation: dyld加载System framework以及一些dylib到内存
op4=>operation: Objc runtime 初始化 注册dyld_register_image_state_change_handler
op5=>operation: 在每次有新的镜像加入运行时的时候，进行回调执行 load_images将所有包含load方法的文件加入 loadable_classes
op0->op1->op2->op3->op4->op5->cond
```

对于动态库和静态库的链接方式是不同的，详细的大家可以看看我另外一篇关于 [动态库与静态库的文章](https://www.jianshu.com/p/d643c1368c9d)

**从整个APP启动流程中，我们可以做优化的点主要有**
1. 是减少系统依赖库
2. 减少自己需要加入的各种三方库(库越少dyld加载的速度越快，就能越早的返回程序入口main函数的地址)
3. 有一些自己加入的库，能选择静态库就选择静态库，少用动态库，因为动态库的加载方式比静态库慢。如果必须依赖动态库，则把多个非系统的动态库合并成一个动态库。
4. 自己加入的各种framework库根据情况设为optional和required，如果该framework在当前App支持的所有iOS系统版本中都存在，那么就设为required，否则就设为optional，因为optional会有些额外的检查会导致加载变慢。
5. 将不必须在+load方法中做的事情延迟到+initialize中，尽量不要用C++虚函数(创建虚函数表有开销)。
6. 减少项目文件中Category，静态变量等的使用数量.
7. 使用appCode检查项目中，那些类和方法没有使用到。 把没有使用到的删除。
8. 让UI大佬尽量把给的资源压缩到最小，因为在启动加载时会加载资源图片进行IO操作。所以图片小加载速度也会响应提升。
9. 内存上优化：类和方法名不要太长：iOS每个类和方法名都在__cstring段里都存了相应的字符串值，所以类和方法名的长短也是对可执行文件大小是有影响，及影响加载速度也消耗内存；因为OC的动态特性，都是加载后通过类/方法名反射找到这个类/方法进行调用，OC的对象模型会把类/方法名字符串都保存下来(压缩算法TinyPNG)。

**冷启动、热启动**
如果程序刚被运行过一次，那么程序的代码会被dyld缓存起来，因此即使杀掉进程再次重启加载时间也会相对快一点，如果长时间没有启动或者当前dyld的缓存已经被其他应用占据，那么这次启动所花费的时间就要长一点，这就分别是热启动和冷启动的概念。

**启动时间测试方法**
在 Xcode 中 Edit scheme -> Run -> Auguments 将环境变量 DYLD_PRINT_STATISTICS设为 1。随后启动APP时控制台会输出的启动耗时内容。

#### APP的初始化流程

1. main 函数
2. 执行UIApplicationMain
1.1 创建UIApplication对象
1.2 创建UIApplication的delegate对象
1.3 创建MainRunloop
1.4 delegate对象开始处理(监听)系统事件(没有storyboard)

3. 根据Info.plist获得最主要storyboard的文件名,加载最主要的storyboard(有storyboard)
4. 程序启动完毕的时候, 就会调用代理的application:didFinishLaunchingWithOptions:方法
在application:didFinishLaunchingWithOptions:中创建UIWindow
创建和设置UIWindow的rootViewController
5. 最终显示第一个窗口

**main.m文件说明**

``` objectivec

#import <UIKit/UIKit.h>
#import "AppDelegate.h"

//main函数是整个程序的入口
int main(int argc, char * argv[]) {
//参数argc说明:命令行总的参数个数。
//参数argv说明:是参数的数组，argv中第一个参数为app的路径＋全名。
printf("argc = %d\n", argc);
char *argChar = argv[0];
printf("index = %i ,argv = %s\n", 0, argChar);
@autoreleasepool {
//UIApplicationMain函数说明
//第一个参数argc:参数是main函数C语言中传入的，保持与main函数相同。
//第二个参数argv:同argc参数一样
//第三个参数nil:该参数为principalClassName (主要类名) 
//    如果principalClassName是nil，那么它的值将从Info.plist去获取，如果Info.plist没有，则默认为UIApplication。
//    principalClass这个类除了管理整个程序的生命周期之外什么都不做，它只负责监听事件然后交给delegateClass去做。
//第四个参数NSStringFromClass([AppDelegate class]):委托代理类的类名，UIApplication创建的delegate对象的类名
return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
}
}

```

APP初始化的UIApplication调用顺序为：
1. application:didFinishLaunchingWithOptions:
2. applicationDidBecomeActive:

**APP初始化流程的优化点**
1. 尽量使用纯代码而不是xib或者storyboard来进行UI框架的搭建，尤其是使用的TabBarController这种，尽量避免使用xib和storyboard，因为xib和storyboard也还是要解析成代码来渲染页面，并且官网为了满足更多的需求，必定做了更多的适配判断处理，会多很多步骤。会增加代码的执行效率从而增加启动时长。
2. 尽量在application:didFinishLaunchingWithOptions:中代码的执行时间。能多线程就多线程，能后台执行就后台执行。部分加载可以选择懒加载或者后台加载。不要阻塞主线程从而造成启动时间加长。

#### 生命周期
**ViewController的生命周期方法说明:(详细说明都在代码注释中)**

``` objectivec

#pragma mark --- sb相关的life circle
//执行顺序1
// 当使用storyBoard时走的第一个方法。这个方法而不走initWithNibName方法。
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
NSLog(@"%s", __func__);
if (self = [super initWithCoder:aDecoder])
{
//这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
}
return self;
}
#pragma mark --- life circle
//执行顺序1
// 当控制器不是SB时，都走这个方法。(xib或纯代码都会走这个方法)
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
NSLog(@"%s", __func__);
if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) 
{
//这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
}
return self;
}

//执行顺序2
// xib加载完成时调用，纯代码不会调用。系统自行调用
- (void)awakeFromNib {
[super awakeFromNib];
//当awakeFromNib方法被调用时，所有视图的outlet和action已经连接，但还没有被确定。
NSLog(@"%s", __func__);
}

//执行顺序3
// 加载控制器的self.view视图。(默认从nib)
- (void)loadView {
//该方法一般开发者不主动调用，应该由系统自行调用。
//系统会在self.view为nil的时候调用。当控制器生命周期到达需要调用self.view的时候会自行调用。
//或者当我们设置self.view=nil后，下次需要用到self.view时，系统发现self.view为nil，则会调用该方法。
//该方法一般会首先根据nibName去找对应的nib文件然后加载。
//如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)
NSLog(@"%s", __func__);
//该方法比较特殊，如果重写不能调用父类的方法[super loadView];
self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
}

//执行顺序4
//视图控制器中的视图加载完成，viewController自带的view加载完成后会第一个调用的方法
- (void)viewDidLoad {
//当self.view被创建后，会立即调用该方法。一般用于完成各种初始化操作
NSLog(@"%s", __func__);
[super viewDidLoad];
}

//执行顺序5
//视图将要出现
- (void)viewWillAppear:(BOOL)animated {
NSLog(@"%s", __func__);
[super viewWillAppear:animated];
}

//执行顺序6
// view 即将布局其 Subviews
- (void)viewWillLayoutSubviews {
//view即将布局它的Subviews子视图。 当view的的属性发生了改变。
//需要要调整view的Subviews子视图的位置，在调整之前要做的工作都可以放在该方法中实现
NSLog(@"%s", __func__);
[super viewWillLayoutSubviews];
}

//执行顺序7
// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {
//view已经布局其Subviews，这里可以放置调整完成之后需要做的工作
NSLog(@"%s", __func__);
[super viewDidLayoutSubviews];
}

//执行顺序8
//视图已经出现
- (void)viewDidAppear:(BOOL)animated {
NSLog(@"%s", __func__);
[super viewDidAppear:animated];
}

//执行顺序9
//视图将要消失
- (void)viewWillDisappear:(BOOL)animated {
NSLog(@"%s", __func__);
[super viewWillDisappear:animated];
}

//执行顺序10
//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
NSLog(@"%s", __func__);
[super viewDidDisappear:animated];
}

//执行顺序11
// 视图被销毁
- (void)dealloc {
//系统会在此时释放掉init与viewDidLoad中创建的对象
NSLog(@"%s", __func__);
}

//执行顺序12
//出现内存警告  //模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
//在内存足够的情况下，app的视图通常会一直保存在内存中，但是如果内存不够，一些没有正在显示的viewController就会收到内存不足的警告。
//然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常我们需要在这里将不需要显示在内存中保留的对象释放它的所有权，将其指针置nil。
NSLog(@"%s", __func__);
[super didReceiveMemoryWarning];
}

```

----
##### 补充：+load与+initialize

-  **+load的调用时机与规则**
当类被程序引用的时候就会调用类的+load方法，当程序启动时，会加载相关Mach-O文件，这个时候就会查找项目中那些文件被引用，这个时候就会调用+load。
1. 当父类和子类都实现+load方法时,父类+load方法的执行顺序要优先于子类执行
2. 当子类未实现+load方法时,不会调用父类+load方法
3. 类中的+load方法执行顺序要优先于类别(Category)中的+load方法，虽然在APP启动流程中，Category的加载顺序在OC的+load方法之前，但是Category中的+load方法的执行顺序却在OC类的+load方法之后。
4. 当有多个类别(Category)都实现了+load方法时,这几个+load方法都会执行,其执行顺序与类别文件在Build Phases里的Compile Sources中出现顺序一样。
5. 由4可以得知，当有多个不同的类或者不同类的Category的时候,每个类+load 执行顺序与其在Compile Sources出现的顺序一致。

- **+initialize的调用实际与规则**
不同于+load，类中的+initialize是在引用类被首次使用时被调用。也就是说也许你工程里import了一个类，但是你并没有调用这个类的任何方法，那么这个类的+initialize方法就不会被调用。而+initialize方法也只会调用一次，那就是首次调用这个类的任意方法时。
1. 父类的+initialize方法会比子类的先执行
2. 当子类未实现+initialize方法时,会调用父类+initialize方法,子类实现+initialize方法时,会覆盖父类+initialize方法.
3. 当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法，因为覆盖的原因，所以最后一个覆盖了前面的)

----
##### 参考文章
[iOS-APP的启动流程和生命周期](https://www.jianshu.com/p/229dd6190b95)
















