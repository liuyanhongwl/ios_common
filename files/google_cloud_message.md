## google cloud message

### problem

** 1. iOS app client端显示获取token，但是server发送notification时，失败（NotRegistered）。**

解决： iOS Development Provisioning Profile 不是正确的。

** 2. gcm not send to apn when iOS app was killed. **

解决： fix it by adding "priority": "high" to push message:
 	{ 
 		"registration_ids": ["nYG2y3HGY3...TSR-i3KWNqI"], 
   		"notification": { "title": "Hello, world!", "body": "a", "badge": 3, "sound": "default" }, "priority": "high" 
   	} 
   	

### 调研结果：
1. iOS 可接受gcm类型是notification的通知。

2. 调整发送的参数设置成可发送通知给killed状态和background状态的app。

规则：

1. 定义推送数据格式。
2. 实现推送跳转规则。
3. 订阅topic规则。

操作：

1. 启动： 判断Notification开关和本地gcmToken -> 注册APNS -> 注册gcm -> 订阅相关topic -> 调用“notificationtokenupload post”接口。
2. 启动： 判断注册了gcm token -> 检查4中订阅的topic。（没订阅上的订阅。升级的取消订阅老的topic，订阅新的topic。）
3. 退出登陆：调用“notificationtokenupload post”接口。
4. 登陆： 调用“notificationtokenupload post”接口。
5. app内关闭推送： 注销APNS -> 清空本地存储的gcm字段。
6. app内开启推送：  -> 注册APNS -> 注册gcm -> 订阅相关topic -> 调用“notificationtokenupload post”接口。
1. gcm token 刷新：获取新的gcm token -> 订阅相关topic -> 调用“notificationtokenupload post”接口。

服务器接口：

7. 上传token接口(notificationtokenupload post)
8. 删除token接口(notificationtokenupload delete) ： 暂时未用。

本地存储：

1. gcm token ： 有就不再注册gcm， 只监听刷新。（与是否注册gcm同步）
2. notification 开关。(与是否注册apns同步)
2. 4种订阅的topic : gcmTopicCommon/gcmTopicIOS/gcmTopicVersion/gcmTopicTimeZone。

待完成：

1. 注销老版本通知。
2. 增加新通知的GA。
3. 与服务器同步失败处理。
4. 通知跳转。（含有重构play)


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