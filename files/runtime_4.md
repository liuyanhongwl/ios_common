## Runtime 4 isa swizzling


### 介绍

对比上一篇 [Runtime 3 Method Swizzling](https://www.jianshu.com/p/944c06b316aa)， isa swizzling 顾名思义就是把对象的 isa 指针进行替换。


根据第一篇 [Runtime 1 简介,对象、类的结构,消息传递](http://www.jianshu.com/p/c546d3e7858d)，我们知道对象都有 isa 指针指向它的类，消息传递时也通过isa指针找到类中所对应的方法。更改对象的 isa 指针，不仅改变了它所属于的类型，也更改了它的行为（方法）。

举个例子：

```
@interface Father: NSObject
@property (nonatomic, assign) NSInteger f;
@end
@implementation Father
@end

@interface Child: Father
@property (nonatomic, assign) NSInteger c;
@end
@implementation Child
@end
```
然后执行：

```
//1
Child *child = [[Child alloc] init];
Class class = object_getClass(child); //Child
//2
NSLog(@"%ld, %ld", child.c, child.f);  //0, 0
//3
object_setClass(child, [NSObject class]);
class = object_getClass(child); //NSObject
//4
NSLog(@"%ld, %ld", child.c, child.f);  //error: -[NSObject c]: unrecognized selector sent to instance 0x60400002aaa0
```

上面的代码就是将 child 对象进行 isa swizzling，具体步骤分析如下：

1. 上面代码创建了实例 child，它的 isa 指向 Child 类。
2. `child.c` 和 `child.f` 是通过消息传递找到方法实现的，通过 child 的 isa 指针找到它的 Child 类，然后在 Child 类中的 method list 找 c。f 方法的查找同理，只是多了一步在 Child 类中找不到，则往它的 superclass 中找。
3. `object_setClass(child, [NSObject class])` 方法将 child 对象的 isa 指向 NSObject。
4. 这时再进行消息传递时，查找的是 child 对象的 isa 指向的 NSObject 类，由于 NSObject 类没有对应的 c 和 f 方法，最终找不到方法程序崩溃。


### 应用之KVO

KVO在调用存取方法之前总是调用 willChangeValueForKey: ，之后总是调用 didChangeValueForkey: 。怎么做到的呢?答案是通过 isa 混写（isa-swizzling）。

[Apple 的文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOImplementation.html)对 KVO 实现的描述：

>Automatic key-value observing is implemented using a technique called isa-swizzling.
>
>...
>
>When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class.
>
>...

从Apple 的文档可以看出：Apple 并不希望过多暴露 KVO 的实现细节。不过，要是借助 runtime 提供的方法去深入挖掘，所有被掩盖的细节都会原形毕露：

>当你观察一个对象时，一个新的类会被动态创建。这个类继承自该对象的原本的类，并重写了被观察属性的 setter 方法。重写的 setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象：值的更改。最后通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。我画了一张示意图，如下所示：

<img src="../images/runtime/isa_swizzling_kvo.png"/>

然而 KVO 在实现中使用了 isa-swizzling 的确不是很容易发现：Apple 还重写了`-class`方法并返回原来的类。企图欺骗我们：这个类没有变，就是原本那个类。。。如下：

```
Father *father = [[Father alloc] init];
[father addObserver:self forKeyPath:@"f" options:NSKeyValueObservingOptionNew context:nil];
NSLog(@"%@", object_getClass(father)); //NSKVONotifying_Father
NSLog(@"%@", father.class); //Father
```

假设“被监听的对象”的类对象是 MYClass ，有时候我们能看到对 NSKVONotifying_MYClass 的引用而不是对 MYClass 的引用。借此我们得以知道 Apple 使用了 isa 混写（isa-swizzling）。具体探究过程可参考 这篇博文 。

由下面执行过程可知，通过KVO生成的中间类继承原来的类：

```
Class kvoClass = object_getClass(father);
Class kvoSuperClass = class_getSuperclass(kvoClass);
NSLog(@"%@", kvoSuperClass); //Father
```

### 注意

isa-swizzling 改变了对象说属类型，因此更改范围比 method-swizzling 范围更广，使用时要更加注意。

runtime 极其尖锐，选择使用 runtime 时要清楚每一步的真正原理。


