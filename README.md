# 编写高质量iOS与OS X代码的52个有效方法
读后总结

1. OC是为C语言添加了面向对象的特性，是C的超集。OC使用动态绑定的消息结构，既在运行时才检查对象的类型。
2. 导入头文件时，尽量使用向前声明的(@class)。
3. 最好使用字面量创建字符串、数值、数组、字典。
4. 多用类型常量，少用#define。
#define 不含类型信息，编译器不会对其报警告。
对外常量在.h中使用extern，并在.m中实现。
内部常量在.m中使用 static const
5. 使用枚举表示状态、选项、状态码。
6. @property 用来封装存储实例对象的数据，注意使用的内存管理语义。
7. 在没有重载setter和getter方法，也没有使用KVO时，可以在类实现内部直接使用实例变量，不过要确保没有内存问题；使用setter方法写数据。在init和dealloc中使用实例变量读写。
8. 比较对象， == 比较的是指针。NSObject 协议中比较对象的关键方法 - (BOOL)isEqual:(id)object;
@property (readonly) NSUInteger hash;
9. 类簇将一些私有的、具体的子类组合在一个公共的、抽象的超类下面，以这种方法来组织类可以简化一个面向对象框架的公开架构，而又不减少功能的丰富性。
10. runtime 中可以关联对象，并指定为是否拥有(强引用)关联对象
11. OC中对象调用方法都会由 void objc_send(id self, SEL cmd, ….) 函数进行动态消息发送。此函数的由 接受者、选择器和不定数的参数组成
12. 消息的转发，调用方法时方法的查找顺序，1. 对象的方法缓存列表、2. 对象的方法列表 3. 父类方法列表 只到根类。
如果找不到则会进行消息转发,1、 + (BOOL)resolveInstanceMethod:(SEL)sel，2、若是已返回 NO，则转发给 - (id)forwardingTargetForSelector:(SEL)aSelector，3、如实2返回nil，则转发给 - (void)forwardInvocation:(NSInvocation *)anInvocation，若是 3 依然处理不了此条消息则抛出异常。

13. runtime 可以改变方法的实现地址（IMP）：动态添加方法，替换方法、交换方法等
Method _Nullable class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name)获取对象方法的
void method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2) 交换两个方法的 method

IMP _Nullable class_getMethodImplementation(Class _Nullable cls, SEL _Nonnull name)  获取方法的实现地址
IMP _Nullable class_replaceMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types) 替换方法的实现地址

14. 类(Class)在runtime文件中的定义  typedef struct objc_class *Class; 类是一个objc_class类型结构体，其定义结构可去<objc/runtime.h>中查看
typedef struct objc_object *id; id是 objc_object 类型的结构，只有一个 Class *isa 指针

15. 使用特定的前缀来创建类和接口，避免命名冲突。
16. 创建一个全能init方法，其他init方法都应调用全能init方法。若全能init方法与父类不同，则要重载父类中的对应方法。若父类的全能init方法不适用与子类，则要在子类中重载，并作出对应处理，比如抛出异常。
17. 按需要实现description 或 debugDescription方法，便于调试。
18. 尽量使用不可变对象。若属性仅在内不可修改，则定义为readonly，在class-continuation分类中重定义为readwrite。可变属性尽量不要公开，使用提供相关修改用的方法替代。
19. 使用清晰协调的命名方式。遵从OC的命名规范，让其读起来简单易懂。
20. 给私有的方法加前缀，如abc_ ，易于和公共方法区分。
21. 一般只在发生严重的地方才抛出异常，其他可以提供返回error信息的参数。
22. 自定定义的类只有遵从 NSCopy 或 NSMutableCopy 并实现对应协议的方法，才能直接使用copy。一般使用浅copy，只有在需要时才进行深copy。
23. 使用委托和数据源的协议进行对象间的通信。
24. 将类的实现代码分散到多个便于管理的分类中去，可将视为私有的方法放到名为Private的分类中
25. 总为第三方类的分类添加前缀，方法名也加上前缀，以防和系统冲突
26. 尽量不要在分类中声明属性，尽管可以使用关联对象来实现属性的定义，但要注意内存管理语义
27. 对不需要公开的方法，实例变量，遵从的协议和外部只读内部可读写的属性可使用 class-continuation（没有名字的分类）在本类的实现文件里声明
28. 通过协议提供匿名对象，也就是准从某协议的id类型，不需要知道此对象的任何信息，只要实现协议就可以
29. 正确理解 引用计数 管理内存，注意循环引用
30. ARC可以省去内存管理语义，它是在编译时再适合的地方自动添加管理语义，注意循环引用，CoreFoundation对象必须手动释放，在ARC中以alloc、new、copy、mutableCopy开头的方法一般要保留创建的对象
31. 在dealloc 只用来释放对象，移除监听。
32. 捕获异常时一定要注意@try内对象的能存管理，处理error也要注意释放能存后在返回error；默认情况下ARC在出现异常时，是不会自动清理异常代码块内的能存的，可以开启-fobjc-arc-exceptions但就是编译后程序变大，运行效率变低
33. 无论是不是ARC环境都要注意避免出现循环引用，可以使用weak、assign，__block等来避免强引用环
34. 自动释放池（autoreleasepool）是以栈的形式存储对象的；嵌套的释放池是以在上一个池里加入一个哨兵开始的，释放时遇到哨兵结束。
35. 使用“僵尸对象”(Enable Zombie Objects)调试内存管理，也就是已经释放的对象，在开启“僵尸对象”不会真的释放，而是被转换成僵尸类对象，在使用僵尸对象时，会收到其对应的异常信息。
36. 不要使用 retainCount 方法，对于单利、系统框架使用的对象、字面常量都会得到一个超出你想象的结果；自动释放池会延迟retainCount的递减；很多情况下对象释放时其retainCount是不为0的。
37. 理解 块(block) 这一概念。
38. 为常用的 block  创建 typedef；typedef void(^VSAlertViewActionHandler)(NSString * _Nullable, NSInteger);
39. 使用 handler 块降低代码分散程度。 + (instancetype)alertViewControllerWithTitle:(nullable NSString *)title message:(nullable NSString *)message buttonTitles:(NSArray<NSString *> * _Nullable)buttonTitles actionHandler:(VSAlertViewActionHandler _Nullable )actionHandler;
40. block会捕获并保留外部变量的引用计数，使用时注意避免出现循环引用。
41. 能用派发队列，就不要用同步锁(@synchronized, NSLock)。
42. 能使用GCD，就不要用performSelector。后者在参数和返回值上都有局限，而且还可能造成内存泄露。
43. 处理多线程可以使用：GCD 或NSOperationQueue。了解他们的差异
44. dispatch group可以执行一系列操作，并在结束时得到通知；其在并发队列中会根据系统资源状况来调度并发执行的任务。
45. 使用dispatch_once来执行只运行一次的线程安全代码。
46. dispatch_get_current_queue已经启用
47. 熟悉系统框架。
48. 多使用块枚举，少使用for。
49. 对自定义其内存管理语义的collection使用无缝桥接__bridge、__bridge_returned、__bridge_transfer。
50. 做缓存可以使用NSCache替代NSDictionary。
51. 在+load 和+initialize 中尽量少做操作。
52. NSTimer会保留其target，使用后必须调用invalidate将其无效化然后设置为nil。
