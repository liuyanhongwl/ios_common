## AutoLayout

### 当有两个label

Content Hugging 和 Content Compression Resistance

这两个属性对有intrinsic content size的控件（例如button，label）非常重要。通俗的讲，具有intrinsic content size的控件自己知道（可以计算）自己的大小，例如一个label，当你设置text，font之后，其大小是可以计算到的。

##### UIView中关于Content Hugging 和 Content Compression Resistance的方法有：

```
- (UILayoutPriority)contentCompressionResistancePriorityForAxis:(UILayoutConstraintAxis)axis
- (void)setContentCompressionResistancePriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis
- (UILayoutPriority)contentHuggingPriorityForAxis:(UILayoutConstraintAxis)axis
- (void)setContentHuggingPriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis

```

##### 大概的意思就是设置优先级的。

Hugging priority 确定view有多大的优先级阻止自己变大。

Compression Resistance priority确定有多大的优先级阻止自己变小。

##### 1. content Hugging

很抽象，其实content Hugging就是要维持当前view在它的optimal size（intrinsic content size），可以想象成给view添加了一个额外的width constraint，此constraint试图保持view的size不让其变大：

view.width <= optimal size

此constraint的优先级就是通过上面的方法得到和设置的，content Hugging默认为250.

##### 2. Content Compression Resistance

Content Compression Resistance就是要维持当前view在他的optimal size（intrinsic content size），可以想象成给view添加了一个额外的width constraint，此constraint试图保持view的size不让其变小：

view.width >= optimal size

此默认优先级为750.