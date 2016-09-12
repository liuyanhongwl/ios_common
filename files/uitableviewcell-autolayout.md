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