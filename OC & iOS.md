## viewController 的生命周期
1. `alloc` 创建对象，分配空间  
2. `init (initWithNibName)` 初始化对象，初始化数据  
3. `initWithNibName:bundle/initWithCoder` 
4. `loadView` 从nib载入视图 ，通常这一步不需要去干涉。除非你没有使用xib文件创建视图
5. `viewDidLoad` 载入完成，可以进行自定义数据以及动态创建其他控件
6. `viewWillLayoutSubviews ` view即将布局其Subviews
7. `viewDidLayoutSubviews ` view已经布局其Subviews
8. `viewWillAppear ` 视图将出现在屏幕之前，马上这个视图就会被展现在屏幕上了
9. `viewDidAppear ` 视图已在屏幕上渲染完成  
10. `viewWillDisappear ` 视图将被从屏幕上移除之前执行
11. `viewDidDisappear ` 视图已经被从屏幕上移除，用户看不到这个视图了
12. `dealloc ` 视图被销毁，此处需要对你在init和viewDidLoad中创建的对象进行释放  

_*当一个视图被移除屏幕并且销毁的时候的执行顺序，这个顺序差不多和上面的相反_  

关于 viewDidUnload ：在发生内存警告的时候如果本视图不是当前屏幕上正在显示的视图的话， viewDidUnload 将会被执行，本视图的所有子视图将被销毁，以释放内存,此时开发者需要手动对 viewLoad、viewDidLoad 中创建的对象释放内存。 因为当这个视图再次显示在屏幕上的时候，viewLoad、viewDidLoad 再次被调用，以便再次构造视图。

layoutSubviews 在以下情况下会被调用：  

1. init 初始化不会触发 layoutSubviews
2. addSubview 会触发 layoutSubviews
3. 设置 view 的 Frame 会触发 layoutSubviews，当然前提是frame 的值设置前后发生了变化
4. 滚动一个 UIScrollView 会触发 layoutSubviews
5. 旋转Screen会触发父 UIView 上的 layoutSubviews事件
6. 改变一个 UIView 大小的时候也会触发父 UIView 上的layoutSubviews 事件



## NSURLSession

NSURLSession是由NSURLSessionConfiguration和optional delegate构成。通过网络需求通过NSURLSessionTask来建立session

NSURLSessionConfiguration 对NSURLSession会话属性进行配置，有三种工作模式。

- defaultSessionConfiguration 
- ephemeralSessionConfiguration
- backgroundSessionConfiguration  

config的几种属性

- networkServiceType：一般不需要设置这个。
- allowsCellularAccess和discretionary：后台传输使用discretionary这个属性。
- timeoutIntervalForRequest和timeoutIntervalForResource：前者是packet之间请求时间。如果希望限制整体超时时间，使用后者timeoutIntervalForResource
- HTTPMaximumConnectionsPerHost：限制连接到主机的数量
- HTTPShouldUsePipelining：默认禁止，因为很多服务器没有支持
- sessionSendsLaunchEvents：session是否应该从后台启动
- connectionProxyDictionary：指定连接的代理服务器。


## KVO 是同步的还是异步的？
同步的。为了保证属性变数时及时作出响应。
陷阱：一旦在后台线程修改监听对象的键值，会在相同线程调用 KVO 的方法。
一旦在 KVO 执行的方法中，如果要修改某一个对象的属性，可能会涉及到线程安全问题。一不小心，就会出现资源抢夺的问题，需要考虑使用互斥锁。

提示：KVO 在日常开发中，绝大多数应该尽量设计的简单。最好坚挺的对象属性，不要跨线程修改。如果一定要在后太修改，注意在监听方法中，代码要考虑加锁。
实际：一般不会使用 KVO，但是有些时候，必须使用 KVO 才能解决问题。

不要忘记 dealloc (所有观察者模式不用的时候都要把观察者去掉)

_*自动布局的底层就是KVO实现的_

## 下面是 bestswifter 大大博客面试题部分回答

MVC 具有什么样的优势，各个模块之间怎么通信，比如点击 Button 后 怎么通知 Model？

MVC M（Model 层）处理数据和业务逻辑、V（View）展示视图、C（Controller）Model 和 View 之间的桥梁，监听时间，根据需要操作数据。  
优点：

1. 低耦合。
2. 分工明确容易维护，也有利于团队开发。

各模块的的通讯：

- M -> C: 通过 notification 或者 KVO 传递 Model 中数据的更新
- C -> V: 通过赋值...
- V -> C: delegate & block & data sources 将用户操作委托给 controller 去判断。View 负责得到答案并展示

Button 通知 Model：点击触发方法，比如页面数据要更新，让后在 Controller 层调用 Model 层中的 API 来完成。

## UITableView 的相关优化

系统方面：cell 重用机制，系统只生成屏幕中显示的 cell 的个数，在 cell 进入屏幕时从重用队列中拿出一个对象；在 cell 退出屏幕放入重用队列。 其中维护的是 key 为 identifier，value为cell的一个字典。在重用之前需要 registerClass:forCellIdentifier。
编程方面：UITableView 是继承 UIScrollView 的，所以要首先确定的是 cell 的 contentSize 和 cell 的位置。由于tableView的机制 tableView:heightForRowAtIndexPath 方法在 tableView:cellForRowIndexPat 方法之前调用，并且前者是多次调用的，来确定 contentSize 和 Cell 的位置。也就需要我们去预先估计 cell 的高度。这里的优化就是在获取数据的时候计算 cell 内容的高度，并且缓存起来。这样就不用每次都计算其高度了。

###小结：
1. 提前计算并缓存 cell 的高度。
2. 遇到大量图片一定需要异步加载。
3. 遇到复杂的布局，需要异步你绘制。

## KVO、Notification、delegate 各自的优缺点，效率还有使用场景

KVO （Key-Values Observing）键值观察：
优点：
1. 使用起来比较方便调用 addObserver：forKeyPath：options：context，注意 remove。 
2. 能够对非自己创建的对象的属性进行监听。eg：监听tableView 或者 scrollView 的 contentOffset。
3. 能够给观察者提供被观察的改变的新值和旧值。eg：NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld
缺点：
1. 观察的属性必须是使用 NSString 来定义的。不容易查错。

Notification 通知 （在哪个线程中发送通知就在哪个线程中响应通知） Notification Center 通知中心，用 name 来唯一标示。  
优点：
1. 能进行1对多的通知。
2. 通知可以传递自定义的信息。这只 UserInfo（NSDictionary）
缺点：
1. 在调试的时候难以追踪。
2. 注意 remove 通知中心

delegate 代理：
优点：
1. 语法严格。在 delegate 的声明中有清晰的定义。
2. 能接收处理的返回值。有反馈。
缺点：
1. 容易产生循环引用。 delegate 的属性修饰 （weak）。
2. 当 delegate 的对象被释放（nil）之后，容易产生 crash

## 如何手动通知 KVO

## Objective-C 中的 copy 方法

对于 NSString 来说，copy 是浅拷贝，mutableCopy 是深拷贝
但是自定义类的话，对于 copy 和 mutalbleCopy 是否是深拷贝还是浅拷贝，要看具体方法的实现。

在 OC copy 方法属于浅拷贝（将对象的指针拷贝，地址相同）。
一个对象能够被 copy 的前提是该类遵循 NSCopying 协议，并实现 - (id) copyWithZone:(NSZone *) zone 方法。

## runtime 中，SEL、IMP 和 Method 的区别  

SEL 方法名称。 runtime时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(Int类型的地址)

IMP 指向该方法的具体实现的函数指针。指向方法实现的首地址
Method 对应 objc_method 结构体。相当于把 SEL 和 IMP 关联起来，找到 SEL 就可以找到其实现。

```
objc_method {
    SEL method_name
    char *method_type
    IMP method_imp
}


objc_msgSend(receiver, selector)
```

## autoreleasepool 的使用场景和原理 

对于 autoreleasepool 数据结构类似于栈，@autoreleasepool{} 这样来规定一个自动释放池的作用域，再超出改作用域范围的变量，系统对每个变量发送一个 release 消息。在 main.m 的文件里默认存在一个 autoreleasepool 但是对于 autoreleasepool 的生命周期结束时在主线程中的 runloop 迭代结束时才被释放。autoreleasepool 的底层实现一个双链表，每个节点 NSAutoreleasePage，每个节点的大小 1024 字节。在 ARC 中在@autoreleasepool{} 内所创建的对象都会被记录在这个 NSAutoreleasePage 中（入栈一样），｛｝的起始点对应双链表中的一个标志位（flag）。在释放池中对象的时候，依次 pop 到标志位为止。autoreleasepool 可以嵌套。

场景：在一个 for 循环中创建大量的图片，此时系统瞬间内存达到峰值。此对象释放不到，此时，需要将创建图片的代码加入 @autoreleasepool{} 中，对象就可以在一次循环处理完之后，自动释放。

## RunLoop 的实现原理和数据结构，什么时候会用到

原理 ：runloop 是一个 ‘等待事件 -> 处理事件’ 的循环过程，等到接收到 quit 的信息之后，runloop 也就结束了。其中 runloop 有四种 Mode（Common（其他三个的集合）, default（默认模式／空闲状态）, initilize（初始化）, track（滚动模式）），对我们可用的 Common, default 两个 Mode。在 UIScrollView 滑动的时候主线程的 runloop 的 Mode 会自动切换到 track 模式。

##block 为什么会有循环引用


## 使用 GCD 如何实现这个需求：A、B、C 三个任务并发，完成后执行任务 D 

使用 dispatch group， 具体就是 dispatch_group_create() 得到一个 dispatch_group_t 对象。再将 A, B, C 任务 dispatch_group_async（group,queue,^{}）的方追加到 group 中，最后使用 dispatch_notify(group,queue,^{}) 来处理 D

## NSOperation 和 GCD 的区别 

从实现的角度：GCD 是用 C语言编写，NSOperation 对 GCD 的高层封装

从功能的角度：NSOperation 可以定制 queue 之间的依赖关系 调用 addDependency:。可以规定最大的并发数量 maxConcurrentOperationCount。有三种状态可以监听 isExecuted，isFinishing，isCancel。 可以取消任务，暂停，恢复。GCD 做不到。

## Core Data 的使用，如何处理多线程问题


## 如何设计图片缓存？


## 有没有自己设计过网络控件？

## 如果页面 A 跳转到 页面 B，A 的 viewDidDisappear 方法和 B 的 viewDidAppear 方法哪个先调用？

如果是present的话是B的viewDidAppear先调用，如果是导航栏push的话是A的viewDidDisappear先调用（页面 B 的 viewWillAppear 方法 -> 页面 A 的 viewDidDisappear 方法 -> 页面 B 的 viewDidAppear 方法）。


## block 循环引用问题

循环引用问题原因：对象互相持（强引用）有对方的成员变量。[A setA:B.b];[B setB:A.a]; 或者说是一个 A 持有 B 对象有一条有向边的话，图中出现环路，说明发生循环引用了。

```
[A setA:B.b];
[B setB:A.a]; 
```

A 强引用 B 中的成员变量 b，B 强引用 A 中的成员变量 a。导致 A 对象中含有引用计数不为0的实例（a 被 B 持有）。所以 A 无法被释放。B 对象中含有引用计数不为0的实例（b 被 A 持有）。所以两者在超出作用域范围之后，仍然无法释放，也就是产生了内存泄漏问题。

解决办法：在发生循环引用的对象其一，用弱引用。来避免循环引用。

## ARC 的本质 （－fobjc-arc / -fobjc-no-arc）

自动引用计数。编译器帮助我们管理引用计数。

## RunLoop 的基本概念，它是怎么休眠的？


## Autoreleasepool 什么时候释放，在什么场景下使用？

释放时间：在主线程 Runloop 迭代结束的时候释放，Runloop 每次迭代都会生成或者废弃 NSAutorelasePool 对象。

场景：在一个 for 循环中创建大量的图片，此时系统瞬间内存达到峰值。此对象释放不到，此时，需要将创建图片的代码加入 @autoreleasepool{} 中，对象就可以在一次循环处理完之后，自动释放。

## 并行和并发的区别

并发程序含有多个逻辑上的独立执行块，它们可以独立地并行执行，也可以串行执行；而并行程序是同时执行整个任务的多个部分，并行程序可能有多个独立的执行块，也可能只有一个。

如果从处理器和任务的角度来讲，粗略的理解可以是：

并发是单个处理器执行多个任务，这些任务在重叠的时间段内交叉执行，但在任何一个时间点都只有一个任务在执行。这些任务在逻辑上是同时执行，而实际上是交叉执行的。
并行是多个处理器或多核下执行多个任务，这些任务可以同时执行，即同一时间点可以有多个不同的任务在执行。这些任务在物理上是同时执行的。

简单点说就是：并行指物理上同时执行，并发指能够让多个任务在逻辑上交织执行的程序设计

# iOS 中小 Tips  
1. UIWebview 中页面在出发一个点击事件（特指 URL 跳转的那种事件），如果是 URL scheme 等，UIWebView 处理不了会转发到 appdelegate 中的代理 `application:openURL:sourceApplication:annotation:` 来处理；但是对于 WKWebview 来说，不会转发到 appdelegate 中，如果需要处理的链接可以在 `webView:decidePolicyForNavigationAction:
decisionHandler:` 来做处理或者转发。
2. 方便的 app 内嵌页面的调试或者开发。
 - [Safari] -> [偏好设置] -> [高级] -> [在菜单栏中显示‘开发’]
 - [手机] -> [设置] -> [Safari] -> [高级] -> [web 检查器]

3. 关于做一个 framework 需要注意的，在同一个 module 中的，默认的 internal 的访问层级就可以让 ViewController 访问到关于 Timer 和相应方法的信息。但是它们处于不同的 module 中，所以我们需要对 Timer.swift 的访问权限进行一些修改，在需要外部访问的地方加上 public 关键字。简单说就是 private 只允许本文件访问，不写的话默认是 internal，允许统一 module 访问，而要提供给别的 module 使用的话，需要声明为 public。  

  - fileprivate :如名字一样,只有这个文件才能访问.
  - private: 只能在作用域访问.
  - interal: 默认,在整个模块可以访问.
  - public: 在模块里面是可以继承或者重写,在模块外可以访问,但不可以重写和继承.
  - open:在所有模块都可以访问,重写和继承.
open> public > interal > fileprivate > private

4. 远程推送添加 Capabilities 的时候自动增加的 lukou.entitlements 文件中的 APS-Enviroment 字段会自动根据证书来确定是 development 还是 producation 。所以打包发布的时候这个字段会自动改变
5. 关于 share extension 在 safari 中注入 js。在 NSExtensionAttributes 节点下 增加 `NSExtensionJavaScriptPreprocessingFile ＝ ExtensionPreprocessingJS` 字段，其中并在 extension 的 根目录下添加一个名为 ExtensionPreprocessingJS 的 js 文件。其中内容的模版如下  

	```
	var MyPreprocessor = function() {};
	
	MyPreprocessor.prototype = {
	    run: function(arguments) {
	        arguments.completionFunction({"URL": document.URL})
	};
	
	var ExtensionPreprocessingJS = new MyPreprocessor;
	```  

	值得注意的是 `var ExtensionPreprocessingJS = new MyPreprocessor;` 变量名是固定的。 在 `{"URL": document.URL}` 类似 json 格式。可以进行 DOM 操作来得到网页中的值。

	```
	[self.extensionContext.inputItems enumerateObjectsUsingBlock:^(NSExtensionItem *  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
	        [obj.attachments enumerateObjectsUsingBlock:^(NSItemProvider *  _Nonnull itemProvider, NSUInteger idx, BOOL * _Nonnull stop) {
	            if ([itemProvider hasItemConformingToTypeIdentifier:(NSString *)kUTTypePropertyList]) {
	                [itemProvider loadItemForTypeIdentifier:(NSString *)kUTTypePropertyList options:nil completionHandler:^(NSDictionary *jsDict, NSError *error) {
	                    dispatch_async(dispatch_get_main_queue(), ^{
	                        NSDictionary *jsPreprocessingResults = jsDict[NSExtensionJavaScriptPreprocessingResultsKey];
	                        NSString *url = jsPreprocessingResults[@"URL"];
	                    });
	                }];
	            }
	        }];
	    }];
	```  


