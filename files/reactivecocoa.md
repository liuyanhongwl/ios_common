## ReactiveCocoa

### 参考

关于ReactiveCocoa基本功，BenBeng的文章写得很好，图文并茂得描述了一个小项目的RAC开发流程：

- [ReactiveCocoa入门教程——第一部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part1) 
- [ReactiveCocoa入门教程——第二部分](http://benbeng.leanote.com/post/ReactiveCocoaTutorial-part2)

关于ReactiveCocoa RACCommamd

- [http://blog.csdn.net/womendeaiwoming/article/details/37597779](http://blog.csdn.net/womendeaiwoming/article/details/37597779)

### 编程思想 

[这里](chained.md)

### ReactiveCocoa 编程思想

ReactiveCocoa结合了几种编程风格：

- 函数式编程（Functional Programming）：使用高阶函数，例如函数用其他函数作为参数。
- 响应式编程（Reactive Programming）：关注于数据流和变化传播。
 
所以，你可能听说过ReactiveCocoa被描述为函数响应式编程（FRP）框架。

比喻：

>可以把信号想象成水龙头，只不过里面不是水，而是玻璃球(value)，直径跟水管的内径一样，这样就能保证玻璃球是依次排列，不会出现并排的情况(数据都是线性处理的，不会出现并发情况)。水龙头的开关默认是关的，除非有了接收方(subscriber)，才会打开。这样只要有新的玻璃球进来，就会自动传送给接收方。可以在水龙头上加一个过滤嘴(filter)，不符合的不让通过，也可以加一个改动装置，把球改变成符合自己的需求(map)。也可以把多个水龙头合并成一个新的水龙头(combineLatest:reduce:)，这样只要其中的一个水龙头有玻璃球出来，这个新合并的水龙头就会得到这个球。

### 内存管理

为了支持这种模型，ReactiveCocoa自己持有全局的所有信号。如果一个signal有一个或多个订阅者，那这个signal就是活跃的。如果所有的订阅者都被移除了，那这个信号就能被销毁了。

上面说的就引出了最后一个问题：如何取消订阅一个signal？在一个completed或者error事件之后，订阅会自动移除（马上就会讲到）。你还可以通过RACDisposable 手动移除订阅。

### RACCommand

### RAC 手势

 