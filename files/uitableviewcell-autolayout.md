## UITableView利用AutoLayout自动调整高度

### 参考
- [使用autolayout自定义动态高度的cell](http://www.jianshu.com/p/f2ef93a3dbb5)

### 要点

- 使用AutoLayout对Cell进行布局。
- iOS8以上只需如下设置：

```
self.tableView.estimatedRowHeight = 54;
self.tableView.rowHeight = UITableViewAutomaticDimension;
```

或者通过实现代理方法：

```
-(CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath{
    return UITableViewAutomaticDimension;
}
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    return UITableViewAutomaticDimension;
}
```

- iOS7需要获取约束算好的高度

```
//如果要支持iOS7这个方法必须实现
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    self.cell = (MyTableViewCell *)[tableView dequeueReusableCellWithIdentifier:@"cell"];

    MyTableViewCell *cell = self.cell;
    cell.text = [self.tableData objectAtIndex:indexPath.row];

    //这句代码必须要有，也就是说必须要设定contentView的宽度约束。
    //设置以后，contentView里面的内容才知道什么时候该换行了
    CGFloat contentViewWidth = CGRectGetWidth(self.tableView.frame);
    [cell.contentView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.width.equalTo(@(contentViewWidth));
    }];

    //重新加载约束,每次计算之前一定要重新确认一下约束
    [cell setNeedsUpdateConstraints];
    [cell updateConstraintsIfNeeded];

    //自动算高度，+1的原因是因为contentView的高度要比cell的高度小1
    CGFloat height = [cell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height + 1;

    return height;
}
```

### 问题

**问题：**给cell内子视图设置宽高比，以支撑cell高度，报警告
 
使用约束控制cell高度，如果cell内有子视图，以子视图固定宽高比确定cell的高度，比如：

```
[self.bannerView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(weakSelf.contentView);
    make.width.mas_equalTo(weakSelf.contentView.mas_height).multipliedBy(su_bannerWidthHeight);
}];
```
报警告，显示宽高比这个约束和其他约束冲突（宽高比算出来的高度、宽度是整数时不会有冲突）。

**解决：**给宽高比那条约束设个优先级中级，并给子视图的抵抗压缩的优先级为低级。

```
[self.bannerView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(weakSelf.contentView);
    //设置宽高比优先级为
    make.width.mas_equalTo(weakSelf.contentView.mas_height).multipliedBy(su_bannerWidthHeight).priorityMedium();
}];
        
[self.bannerView setContentCompressionResistancePriority:UILayoutPriorityDefaultLow forAxis:UILayoutConstraintAxisVertical];
[self.bannerView setContentCompressionResistancePriority:UILayoutPriorityDefaultLow forAxis:UILayoutConstraintAxisHorizontal];
```