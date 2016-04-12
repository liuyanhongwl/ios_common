## UITableView

### UITableView Section Header Height

1. 问题tableview的style设置成group后，有默认的section header 和 footer 的高度

```
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
    return 0;
}

```
如上代码设置header的高度并不生效， 需要如下方式设置

```
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
    return CGFLOAT_MIN;
}
```


### 隐藏GroupedTableView上边多余的间隔


研究发现，这里其实是一个被 UITableView 默认填充的 HeaderView。而且，当试图将它的高度设置为 0 时，完全不起效果。但我们用下面的代码创建一个高度特别小的 HeaderView 时，上面的边距就不见了：

```
tableView.tableHeaderView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 0, CGFLOAT_MIN)];

```

CGFLOAT_MIN 这个宏表示 CGFloat 能代表的最接近 0 的浮点数，64 位下大概是 0.00(300左右个)0225 这个样子
这样写单纯的为了避免一个魔法数字，这里用 0.1 效果是一样的，后面再讲。