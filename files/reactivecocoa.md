## ReactiveCocoa

### 编程思想 

[这里](chained.md)

### ReactiveCocoa 编程思想

ReactiveCocoa结合了几种编程风格：

- 函数式编程（Functional Programming）：使用高阶函数，例如函数用其他函数作为参数。
- 响应式编程（Reactive Programming）：关注于数据流和变化传播。
 
所以，你可能听说过ReactiveCocoa被描述为函数响应式编程（FRP）框架。

比喻：

>可以把信号想象成水龙头，只不过里面不是水，而是玻璃球(value)，直径跟水管的内径一样，这样就能保证玻璃球是依次排列，不会出现并排的情况(数据都是线性处理的，不会出现并发情况)。水龙头的开关默认是关的，除非有了接收方(subscriber)，才会打开。这样只要有新的玻璃球进来，就会自动传送给接收方。可以在水龙头上加一个过滤嘴(filter)，不符合的不让通过，也可以加一个改动装置，把球改变成符合自己的需求(map)。也可以把多个水龙头合并成一个新的水龙头(combineLatest:reduce:)，这样只要其中的一个水龙头有玻璃球出来，这个新合并的水龙头就会得到这个球。

#### 1. RACSignal

ReactiveCocoa signal（RACSignal）发送事件流给它的subscriber。目前总共有三种类型的事件：next、error、completed。一个signal在因error终止或者完成前，可以发送任意数量的next事件。

每次next事件发生时，subscribeNext:方法提供的block都会执行。

#### 2. 基本UI添加了signal

ReactiveCocoa框架使用category来为很多基本UIKit控件添加signal

#### 3. filter: 过滤某些事件

#### 4. map: 转化事件值

map操作通过block改变了事件的数据。

#### 5. RAC宏

RAC宏允许直接把信号的输出应用到对象的属性上。RAC宏有两个参数，第一个是需要设置属性值的对象，第二个是属性名。每次信号产生一个next事件，传递过来的值都会应用到该属性上。

#### 6. 聚合信号

`combineLatest:reduce:` 把多个signal产生的最新的值聚合在一起，并生成一个新的信号。每次这两个源信号的任何一个产生新值时，reduce block都会执行，block的返回值会发给下一个信号。

#### 7. 创建信号

`createSignal:` 方法的入参是一个block，这个block描述了这个信号。当这个信号有subscriber时，block里的代码就会执行。

block的入参是一个subscriber实例，**它遵循RACSubscriber协议，协议里有一些方法来产生事件**，你可以发送任意数量的next事件，或者用error\complete事件来终止。

这个block的返回值是一个RACDisposable对象，它允许你在一个订阅被取消时执行一些清理工作。当前的信号不需要执行清理操作，所以返回nil就可以了。

#### 8. 信号中的信号

`flattenMap:` 把原来的信号转成另一个信号传播

#### 9. 添加附加操作

`doNext:` doNext: block并没有返回值。因为它是附加操作，并不改变事件本身。

#### 10.内存管理

ReactiveCocoa设计的一个目标就是支持匿名生成管道这种编程风格。

为了支持这种模型，ReactiveCocoa自己持有全局的所有信号。如果一个signal有一个或多个订阅者，那这个signal就是活跃的。如果所有的订阅者都被移除了，那这个信号就能被销毁了。

上面说的就引出了最后一个问题：如何取消订阅一个signal？在一个completed或者error事件之后，
订阅会自动移除（马上就会讲到）。你还可以通过RACDisposable 手动移除订阅。

RACSignal的订阅方法都会返回一个RACDisposable实例，它能让你通过dispose方法手动移除订阅。

#### 11. 链接signal

`then:` 方法会等待completed事件的发送，然后再订阅由then block返回的signal。这样就高效地把控制权从一个signal传递给下一个。

then方法会跳过error事件，被接下来的subscribeNext:error: 方法的error block接收。

#### 12. 线程

`deliverOn:` 把next事件流切换到不同的线程

`subscribeOn:` 确保signal在指定的scheduler上执行

#### 13. 节流

`throttle:` 只有当，前一个next事件在指定的时间段内没有被接收到后，throttle操作才会发送next事件。就是这么简单。





### RACCommand

### RAC 手势

### 参考

关于ReactiveCocoa基本功，BenBeng的文章写得很好，图文并茂得描述了一个小项目的RAC开发流程：

- [ReactiveCocoa入门教程——第一部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part1) 
- [ReactiveCocoa入门教程——第二部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part2)

关于ReactiveCocoa RACCommamd

- [http://blog.csdn.net/womendeaiwoming/article/details/37597779](http://blog.csdn.net/womendeaiwoming/article/details/37597779)

 