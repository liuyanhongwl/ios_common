## Core Animation

【参考】[ios核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/index.html)

### 介绍
>Core Animation其实是一个令人误解的命名。你可能认为它只是用来做动画的，但实际上它是从一个叫做Layer Kit这么一个不怎么和动画有关的名字演变而来，所以做动画这只是Core Animation特性的冰山一角。

>Core Animation是一个复合引擎，它的职责就是尽可能快地组合屏幕上不同的可视内容，这个内容是被分解成独立的图层，存储在一个叫做图层树的体系之中。于是这个树形成了UIKit以及在iOS应用程序当中你所能在屏幕上看见的一切的基础。


### 1. 图层树

#### 但是为什么iOS要基于UIView和CALayer提供两个平行的层级关系呢？为什么不用一个简单的层级来处理所有事情呢？

>原因在于要做职责分离，这样也能避免很多重复代码。在iOS和Mac OS两个平台上，事件和用户交互有很多地方的不同，基于多点触控的用户界面和基于鼠标键盘有着本质的区别，这就是为什么iOS有UIKit和UIView，但是Mac OS有AppKit和NSView的原因。他们功能上很相似，但是在实现上有着显著的区别。

### 2. 寄宿图

#### 2.1. contents属性。

`contents`

`contentsGravity` (UIView : contentMode)

`contentsScale` (UIView : contentScaleFactor) 

如果contentsScale设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片，这就是我们熟知的Retina屏幕。（如果你对像素和点的概念不是很清楚的话，这个章节的后面部分将会对此做出解释）。

`masksToBounds` (UIView : clipsToBounds)

`contentsRect`

和bounds，frame不同，contentsRect不是按点来计算的，它使用了单位坐标，单位坐标指定在0到1之间，是一个相对值（像素和点就是绝对值）。所以他们是相对与寄宿图的尺寸的。iOS使用了以下的坐标系统：

- 点 —— 在iOS和Mac OS中最常见的坐标体系。点就像是虚拟的像素，也被称作逻辑像素。在标准设备上，一个点就是一个像素，但是在Retina设备上，一个点等于2*2个像素。iOS用点作为屏幕的坐标测算体系就是为了在Retina设备和普通设备上能有一致的视觉效果。
- 像素 —— 物理像素坐标并不会用来屏幕布局，但是仍然与图片有相对关系。UIImage是一个屏幕分辨率解决方案，所以指定点来度量大小。但是一些底层的图片表示如CGImage就会使用像素，所以你要清楚在Retina设备和普通设备上，他们表现出来了不同的大小。
- 单位 —— 对于与图片大小或是图层边界相关的显示，单位坐标是一个方便的度量方式， 当大小改变的时候，也不需要再次调整。单位坐标在OpenGL这种纹理坐标系统中用得很多，Core Animation中也用到了单位坐标。

`contentsCenter` (UIView : resizableImageWithCapInsets:)

#### 2.2. Custom Drawing

也可以直接用Core Graphics直接绘制寄宿图。

-drawRect: 方法没有默认的实现，因为对UIView来说，寄宿图并不是必须的，它不在意那到底是单调的颜色还是有一个图片的实例。如果UIView检测到-drawRect: 方法被调用了，它就会为视图分配一个寄宿图，这个寄宿图的像素尺寸等于视图大小乘以 contentsScale的值。

如果你不需要寄宿图，那就不要创建这个方法了，这会造成CPU资源和内存的浪费，这也是为什么苹果建议：如果没有自定义绘制的任务就不要在子类中写一个空的-drawRect:方法。

UIView 的 -drawRect: 方法， 底层是CALayer安排(CALayerDelegate)了重绘工作和保存了因此产生的图片。

CALayerDelegate ： 

当需要被重绘时，CALayer会请求它的代理给他一个寄宿图来显示。它通过调用下面这个方法做到的:

```
(void)displayLayer:(CALayerCALayer *)layer;
```

可以在这个方法里直接设置contents属性。如果代理不实现-displayLayer:方法，CALayer就会转而尝试调用下面这个方法：

```
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```

在调用这个方法之前，CALayer创建了一个合适尺寸的空寄宿图（尺寸由bounds和contentsScale决定）和一个Core Graphics的绘制上下文环境，为绘制寄宿图做准备，他作为ctx参数传入。

UIView的图层的delegate是UIView自己。

### 3. 图层几何学

#### 3.1. 布局

UIView : `frame` `bounds` `center`。

相当于

CALayer : `frame` `bounds` `position`。

对于视图或者图层来说，frame并不是一个非常清晰的属性，它其实是一个虚拟属性，是根据bounds，position和transform计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值。

记住当对图层做变换的时候，比如旋转或者缩放，frame实际上代表了覆盖在图层旋转之后的整个轴对齐的矩形区域，也就是说frame的宽高可能和bounds的宽高不再一致了。

![](../images/CoreAnimation/3.2.jpeg)

#### 3.2. 锚点 (anchorPoint)

之前提到过，视图的center属性和图层的position属性都指定了anchorPoint相对于父图层的位置。图层的anchorPoint通过position来控制它的frame的位置，你可以认为anchorPoint是用来移动图层的把柄。

默认来说，anchorPoint位于图层的中点，所以图层的将会以这个点为中心放置。anchorPoint属性并没有被UIView接口暴露出来，这也是视图的position属性被叫做“center”的原因。但是图层的anchorPoint可以被移动，比如你可以把它置于图层frame的左上角，于是图层的内容将会向右下角的position方向移动（图3.3），而不是居中了。

![](../images/CoreAnimation/3.3.jpeg)

#### 3.3. 坐标系

CALayer给不同坐标系之间的图层转换提供了一些工具类方法：

```
- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer;
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;
```

`geometryFlipped`

`Z坐标轴` 和UIView严格的二维坐标系不同，CALayer存在于一个三维空间当中。除了我们已经讨论过的position和anchorPoint属性之外，CALayer还有另外两个属性，`zPosition`和`anchorPointZ`

#### 3.4. Hit Testing

CALayer并不关心任何响应链事件，所以不能直接处理触摸事件或者手势。但是它有一系列的方法帮你处理事件：`-containsPoint:`和`-hitTest:`。

- `-containsPoint:` 接受一个在本图层坐标系下的CGPoint，如果这个点在图层frame范围内就返回YES。
- `-hitTest:` 方法同样接受一个CGPoint类型参数，而返回值不是BOOL类型，它返回图层本身，或者包含这个坐标点的叶子节点图层。这意味着不再需要像使用`-containsPoint:`那样，人工地在每个子图层变换或者测试点击的坐标。如果这个点在最外面图层的范围之外，则返回nil。

#### 3.5. 自动布局

当使用视图的时候，可以充分利用UIView类接口暴露出来的UIViewAutoresizingMask和NSLayoutConstraintAPI，但如果想随意控制CALayer的布局，就需要手工操作。最简单的方法就是使用CALayerDelegate如下函数：

```
- (void)layoutSublayersOfLayer:(CALayer *)layer;
```
当图层的bounds发生改变，或者图层的-setNeedsLayout方法被调用的时候，这个函数将会被执行。这使得你可以手动地重新摆放或者重新调整子图层的大小，但是不能像UIView的autoresizingMask和constraints属性做到自适应屏幕旋转。

这也是为什么最好使用视图而不是单独的图层来构建应用程序的另一个重要原因之一。

### 4. 视觉效果

#### 4.1. 圆角

`conrnerRadius` : 属性控制着图层角的曲率。它是一个浮点数，默认为0（为0的时候就是直角），但是你可以把它设置成任意值。默认情况下，这个曲率值只影响背景颜色而不影响背景图片或是子图层。不过，如果把 `masksToBounds` 设置成YES的话，图层里面的所有东西都会被截取。

#### 4.2. 图层边框

`borderWidth`和`borderColor`。二者共同定义了图层边的绘制样式。这条线（也被称作stroke）沿着图层的bounds绘制，同时也包含图层的角。

 边框是绘制在图层边界里面的，而且在所有子内容之前，也在子图层之前。
 
 ![](../images/CoreAnimation/4.3.png)

#### 4.3 阴影

`shadowOpacity`

`shadowColor` : 属性控制着阴影的颜色

`shadowOffset` : 属性控制着阴影的方向和距离

`shadowRadius` : 控制着阴影的模糊度

- 和图层边框不同，图层的阴影继承自内容的外形，而不是根据边界和角半径来确定。
- maskToBounds属性裁剪掉了阴影和内容 （如果你想解决，需要用到两个图层：一个只画阴影的空的外图层，和一个用masksToBounds裁剪内容的内图层。）

`shadowPath` :   如果你事先知道你的阴影形状会是什么样子的，你可以通过指定一个shadowPath来提高性能。shadowPath是一个CGPathRef类型（一个指向CGPath的指针）。CGPath是一个Core Graphics对象，用来指定任意的一个矢量图形。我们可以通过这个属性单独于图层形状之外指定阴影的形状。

#### 4.4. 图层蒙板

`mask` : mask 图层定义了父图层的部分可见区域

#### 4.5. 拉伸过滤

`minificationFilter`

`magnificationFilter`

三种拉伸过滤方法: 

- `kCAFilterLinear` (双线性滤波算法) : minification（缩小图片）和magnification（放大图片）默认的过滤器都是kCAFilterLinear。
- `kCAFilterNearest` (最近过滤) : 就是取样最近的单像素点而不管其他的颜色。
- `kCAFilterTrilinear` (三线性滤波算法) : kCAFilterTrilinear和kCAFilterLinear非常相似，大部分情况下二者都看不出来有什么差别。但是，较双线性滤波算法而言，三线性滤波算法存储了多个大小情况下的图片（也叫多重贴图），并三维取样，同时结合大图和小图的存储进而得到最后的结果。这个方法的好处在于算法能够从一系列已经接近于最终大小的图片中得到想要的结果，也就是说不要对很多像素同步取样。这不仅提高了性能，也避免了小概率因舍入错误引起的取样失灵的问题。

>总的来说，对于比较小的图或者是差异特别明显，极少斜线的大图，最近过滤算法会保留这种差异明显的特质以呈现更好的结果。但是对于大多数的图尤其是有很多斜线或是曲线轮廓的图片来说，最近过滤算法会导致更差的结果。**换句话说，线性过滤保留了形状，最近过滤则保留了像素的差异**。

#### 4.6. 组透明

`opacity` (UIView : alpha) : 这两个属性都是影响子层级的。

**给父视图直接使用透明度，容易造成透明度的混合叠加。** 当你显示一个50%透明度的图层时，图层的每个像素都会一半显示自己的颜色，另一半显示图层下面的颜色。这是正常的透明度的表现。但是如果图层包含一个同样显示50%透明的子图层时，你所看到的视图，50%来自子视图，25%来了图层本身的颜色，另外的25%则来自背景色。

**方案一**： 

理想状况下，当你设置了一个图层的透明度，你希望它包含的整个图层树像一个整体一样的透明效果。你可以通过设置Info.plist文件中的`UIViewGroupOpacity`为YES来达到这个效果，但是这个设置会影响到这个应用，整个app可能会受到不良影响。如果UIViewGroupOpacity并未设置，iOS 6和以前的版本会默认为NO（也许以后的版本会有一些改变）。

**方案二**：

可以设置CALayer的一个叫做`shouldRasterize`属性来实现组透明的效果，如果它被设置为YES，在应用透明度之前，图层及其子图层都会被整合成一个整体的图片，这样就没有透明度混合的问题了.

如果你使用了`shouldRasterize`属性，你就要确保你设置了`rasterizationScale`属性去匹配屏幕，以防止出现Retina屏幕像素化的问题

**当shouldRasterize和UIViewGroupOpacity一起的时候，性能问题就出现了**

### 5. 变换

#### 5.1. 仿射变换

`CGAffineTransform` 是一个可以和二维空间向量（例如CGPoint）做乘法的3X2的矩阵。

![](../images/CoreAnimation/5.1.jpeg)

创建一个CGAffineTransform

```
CGAffineTransformMakeRotation(CGFloat angle)
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy)
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty)
```

混合变换 : Core Graphics提供了一系列的函数可以在一个变换的基础上做更深层次的变换

```
CGAffineTransformRotate(CGAffineTransform t, CGFloat angle)
CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy)
CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)
```

**按顺序做了变换，上一个变换的结果将会影响之后的变换，这意味着变换的顺序会影响最终的结果，也就是说旋转之后的平移和平移之后的旋转结果可能不同。**

#### 5.2. 3D变换

`CATransform3D` 和CGAffineTransform类似，CATransform3D也是一个矩阵，但是和2x3的矩阵不同，CATransform3D是一个可以在3维空间内做变换的4x4的矩阵。

`transform` 属性（CATransform3D类型）可以真正做到这点，即让图层在3D空间内移动或者旋转。

![](../images/CoreAnimation/5.6.jpeg)

注意2D的`CGAffineTransform`和3D的`CATransform3D`,分别属于两个不同的框架，Core Graphics和Core Animation。

X，Y，Z轴，以及围绕它们旋转的方向， 遵循左手定则（左手握拳，大拇指朝外，大拇指的方向是轴的方向，四指的方向是旋转的方向）

**透视投影**

>直接使用3D仿射变换，并不会出现我们现实世界中，越远的物体，视角的原因看起来会变小。是因为它默认我们的视角是平行的。**CATransform3D的透视效果通过一个矩阵中一个很简单的元素来控制：`m34`**。m34用于按比例缩放X和Y的值来计算到底要离视角多远。

>m34的默认值是0，我们可以通过设置m34为-1.0 / d来应用透视效果，d代表了想象中视角相机和屏幕之间的距离，以像素为单位，通常500-1000就已经很好了，减少距离的值会增强透视效果，所以一个非常微小的值会让它看起来更加失真，然而一个非常大的值会让它基本失去透视效果。

**灭点**

>当在透视角度绘图的时候，远离相机视角的物体将会变小变远，当远离到一个极限距离，它们可能就缩成了一个点，于是所有的物体最后都汇聚消失在同一个点。

>在现实中，这个点通常是视图的中心，于是为了在应用中创建拟真效果的透视，这个点应该聚在屏幕中点，或者至少是包含所有3D对象的视图中点。

>Core Animation定义了这个点位于变换图层的`anchorPoint`。这就是说，当图层发生变换时，这个点永远位于图层变换之前anchorPoint的位置。

>当改变一个图层的position，你也改变了它的灭点，做3D变换的时候要时刻记住这一点，当你视图通过调整m34来让它更加有3D效果，应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置（而不是直接改变它的position），这样所有的3D图层都共享一个灭点。

**sublayerTransform属性**

>CALayer有一个属性叫做sublayerTransform。它也是CATransform3D类型，但和对一个图层的变换不同，它影响到所有的子图层。这意味着你可以一次性对包含这些图层的容器做变换，于是所有的子图层都自动继承了这个变换方法。

>相较而言，通过在一个地方设置透视变换会很方便，同时它会带来另一个显著的优势：灭点被设置在容器图层的中点，从而不需要再对子图层分别设置了。这意味着你可以随意使用position和frame来放置子图层，而不需要把它们放置在屏幕中点，然后为了保证统一的灭点用变换来做平移。

**背面**

>`doubleSided` CALayer有一个叫做doubleSided的属性来控制图层的背面是否要被绘制。这是一个BOOL类型，默认为YES，如果设置为NO，那么当图层正面从相机视角消失的时候，它将不会被绘制。

**扁平化图层**

>不能够使用图层树去创建一个3D结构的层级关系--在相同场景下的任何3D表面必须和同样的图层保持一致，这是因为每个的父视图都把它的子视图扁平化了。CALayer有一个叫做CATransformLayer的子类来解决这个问题。

#### 5.3. 固体对象

创建一个有光和影的固体对象。

**点击事件**

>点击事件的处理由视图在父视图中的顺序决定的，并不是3D空间中的Z轴顺序。

>你也许认为把doubleSided设置成NO可以解决这个问题，因为它不再渲染视图后面的内容，但实际上并不起作用。因为背对相机而隐藏的视图仍然会响应点击事件（这和通过设置hidden属性或者设置alpha为0而隐藏的视图不同，那两种方式将不会响应事件）。所以即使禁止了双面渲染仍然不能解决这个问题（虽然由于性能问题，还是需要把它设置成NO）。

### 6. 专用图层

#### 6.1 CAShapeLayer


