## App Extension Tips

#### 多个target直接，本地bundle内的图片、字体、视频等资源共享。

选中要共享的资源，在xcode右侧的Show the File inspector，在Target Membership里选中要共享的target。

<img src="../images/App-Extension-Tips/inspector.jpeg" width=300>

#### iOS10 Today Extension 里的高度


#### Today Extension 内存警告

对扩展的内存要求限制比较大，iphone 6p Today Extension 内存占用到15M差不多就会内存警告，持续则会被系统杀掉。

如果扩展做的比较复杂，图片是占内存的主要因素，可用如下方法将图片压缩。

```
UIGraphicsBeginImageContextWithOptions(theNewSize, NO, 1.0);
[theImage drawInRect:CGRectMake(0, 0, theNewSize.width, theNewSize.height)];
UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
return newImage;
```

#### Today Extension 最大高度

iOS10以下，最多显示一屏today extension内容的高度。
iOS10以后，有获取当前mode下最大高度的方法，

```
[self.extensionContext widgetMaximumSizeForDisplayMode:NCWidgetDisplayModeExpanded]
```

#### iOS10 Today Extension 标题自动大写