### UITableView性能优化


[一次TableView性能优化经历](http://www.cocoachina.com/cms/wap.php?action=article&id=13212)

1. 判断是否有重用
2. 用Charles分析是否正确进行网络请求
3. 跑下Instruments三件套(Time Profiler,Core Animation,GPU Driver)
	1. 看下是不是GPU的问题 (Renderer Utilization,Tiler Utilization)
	2. CPU (例如：tableView:cellForRowAtIndexPath: 中的CPU时间总利用率只有~28%，非常低。于是建议是CPU/IO并不是真正的限制因素)
	3. 在cell中反复创建View是不可取的，一般在初始化的时候全部创建，然后控制显示和隐藏。（另外还有圆角，[参考：小心别让圆角成了你列表的帧数杀手](http://supermao.cn/xiao-xin-bie-rang-yuan-jiao-cheng-liao-ni-lie-biao-de-zheng-shu-sha-shou/)）
	