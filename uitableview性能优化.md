### UITableView性能优化

------
#### [参考： 一次TableView性能优化经历](http://www.cocoachina.com/cms/wap.php?action=article&id=13212)

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