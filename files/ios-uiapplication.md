## UIApplication

利用UIApplication对象,能进行一些应用级别的操作.可以设置应用程序图标右上角的红色提醒数字设置联网指示器的可见性可以设置应用程序的状态栏，进行应用之间的跳转.

### UIApplication功能

#### 1. 设置应用提醒数字

```
UIApplication *ap = [UIApplication sharedApplication];
ap.applicationIconBadgeNumber = 20;
```

#### 2. 设置连网状态

```
ap.networkActivityIndicatorVisible = YES;

```

#### 3. 设置状态栏

- 应用程序的状态栏,默认是交给控制器来管理的.

控制器提供的方法,可以直接重写这个方法在控制器当中设置状态栏样式

```
- (UIStatusBarStyle)preferredStatusBarStyle {
return UIStatusBarStyleLightContent;
}

```

- 隐藏状态栏,通过控制器的方式.同样实现方法:返回NO时为不隐藏返回YES时为隐藏

```
- (BOOL)prefersStatusBarHidden {
return NO;
}
```

- 通过UIApplication来管理状态. 

通常在开发当中都是应用程序来管理状态栏的.来做统一管理,不然的话, 会有很多个控制器. 会非常的麻烦.

想要让应用程序管理状态栏,要在info.plist当中进行配置,在最后一个添加一个key值:View controller-based status bar appearance设置为NO.就是应用程序来管理了. 并且控制器管理会无效

```
UIApplication *ap = [UIApplication sharedApplication];
ap.statusBarStyle = UIStatusBarStyleLightContent;
ap.statusBarHidden = YES;
```

#### 4. 跳转网页

```
UIApplication *ap = [UIApplication sharedApplication];
//URL:协议头://域名 应用程序通过协议头的类型,去打开相应的软件.
NSURL *url =[NSURL URLWithString:@"http://www.baidu.com"];  
[ap openURL:url];

//打电话
[application openURL:[NSURL URLWithString:@"tel://10086"]];
//发短信
[app openURL:[NSURL URLWithString:@"sms://10086"]];

```

### 应用程序的启动原理

程序启动时执行main函数,在main函数当中有以下操作.

```
int main(int argc, char * argv[]) {
	@autoreleasepool {
		return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	}
}
```

程序完整启动流程

1. 执行Main
2. 执行UIApplicationMain函数.
3. 创建UIApplication对象,并设置UIApplicationMain对象的代理.UIApplication的第三个参数就是UIApplication的名称,如果指定为nil,它会默认 为UIApplication.UIApplication的第四个参数为UIApplication的代理.
4. 开启一个主运行循环.保证应用程序不退出.
5. 加载info.plist.加载配置文文件.判断一下info.plist文件当中有没有Main storyboard file base name里面有没有指定storyboard文件,如果有就去加载info.plist文件,如果没有,那么应用程序加载完毕.
6. 通知应用程序，调用代理方法

