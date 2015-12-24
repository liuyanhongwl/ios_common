## UILabel的高度

问题 : 使用 `boundingRectWithSize` 得不到正确的高度。

解决 : 使用一个`UILabel`的`sizeThatFits`方法来间接计算高度

```

UILabel *gettingSizeLabel = [[UILabel alloc] init];w
gettingSizeLabel.font = [UIFont fontWithName:@"YOUR FONT's NAME" size:16];
gettingSizeLabel.text = @"YOUR LABEL's TEXT";
gettingSizeLabel.numberOfLines = 0;
gettingSizeLabel.lineBreakMode = NSLineBreakByWordWrapping;
CGSize maximumLabelSize = CGSizeMake(310, 9999);

CGSize expectSize = [gettingSizeLabel sizeThatFits:maximumLabelSize];


```

