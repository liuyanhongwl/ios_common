##Local/Remote Notification


### 处理接收到远程通知消息（会回调以下方法中的某一个）

#### application: didFinishLaunchingWithOptions:
此方法在程序第一次启动是调用，也就是说App从Terminate状态进入Foreground状态的时候，根据方法内代码判断是否有推送消息。

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    //  userInfo为收到远程通知的内容
    NSDictionary *userInfo = launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey];
    if (userInfo) {
         // 有推送的消息，处理推送的消息
    }
    return YES；
}
```

#### application: didReceiveRemoteNotification:
如果App处于Background状态时，只用用户点击了通知消息时才会调用该方法；如果App处于Foreground状态，会直接调用该方法


#### application: didReceiveRemoteNotification: fetchCompletionHandler:
此方法不论App处于Foreground状态还是处于Background状态，收到远程推送消息的时候都会立即调用此方法。此方法需要配置后台模式并且在推送负载中必须有content-available此key值，对应的value值为1


>iOS7之前苹果是不支持多任务的，这也是iOS系统对硬件要求低，流畅性好的原因之一。iOS7之后，苹果开始支持多任务，即App可在后台做一些更新UI、下载数据的操作等。若要接收到远程推送的时候要在后台做一些事情则需要把后台远程推送模式打开。不适配iOS7之前系统的项目建议使用此后台模式，充分利用苹果推出的多任务模式，不枉费苹果的一片苦心啊！设置后台模式方法是在项目的Background Modes中增加Remote notifications。


```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo 
fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {

    // 在此方法中一定要调用completionHandler这个回调，告诉系统是否处理成功

    UIBackgroundFetchResultNewData, // 成功接收到数据
    UIBackgroundFetchResultNoData,  // 没有接收到数据
    UIBackgroundFetchResultFailed   // 接受失败
    if (userInfo) {
        completionHandler(UIBackgroundFetchResultNewData);
    } else {
        completionHandler(UIBackgroundFetchResultNoData);
    }
}
```

### 可操作通知类型收到推送消息时回调方法

```

// 此两个回调方法对应可操作通知类型，具体使用方法参考以上方法很容易理解，不在详细叙述
- (void)application:(UIApplication *)application handleActionWithIdentifier:(nullable NSString *)identifier 
forRemoteNotification:(NSDictionary *)userInfo 
completionHandler:(void(^)())completionHandler {

}

- (void)application:(UIApplication *)application handleActionWithIdentifier:(nullable NSString *)identifier 
forRemoteNotification:(NSDictionary *)userInfo withResponseInfo:(NSDictionary *)responseInfo 
completionHandler:(void(^)())completionHandler {

}
```


### 用户手动滑kill掉应用时

>Also keep in mind that if you kill your app from the app switcher (i.e. swiping up to kill the app) then the OS will never relaunch the app regardless of push notification or background fetch. In this case the user has to manually relaunch the app once and then from that point forward the background activities will be invoked.

That post was by an Apple employee so I think i can trust that this information is correct.

So it looks like when the app is killed from the app switcher (by swiping up), the app will never be launched, even for scheduled background fetches.