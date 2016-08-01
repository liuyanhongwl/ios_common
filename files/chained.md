## 链式编程思想

### 已知编程思想： 

#### 1. 面向过程：处理事情以过程为核心，一步一步实现。

#### 2. 面向对象：万物皆对象。

#### 3. 链式编程思想：将多个操作（多行代码）通过点号（.）链接在一起成为一句代码，使代码可读性好。（`a(1).b(2).c(3)`）

* 链式编程特点：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值）
* 代表：Masonry框架。

#### 4. 响应式编程：不需要考虑调用顺序，只需要知道考虑结果，类似于蝴蝶效应，产生一个事件，会影响很多东西，这些事件像流一样的传播出去，然后影响结果，借用面向对象的一句话，万物皆是流。

* 代表：KVO运用。

#### 5. 函数式编程思想：是把操作尽量写成一系列嵌套的函数或者方法调用。
函数式编程特点：每个方法必须有返回值（本身对象）,把函数或者Block当做参数,block参数（需要操作的值）block返回值（操作结果）

* 代表：ReactiveCocoa。

### 我们这里以链式编程思想代码实现一个计算器:

```
#import
@class CaculatorMaker;
@interface NSObject (CaculatorMaker)

//计算
+ (int)makeCaculators:(void(^)(CaculatorMaker *make))caculatorMaker;

@end

#import "NSObject+CaculatorMaker.h"
#import "CaculatorMaker.h"

@implementation NSObject (CaculatorMaker)

//计算
+ (int)makeCaculators:(void(^)(CaculatorMaker *make))block
{
    CaculatorMaker *mgr = [[CaculatorMaker alloc] init];
    block(mgr);
    return mgr.iResult;
}

@end

#import

@interface CaculatorMaker : NSObject

@property (nonatomic, assign) int iResult;

//加法
- (CaculatorMaker *(^)(int))add;

//减法
- (CaculatorMaker *(^)(int))sub;

//乘法
- (CaculatorMaker *(^)(int))muilt;

//除法
- (CaculatorMaker *(^)(int))divide;

@end


#import "CaculatorMaker.h"

@implementation CaculatorMaker

- (CaculatorMaker *(^)(int))add
{
   return ^(int value)
    {
        _iResult += value;
        return self;
    };
}

@end
```

调用

```
int iResult = [NSObject makeCaculators:^(CaculatorMaker *make) {
     make.add(1).add(2).add(3).divide(2);
   }];
```

分析下这个方法执行过程：
第一步：NSObject 创建了一个block, 这个block里创建了一个CaculatorMaker对象make，并返回出来
第二步：这个对象make调用方法add时，里面持有的属性iResult做了一个加法，并且返回自己，以便可以接下去继续调用方法。 
这就是链式编程思想的一个很小但很明了的例子。


