## OC获取权限
-----

#### iOS7 麦克风权限

```
AVAudioSession *avSession = [AVAudioSession sharedInstance];

if ([avSession respondsToSelector:@selector(requestRecordPermission:)]) {

   [avSession requestRecordPermission:^(BOOL available) {

       if (available) {
          //有权限
           //completionHandler
       }
       else
       {
          //无权限
           dispatch_async(dispatch_get_main_queue(), ^{
               [[[UIAlertView alloc] initWithTitle:@"无法录音" message:@"请在“设置-隐私-麦克风”选项中允许xx访问你的麦克风" delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil] show];
           });
       }
   }];
   
}
```

#### iOS7 以上 相机的权限

```
/// 是否有相机的权限
+(BOOL)isAuthedCamera
{
    AVAuthorizationStatus authStatus = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (authStatus == AVAuthorizationStatusDenied || authStatus == AVAuthorizationStatusRestricted) {
        [XDTools showTips:[LDLocalizedString localizedStringWithKey:@"noAuthCamera"] toView:[XDTools appDelegate].window];
        return NO;
    }
    return YES;
}
```