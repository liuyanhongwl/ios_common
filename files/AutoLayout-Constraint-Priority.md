## AutoLayout - Constraint Priority

### 写在前面

从iOS 6开始推出auto layout，auto layout虽不止局限于constraint，但是constraint确实auto layout的重中之重。

本文主要讲关于constraint的优先级，其中涉及的主要API有：

- setContentCompressionResistancePriority：forAxis：
- setContentHuggingPriority：forAxis：


### 优先级

问题经常出现在当有两个视图时，在固定宽度的区域内，如果不够放置，优先压缩哪个。

Content Hugging 和 Content Compression Resistance

这两个属性对有intrinsic content size的控件（例如button，label）非常重要。通俗的讲，具有intrinsic content size的控件自己知道（可以计算）自己的大小，例如一个label，当你设置text，font之后，其大小是可以计算到的。

**UIView中关于Content Hugging 和 Content Compression Resistance的方法有：**

```
- (UILayoutPriority)contentCompressionResistancePriorityForAxis:(UILayoutConstraintAxis)axis
- (void)setContentCompressionResistancePriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis
- (UILayoutPriority)contentHuggingPriorityForAxis:(UILayoutConstraintAxis)axis
- (void)setContentHuggingPriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis

```

**大概的意思就是设置优先级的。**

Hugging priority 确定view有多大的优先级阻止自己变大。

Compression Resistance priority确定有多大的优先级阻止自己变小。

#### Content Hugging

很抽象，其实content Hugging就是要维持当前view在它的optimal size（intrinsic content size），可以想象成给view添加了一个额外的width constraint，此constraint试图保持view的size不让其变大：

view.width <= optimal size

此constraint的优先级就是通过上面的方法得到和设置的，content Hugging默认为250.

#### Content Compression Resistance

Content Compression Resistance就是要维持当前view在他的optimal size（intrinsic content size），可以想象成给view添加了一个额外的width constraint，此constraint试图保持view的size不让其变小：

view.width >= optimal size

此默认优先级为750.

