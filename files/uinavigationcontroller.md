## UINavigationController

### 全局自定义BackButton

#### 1. backBarButtonItem 和 leftBarButtonItem

首先，我们需要清楚 `self.navigationItem.backBarButtonItem` 和 `self.navigationItem.leftBarButtonItem` 的区别。leftBarButtonItem和rightBarButtonItem设置的是本级页面上的BarButtonItem，而backBarButtonItem设置的是下一级页面上的BarButtonItem


使用pushViewController切换到下一个视图时，navigation controller按照以下3条顺序更改导航栏的左侧按钮。

1. 如果B视图有一个自定义的左侧按钮（leftBarButtonItem），则会显示这个自定义按钮；
2. 如果B没有自定义按钮，但是A视图的backBarButtonItem属性有自定义项，则显示这个自定义项；
3. 如果前2条都没有，则默认显示一个后退按钮，后退按钮的标题是A视图的标题。


#### 2. 在全局自定义 BackButton 

```
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:[[UIImage imageNamed:@"su_navibar_back"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [[UIBarButtonItem appearance] setTitleTextAttributes:@{
                                                           NSForegroundColorAttributeName : 
                                                           [UIColor colorWithHexString:SU_COlOR_C4 andAlpha:1]
                                                           } forState:UIControlStateNormal];
    

```

有标注`UI_APPEARANCE_SELECTOR`的方法可以全局设置。

当时上面的方法不能够自定义BackButton 的 title。

#### 3. 全局设置BackButton不显示（有问题的方法）

想要不显示title，可以在全局这么设置

```
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:[[UIImage imageNamed:@"su_navibar_back"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(-1000, -1000) forBarMetrics:UIBarMetricsDefault];
    
```

通过`setBackButtonTitlePositionAdjustment:forBarMetrics:`方法，把title偏移量设置到看不见的位置，实现看不见title的效果。

这种方法看似很简单，但是**会有问题**。当一级页面的title是个比较长的字符串时

```
self.navigationItem.title = @"一级页面一级页面"
```

虽然用上面的方法在二级页面看不见BackButton的title, 但是当二级页面有title（或者显示titleView）时

```
self.navigationItem.title = @"二级页面"
```

二级页面的title(或者titleView)会向右边偏移，就好像左边有个很长的字符串将title(或者titleView)顶到了右边。


#### 4. 全局设置BackButton不显示（妥善的解决方法）

在navigationController的delegate`- (void)navigationController: willShowViewController: animated:
`方法中实现：

```
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    self.navigationBar.topItem.backBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"" style:UIBarButtonItemStylePlain target:nil action:nil];
}

```

上面代码的意思是，在将要push进入二级页面时，设置一级页面的backBarButtonItem。


### iOS返回侧滑手势

#### 问题： 页面有横向滚动的UIScrollView，scrollView的手势覆盖了页面返回手势，使得页面无法手势返回。

**思路**: 我们既然不想同时响应侧滑和 scrollView 的滑动事件，那么我要要做的就是让 scrollView 在侧滑手势判定为失败后再响应滚动事件。



```
if (self.navigationController.interactivePopGestureRecognizer) {
        [self.scrollView.panGestureRecognizer requireGestureRecognizerToFail:self.navigationController.interactivePopGestureRecognizer];
    }
```

### iOS7侧滑手势的Bug

#### 问题： 页面push的时候，瞬间侧滑，则会有两个效果同时生效，造成页面布局混乱。

**解决**： 

1.在UINavigationController Push的时候， 取消侧滑手势。

```
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    self.interactivePopGestureRecognizer.enabled = NO;
    
    [super pushViewController:viewController animated:animated];
}
```

2.Push结束的时候，再打开侧滑手势。注意：若是当前只有rootViewController的话，就关闭侧滑手势。

```
self.interactivePopGestureRecognizer.enabled = YES;
if (navigationController.viewControllers.count == 1) {
	self.interactivePopGestureRecognizer.enabled = NO;
}
```


