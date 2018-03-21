### Image

#### iOS7以上 设置UIImage的渲染模式

UIImage 的imageWithRenderingMode：方法

- UIImageRenderingModeAutomatic  // 根据图片的使用环境和所处的绘图上下文自动调整渲染模式。
- UIImageRenderingModeAlwaysOriginal   // 始终绘制图片原始状态，不使用Tint Color。
- UIImageRenderingModeAlwaysTemplate   // 始终根据Tint Color绘制图片，忽略图片的颜色信息。



#### 对UIImage裁剪

```
@interface UIImage(UIImageScale)
-(UIImage*)getSubImage:(CGRect)rect;
-(UIImage*)scaleToSize:(CGSize)size;
@end

@implementation UIImage(UIImageScale)

//截取部分图像
-(UIImage*)getSubImage:(CGRect)rect
{
    CGImageRef subImageRef = CGImageCreateWithImageInRect(self.CGImage, rect);
    CGRect smallBounds = CGRectMake(0, 0, CGImageGetWidth(subImageRef), CGImageGetHeight(subImageRef));

    UIGraphicsBeginImageContext(smallBounds.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextDrawImage(context, smallBounds, subImageRef);
    UIImage* smallImage = [UIImage imageWithCGImage:subImageRef];
    UIGraphicsEndImageContext();

    return smallImage;
}

//等比例缩放
-(UIImage*)scaleToSize:(CGSize)size
{
    CGFloat width = CGImageGetWidth(self.CGImage);
    CGFloat height = CGImageGetHeight(self.CGImage);

    float verticalRadio = size.height*1.0/height;
    float horizontalRadio = size.width*1.0/width;

    float radio = 1;
    if(verticalRadio&gt;1 &amp;&amp; horizontalRadio&gt;1)
    {
        radio = verticalRadio &gt; horizontalRadio ? horizontalRadio : verticalRadio;
    }
    else
    {
        radio = verticalRadio &lt; horizontalRadio ? verticalRadio : horizontalRadio;
    }

    width = width*radio;
    height = height*radio;

    int xPos = (size.width - width)/2;
    int yPos = (size.height-height)/2;

    // 创建一个bitmap的context
    // 并把它设置成为当前正在使用的context
    UIGraphicsBeginImageContext(size);

    // 绘制改变大小的图片
    [self drawInRect:CGRectMake(xPos, yPos, width, height)];

    // 从当前context中创建一个改变大小后的图片
    UIImage* scaledImage = UIGraphicsGetImageFromCurrentImageContext();

    // 使当前的context出堆栈
    UIGraphicsEndImageContext();

    // 返回新的改变大小后的图片
    return scaledImage;
}
@end
```

#### 计算图片位置的函数：
1. 计算图片的位置函数    
**AVMakeRectWithAspectRatioInsideRect（）**    
CGRect iamgeAspectRect = AVMakeRectWithAspectRatioInsideRect(image.size, imageView.bounds);    
它可以直接得出一个 image 以任何的比例显示显示在 imageview 中居中所处的位置
需要引入AV框架

2. 如果一个矩形如果做了平移旋转缩放等一系列操作之后，上下左右的四个点（甚至矩形上任意一个点）的位置     
**CGPointApplyAffineTransform()**    
CGPoint originalCenter = CGPointApplyAffineTransform(_mStyleLeftEyeView.center,                                                      CGAffineTransformInvert(_mStyleLeftEyeView.transform));    
CGPoint bottomRight = CGPointApplyAffineTransform(bottomRight, _mStyleLeftEyeView.transform);

