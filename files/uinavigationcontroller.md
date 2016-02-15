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

