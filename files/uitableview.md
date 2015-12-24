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