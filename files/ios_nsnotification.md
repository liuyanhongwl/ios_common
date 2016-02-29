## NSNotification

#### viewController中监听Home键触发

NSNotificationCenter通知： 

```
//监听是否触发home键挂起程序.
UIApplicationWillResignActiveNotification
//监听是否重新进入程序程序.
UIApplicationDidBecomeActiveNotification
```

官方例子中是在viewWillAppear的时候添加，viewWillDisappear的时候remove。



