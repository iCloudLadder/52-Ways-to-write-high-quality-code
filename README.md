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







24. 将类的实现代码分散到多个便于管理的分类中去，可将视为私有的方法放到名为Private的分类中
25. 总为第三方类的分类添加前缀，方法名也加上前缀，以防和系统冲突
26. 尽量不要在分类中声明属性，尽管可以使用关联对象来实现属性的定义，但要注意内存管理语义
27. 对不需要公开的方法，实例变量，遵从的协议和外部只读内部可读写的属性可使用 class-continuation（没有名字的分类）在本类的实现文件里声明
