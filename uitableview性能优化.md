### UITableView性能优化

------
### 一 . [参考： 一次TableView性能优化经历](http://www.cocoachina.com/cms/wap.php?action=article&id=13212)

1. 判断是否有重用
2. 用Charles分析是否正确进行网络请求
3. 跑下Instruments三件套(Time Profiler,Core Animation,GPU Driver)
	1. 看下是不是GPU的问题 (Renderer Utilization,Tiler Utilization)
	2. CPU (例如：tableView:cellForRowAtIndexPath: 中的CPU时间总利用率只有~28%，非常低。于是建议是CPU/IO并不是真正的限制因素)
	3. 在cell中反复创建View是不可取的，一般在初始化的时候全部创建，然后控制显示和隐藏。（另外还有圆角，[参考：小心别让圆角成了你列表的帧数杀手](http://supermao.cn/xiao-xin-bie-rang-yuan-jiao-cheng-liao-ni-lie-biao-de-zheng-shu-sha-shou/)）
	
	
	
#### layer 造成的卡顿
在ios系统中使用layer性对view进行渲染时往往会占用很多系统开销，从而造成UI卡顿的现象。 (`离屏渲染`)   
可以尝试使用如下代码减小系统开销，使界面界面卡顿现象消失。  
   
```
_distanceLabel.layer.shouldRasterize = YES;      
_distanceLabel.layer.rasterizationScale = [UIScreen mainScreen].scale;
```

这样大部分情况下可以马上挽救你的帧数在55帧每秒以上。shouldRasterize = YES会使视图渲染内容被缓存起来，下次绘制的时候可以直接显示缓存，当然要`在视图内容不改变的情况`下。

除了上面非要作死的人外，大家还是采取预先生成圆角图片，并缓存起来这个方法才是比较好的手段。预处理圆角图片可以在后台处理，处理完毕后缓存起来，再在主线程显示，这就避免了不必要的离屏渲染了。

另外也有在图片上面覆盖一个镂空圆形图片的方法可以实现圆形头像效果，这个也是极为高效的方法。缺点就是对视图的背景有要求，单色背景效果就最为理想。



### 二. [参考：UITableVIew 滚动流畅性优化](http://blog.csdn.net/enuola/article/details/41942963)

1. 在代理方法中做了过多的计算占用了UI线程的时间
2. Cell 中 view 的组织复杂，比如使用layer并不会有太大影响，但是如果layer使用了透明，或者圆角、变形等效果，就会影响到绘制速度

#### 1. tableview 的代理方法的调用顺序，和时机

1. 向代理要 number Of Rows。
2. 对于每行向代理要 height For Row At Index Path。
3. 向代理要 当前屏幕可见的 cell For Row At Index Path 。（实测显示4寸屏的手机会取 屏幕显示数量+2，3.5寸屏同4寸屏数量，虽然3.5寸屏可显示的cell 数量要小于 4寸屏！）
4. 然后 cell 就显示出来了。

很多人都把优化的重点放到了 cell for row at indexpath 那个方法里了，在这里尽可能的少计算，但是却忽略了另一个很轻松就能提升加载时间的方法 

##### 1.1 tableView:heightForRowAtIndexPath:

 Table View 在每次 reload data 时都会要所有 cell 的高度！这就是说你有一百行 cell，就像代理要100次每个cell 的高度，而不是当前屏幕显示的cell 的数量的高度！
 （注：iOS7下增加了 **estimatedHeightForRowAtIndexPath:** 方法，对于提升加载 Table View 的速度有非常明显的提高。）
 
 
 但是有人说了，我早听别人说了，reloadData 方法尽量不要调用，我插入新行都用 **insertRowsAtIndexPaths:withRowAnimation: **删除也用 delete 那个，这个总行了吧？！
 

这样也不能忽略 height For Row At Index Path 这个回调的重要性。因为在每次插入或者删除一行后同样需要调用一遍 `所有行` 的这个回调方法！是所有行！你没看错，所有只是简单的减少一个代理方法的计算量，就可以明显的提升加载速度。


对于提升 tableView:heightForRowAtIndexPath: 计算量，就是尽可能的让这个方法的计算复杂度为 O(1)，就是只是简单的从数组中取一个值，然后返回！也许有人又要问了，我的应用都是动态的高度，就像微博那样的，不定数量的文字，可能还有图片，大小也不固定，这些怎么返回固定的高度啊？我指的固定高度不是 row 的高度都一样那种固定，而是让在 tableView:heightForRowAtIndexPath: 这个回调里取这个高度的时间是近乎固定的。


对于高度的计算，还有个小细节需要注意，就是`如果 row 的高度都一定`，那就删除代理中的这个 tableView:heightForRowAtIndexPath: 方法，设置 Table View 的 **rowHeight** 属性，相似的 numberOfRowsInSection: 系列的方法，我就不都写出来了。苹果的文档里介绍这样也可以减少了调用时间。


现在回归正题。对于 cell 高度不固定的，传统的方法是为 cell 写个计算行高的类方法，传入那些动态的元素（文字，图片等），然后返回计算后的高度。在 tableView:heightForRowAtIndexPath: 中调用这个方法，填入需要的参数计算cell 高度。这当然没有什么问题，只是要是计算量很复杂，你每次 reloadData ，光计算行高就要花去 rowCount * 单行高评价计算时间，想想有100行，你不定期的需要 reloadData 或者 insert(delete) row。。。。解决办法就是

* 用 “空间换时间”

将计算行高的时间提前到从服务器搂回数据的时候，计算完了高度一并写回数据库，别告诉我你在主线程里阻塞式的处理网络请求。。。。面壁思过去吧，别浪费了 GCD，NSOperationQueue的青春。最先想到的还是 NSThread 的同学，证明你已经老了。。。现在几乎大部分的多线程操作都不需要用到 NSThread 和 runloop了。

##### 1.2 tableView:cellForRowAtIndexPath:

说完了计算 cell 行高的优化，现在来谈 tableView:cellForRowAtIndexPath: 回调的优化。优化思路同上，也是通过预处理减少在这个回调中的计算时间。这个回调重点谈的是对图片异步加载的优化。

图片异步加载无非就是在这个方法里发起异步请求，图片加载完后根据 UIImageView 的引用设置图片。有经验的程序员可能会使用 懒加载 的方式减少快速滑动时因为网络请求过于频繁与切换线程显示图片造成的卡顿。这里还有个问题，拿回来的图片一定和最后显示的大小不一样，有时候偷懒，直接设置 image view 的 contentMode 属性要 image view 自己 压缩。这是一个很取巧的方法，但是对 table view 的滚动速度也会造成 不容忽视 的影响。对图片变形需要对图片做 transform ，每次压缩图片都要对图片乘以一个变换矩阵，如果你的图片很多，这个计算量是不同忽视的。

优化建议：从网络搂回来图片后先根据需要显示的图片大小切成合适大小的图，每次只显示处理过大小的图片，当查看大图时在显示大图。如果服务器能直接返回预处理好的小图和图片的大小更好。


##### 1.3 手动Drawing视图提升流畅性

手动绘制方法，不是直接子类化 UITableViewCell，然后覆盖 drawRect: 方法，因为 cell 中不是只有一个 content view。如果不了解 cell 的层次结构，可以用 Reveal 去看下。

绘制 cell 不建议使用 UIView，建议使用 CALayer。 UIView 的绘制是建立在 CoreGraphic 上的，使用的是 CPU。CALayer 使用的是 Core Animation，CPU，GPU 通吃，由系统决定使用哪个。View的绘制使用的是自下向上的一层一层的绘制，然后渲染。Layer处理的是 Texure，利用 GPU 的 Texture Cache 和独立的浮点数计算单元加速 纹理 的处理。

问题已解：在 UIView 的 drawRect 里使用 CG 开头的绘图命令只是触发的伪离屏渲染，绘图还是靠 CPU 同步的在主线程绘制，在 layer 中触发的离屏渲染会触发真正的离屏渲染，在一个独立的进程里绘制。


