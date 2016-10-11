## iOS10推送通知进阶(Notification Extension）

- [简介](#简介)
- [UNNotificationServiceExtension - 通知服务扩展](#UNNotificationServiceExtension - 通知服务扩展)
- [UNNotificationContentExtension - 通知内容扩展](#UNNotificationContentExtension - 通知内容扩展)

### 简介

这篇文章主要讲iOS10推送通知的两个扩展框架：**UNNotificationServiceExtension（通知服务扩展）** 和 **UNNotificationContentExtension（通知内容扩展）**。

<img src="../images/ios10_usernotification_extension/xcode-unnotification-extension.jpeg">

- UNNotificationServiceExtension（通知服务扩展）是在收到通知后，展示通知前，做一些事情的。比如，增加附件，网络请求等。
- 想要给通知创建一个自定义的用户界面，需要 UNNotificationContentExtension（通知内容扩展）。


### UNNotificationServiceExtension - 通知服务扩展

如果经常使用iMessage的朋友们，就会经常收到一些信息，附带了一些照片或者视频，所以推送中能附带这些多媒体是非常重要的。如果推送中包含了这些多媒体信息，可以使用户不用打开app，不用下载就可以快速浏览到内容。众所周知，推送通知中带了push payload，即使z去年苹果已经把payload的size提升到了4k bites，但是这么小的容量也无法使用户能发送一张高清的图片，甚至把这张图的缩略图包含在推送通知里面，也不一定放的下去。在iOS X中，我们可以使用新特性来解决这个问题。我们可以通过新的service extensions来解决这个问题。

iOS10给通知添加附件有两种情况：本地通知和远程通知。

1. 本地推送通知，只需给content.attachments设置UNNotificationAttachment附件对象
2. 远程推送通知，需要实现 UNNotificationServiceExtension（通知服务扩展），在回调方法中处理 推送内容时设置 request.content.attachments（请求内容的附件） 属性，之后调用 contentHandler 方法即可。

UNNotificationServiceExtension 提供在远程推送将要被 push 出来前，处理推送显示内容的机会。此时可以对通知的 request.content 进行内容添加，如添加附件，userInfo 等。下图显示了Notification Service Extension的流程：

<img src="../images/ios10_usernotification_extension/unnotification-service-extension.jpg">

处理的细节如下：

1.为了能在service extension 里面的attachment，必须给apns增加 "mutable-content":1 字段，使你的推送通知是动态可变的。

```
{
     "aps":{
     	 "alert":"Testing.. (34)",
	     "badge":1,
    	 "sound":"default",
	     "mutable-content":1
	  }
}
```

2.给项目新建一个Notification Service Extension的扩展。

3.在-didReceiveNotificationRequest:withContentHandler:方法中处理request.content，用来给通知的内容做修改。如面代码示例了收到通知后，给通知增加图片附件：

```
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];
    
    //1. 下载
    NSURL *url = [NSURL URLWithString:@"http://img1.gtimg.com/sports/pics/hv1/194/44/2136/138904814.jpg"];
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:config];
    NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (!error) {
        
            //2. 保存数据
            NSString *path = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).firstObject
                              stringByAppendingPathComponent:@"download/image.jpg"];
            UIImage *image = [UIImage imageWithData:data];
            NSError *err = nil;
            [UIImageJPEGRepresentation(image, 1) writeToFile:path options:NSAtomicWrite error:&err];
            
            //3. 添加附件
            UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"remote-atta1" URL:[NSURL fileURLWithPath:path] options:nil error:&err];
            if (attachment) {
                self.bestAttemptContent.attachments = @[attachment];
            }
        }
        
        //4. 返回新的通知内容
        self.contentHandler(self.bestAttemptContent);
    }];
    
    [task resume];
}
```

**注意：**使用UNNotificationServiceExtension，你有30秒的时间处理这个通知，可以同步下载图像和视频到本地，然后包装为一个UNNotificationAttachment扔给通知，这样就能展示用服务器获取的图像或者视频了。这里需要注意：如果数据处理失败，超时，extension会报一个崩溃信息，但是通知会用默认的形式展示出来，app不会崩溃。

附件通知所带的附件格式大小都是有限的，并不能做所有事情，视频的前几帧作为一个通知的附件是个不错的选择。
 
### UNNotificationContentExtension - 通知内容扩展

没有交互

推送的组成



改进：
点击actions，更新通知界面。


### 结合使用两个扩展

可以在content extension里面绘制界面时，通过notification.request.content.attachments获取附件放到自定义控件里面。