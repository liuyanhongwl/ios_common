## SDWebImage

这个类库提供一个UIImageView类别以支持加载来自网络的远程图片。具有**缓存管理、异步下载**、同一个URL下载次数控制和优化等特征。

### SDWebImage 加载图片流程

1.入口 UIImageView+WebCache 的 sd_setImageWithURL: 方法。

2.进入 UIView+WebCache 的 sd_internalSetImageWithURL: 方法。 设置一些加载视图（activityIndicator）、默认图片（placeholder）、以及图片的显示逻辑。

3.进入 SDWebImageManager 的 loadImageWithURL: 方法。判断url有效性。然后交给 SDImageCache 从缓存查找图片。

4.进入 SDImageCache 的 queryCacheOperationForKey: 方法。先从内存查找图片（NSCache），如果内存中有则回调到 SDWebImageManager。

5.如果内存中没有，则创建 NSOperation，并将从硬盘中查找图片的操作添加到GCD串行队列。将结果回调到 SDWebImageManager。

6.回掉 SDWebImageManager 的 block， 如果从缓存中拿到图片，再回调到 UIView+WebCache 进行显示图片等操作。

7.回掉 SDWebImageManager 的 block，如果没从缓存中拿到图片或者要求强制更新缓存，则需要调用 SDWebImageDownloader 的 downloadImageWithURL: 方法下载图片。

8.进入 SDWebImageDownloader 的 downloadImageWithURL: 方法。根据配置创建 NSMutableURLRequest 和 SDWebImageDownloaderOperation， 然后把 request 和 session（NSURLSession） 交给 operation 处理，把 operation 放入 downloadQueue(NSOperationQueue)。

9.执行下载操作时走 SDWebImageDownloaderOperation 的 start 方法开始下载请求，请求完毕回调从 SDWebImageDownloader 传来的 completedBlock。

10.回调 SDWebImageManager 的 下载完成 block。判断配置，是否需要缓存，需要缓存的通过 SDImageCache 缓存。回调 UIView+WebCache 进行显示图片等操作。

### SDWebImage 库的作用

1. **SDWebImage(WebCache)** : 入口封装，实现读取图片完成后的回调。
2. **SDWebImageManager** : 对图片进行管理的中转站，记录那些图片正在读取。向下层读取Cache(调用SDWebImageCache)，或者向网络读取对象(调用SDWebImageDownloader)。实现SDWebImageCache和SDWebImageDownloader的回调。
3. **SDWebImageCache** : 根据URL的MD5对图片进行存储和读取（实现在内存中或者硬盘上的两种实现），实现图片的内存清理工作。
4. **SDWebImageDownloader** : 根据URL向网络读取数据（实现部分读取和全部读取后的回调）
5. **SDWebImageDecoder** : 图片解码处理。

### 收获

1. 职责分明，结构清晰，将不同任务分给不同的类。
2. 下载使用 NSOperation + NSURLSession， GCD 的 barrier 控制数组的增删改查。
3. 内存缓存使用 NSCache，NSCache 是线程安全的，多线程中使用无需加锁。
4. 硬盘缓存使用 GCD的串行队列进行存取操作，控制 NSFileManager。这里 NSOperation 只是做取消标记用。




