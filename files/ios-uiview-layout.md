## 有关UIView绘制

UIView Layout 相关方法：

```
- (CGSize)sizeThatFits:(CGSize)size
- (void)sizeToFit
——————-

- (void)layoutSubviews
- (void)layoutIfNeeded
- (void)setNeedsLayout
——————–

- (void)setNeedsDisplay
- (void)drawRect
```

#### 布局

- `-layoutSubviews`：这个方法，默认没有做任何事情，需要子类进行重写
- `-setNeedsLayout`：标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，在layoutIfNeeded里会调用layoutSubviews。(在receiver标上一个需要被重新布局的标记，在系统runloop的下一个周期自动调用layoutSubviews)
- `-layoutIfNeeded`：如果有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）


#### 绘制

- `-drawRect:`：重写此方法，执行重绘任务
- `-setNeedsDisplay`：标记为需要重绘，异步掉用drawRect
- `-setNeedsDisplayInRect:`：标记为需要局部重绘


#### 关系

layoutSubviews方法调用先于drawRect



### 参考

- [UIView官方文档](https://developer.apple.com/documentation/uikit/uiview?language=objc)