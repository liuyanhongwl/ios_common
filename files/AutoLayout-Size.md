## AutoLayout - Size

### 写在前面

从iOS 6开始推出auto layout，auto layout不止局限于constraint，苹果还从UIView的角度提供了一些辅助方法。

本文涉及的API包括：

- sizeThatFits:和sizeToFit
- systemLayoutSizeFittingSize:
- intrinsicContentSize


### 总结

- **intrinsicContentSize**：它的最主要作用是告诉auto layout system一些信息，可以认为它是后者的回调方法，auto layout system在对view进行布局时会参考这个回调方法的返回值；一般很少像CGSize size = [view intrinsicContentSize]去使用intrinsicContentSize API。
- **sizeThatFits:**和systemLayoutSizeFittingSize非常相似，都是为开发者直接服务的API（而不是回调方法）。所不同的是，sizeThatFits:是auto layout之前就存在的，一般在leaf-level views中用得比较多，在计算size过程中，它可不会考虑constraints神马的；
- **systemLayoutSizeFittingSize**:，它是随着auto layout（iOS 6）引入的，用于在view完成布局前获取size值，如果view的constraints确保了完整性和正确性，通常它的返回值就是view完成布局之后的view.frame.size的值。

它们之前存在相互调用的关系吗？经过测试发现，三者之前没有直接的调用关系。但是能得出这样的结论：intrinsicContentSize的返回值会直接影响systemLayoutSizeFittingSize:的返回值。至于底层是如何处理的不得而知。


### 参考

- [深入理解Auto Layout 第一弹](http://zhangbuhuai.com/beginning-auto-layout-part-1/)