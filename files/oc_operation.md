### OC操作
-----

#### iOS8 openURL 打开设置    
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:UIApplicationOpenSettingsURLString]];


#### openURL

//    1、调用 自带mail
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"mailto://admin@hzlzh.com"]];
    
//    2、调用 电话phone
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"tel://8008808888"]];
    
//    3、调用 SMS
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"sms://800888"]];
    
//    4、调用自带 浏览器 safari
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@“http://www.hzlzh.com"]];


#### 复制字符串到剪切板

UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
pasteboard.string = _telLabel.text;


