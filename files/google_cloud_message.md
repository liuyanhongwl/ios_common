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

1. 启动： 判断Notification开关和本地gcmToken -> ** 注册APNS -> 注册gcm -> 订阅相关topic -> 调用“notificationtokenupload post”接口。 **
2. 启动： 判断注册了gcm token -> 检查4中订阅的topic。（没订阅上的订阅。升级的取消订阅老的topic，订阅新的topic。）
3. 启动： 重新同步未成功的请求（“notificationtokenupload post”接口）。
3. 退出登陆：调用“notificationtokenupload post”接口。
4. 登陆： 调用“notificationtokenupload post”接口。
5. app内关闭推送： 注销APNS -> 清空本地存储的gcm字段。
6. app内开启推送：  -> ** 注册APNS -> 注册gcm -> 订阅相关topic -> 调用“notificationtokenupload post”接口。**
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





