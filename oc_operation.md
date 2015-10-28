### OC操作
-----

#### iOS8 openURL 打开设置    
[[MyApplication sharedApplication] openURL:[NSURL URLWithString:UIApplicationOpenSettingsURLString]];


#### viewController中监听Home键触发

NSNotificationCenter通知： 

```
//监听是否触发home键挂起程序.
UIApplicationWillResignActiveNotification
//监听是否重新进入程序程序.
UIApplicationDidBecomeActiveNotification
```

官方例子中是在viewWillAppear的时候添加，viewWillDisappear的时候remove。


#### scrollsToTop 问题

UIScrollView是用来展示滚动的一个类。他有UITableView、UITextView等子类。

scrollsToTop是UIScrollView的一个属性，主要用于点击设备的状态栏时，是scrollsToTop == YES的控件滚动返回至顶部。

每一个默认的UIScrollView的实例，他的scrollsToTop属性默认为YES，所以要实现某一UIScrollView的实例点击设备状态栏返回顶部，则需要`关闭其他的UIScrollView的实例的scrollsToTop属性为NO`。很好理解：若多个scrollView响应返回顶部的事件，系统就不知道到底要将那个scrollView返回顶部了，因此也就不做任何操作了。。。

举个栗子：

只有当一个UIViewController控制器有一个scrollview 并把这个属性设置为yes，
其他的scrollview.scrollsToTop = NO 这样才会响应这个事件，原理很简单，如果有3个scrollview，系统根本不知道你需要哪个滚动到最上面。
        比如一个UIViewController中有三个UIView视图，分别为  _pushList,  _photoList,  _starList，且每个视图中都有一个UITableView，设置如下：
        
```
_pushList.table.scrollsToTop = YES;
_photoList.table.scrollsToTop = NO;
_starList.table.scrollsToTop = NO;
```

明白了吧？需要注意的是UIWebView中含有子视图UIWebViewScrollView，它也是UIScrollView的子类，一开始没有意识到这一点，导致一直实现不了点击状态栏返回顶部，将UIWebViewScrollView的scrollsToTop设为NO，正常了。