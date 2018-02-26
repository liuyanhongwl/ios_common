## Runtime 4 isa swizzling


### 介绍

对比上一篇 [Method Swizzling](https://www.jianshu.com/p/944c06b316aa)， isa swizzling 顾名思义就是把对象的 isa 指针进行替换。


根据[第一篇](http://www.jianshu.com/p/c546d3e7858d)，我们知道对象都有 isa 指针指向它的类，消息传递时也通过isa指针找到类中所对应的方法。更改对象的 isa 指针，不仅改变了它所属于的类型，也更改了它的行为（方法）。

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

int main(int argc, char * argv[]) {
	 //1
    Child *child = [[Child alloc] init];
    Class class = object_getClass(child); //Child
    //2
    NSLog(@"%ld, %ld", child.c, child.f);  //0, 0
    //3
    object_setClass(child, [NSObject class]);
    class = object_getClass(child); //NSObject
    //4
    NSLog(@"%ld, %ld", child.c, child.f);  //error
    return 0;
}
```
上面的代码就是将 child 对象进行 isa swizzling，具体步骤分析如下：

1. 上面代码创建了实例 child，它的 isa 指向 Child 类。
2. `child.c` 和 `child.f` 是通过消息传递找到方法实现的，通过 child 的 isa 指针找到它的 Child 类，然后在 Child 类中的 method list 找 c。f 方法的查找同理，只是多了一步在 Child 类中找不到，则往它的 superclass 中找。
3. `object_setClass(child, [NSObject class])` 方法将 child 对象的 isa 指向 NSObject。
4. 这时再进行消息传递时，查找的是 child 对象的 isa 指向的 NSObject 类，由于 NSObject 类没有对应的 c 和 f 方法，最终找不到方法程序崩溃。


### 应用之KVO


KVO在调用存取方法之前总是调用 willChangeValueForKey: ，之后总是调用 didChangeValueForkey: 。怎么做到的呢?答案是通过 isa 混写（isa-swizzling）。