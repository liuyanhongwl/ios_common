## iOS 单例

>单例模式可能是设计模式中最简单的形式了，这一模式的意图就是使得类中的一个对象成为系统中的唯一实例。它提供了对类的对象所提供的资源的全局访问点。因此需要用一种只允许生成对象类的唯一实例的机制。

下面让我们来看下单例的作用：

- 可以保证的程序运行过程，一个类只有一个示例，而且该实例易于供外界访问
- 从而方便地控制了实例个数，并节约系统资源。

### 方法一（误）

```
+ (instancetype)sharedInstance
{
    static Singleton *instance = nil;
    if (!instance) {
        instance = [[Singleton alloc] init];
    }
    return instance;
}
```

这种方式的单例不是线程安全的。

假设此时有两条线程：线程1和线程2，都在调用shareInstance方法来创建单例，那么线程1运行到if (instance == nil)出发现instance = nil,那么就会初始化一个instance，假设此时线程2也运行到if的判断处了，此时线程1还没有创建完成实例instance，所以此时instance = nil还是成立的，那么线程2又会创建一个instace。

为了解决线程安全问题，可以使用dispatch_once、互斥锁。

### 方法二 (误)

```
static Singleton *instance = nil;
+ (instancetype)sharedInstance
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[Singleton alloc] init];
    });
    return instance;
}
```

或

```
static Singleton *instance = nil;
+ (instancetype)sharedInstance
{
    @synchronized (self) {
        if (!instance) {
            instance = [[Singleton alloc] init];
        }
    }
    return instance;
}
```

上面的两个方法保证了线程安全，但是不够全面。如果使用其他方式创建，能创建出不同的对象，违背了单例的设计原则。

```
Singleton *s = nil;
s = [Singleton sharedInstance];
NSLog(@"%@", s);
s = [[Singleton alloc] init];
NSLog(@"%@", s);
s = [Singleton new];
NSLog(@"%@", s);
```

打印出三个不同的地址

```
2016-12-21 20:46:30.414 Singleton[28843:2198096] <Singleton: 0x6000000168c0>
2016-12-21 20:46:30.415 Singleton[28843:2198096] <Singleton: 0x610000016340>
2016-12-21 20:46:30.415 Singleton[28843:2198096] <Singleton: 0x6180000164a0>
```

### 方法三（误）


为了防止别人不小心利用alloc/init方式创建示例，也为了防止别人故意为之，我们要保证不管用什么方式创建都只能是同一个实例对象，这就得重写另一个方法。

在方法二的基础上增加重写下面的方法：

```
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [super allocWithZone:zone];
    });
    return instance;
}
```
再测试，发现打印出来的地址都一样了。

但是，还没结束。

我们添加一些属性：

```
@property (assign, nonatomic)int height;
@property (strong, nonatomic)NSObject *object;
@property (strong, nonatomic)NSMutableArray *array;
```

然后重写-description方法：

```
- (NSString *)description
{
    NSString *result = @"";
    result = [result stringByAppendingFormat:@"<%@: %p>",[self class], self];
    result = [result stringByAppendingFormat:@" height = %d,",self.height];
    result = [result stringByAppendingFormat:@" array = %p,",self.array];
    result = [result stringByAppendingFormat:@" object = %p,",self.object];
    return result;
}
```

还用上面的方法，打印结果：

```
2016-12-21 20:58:03.523 Singleton[29239:2252268] <Singleton: 0x608000039d00> height = 10, arrayM = 0x60800005b150, object = 0x60800000b3e0,
2016-12-21 20:58:03.523 Singleton[29239:2252268] <Singleton: 0x608000039d00> height = 10, arrayM = 0x618000052540, object = 0x61800000b430,
2016-12-21 20:58:03.524 Singleton[29239:2252268] <Singleton: 0x608000039d00> height = 10, arrayM = 0x60800004ae00, object = 0x60800000b3e0,
```

可以看到，尽管使用的是同一个示例，可是他们的属性却不一样。

因为尽管没有为示例重新分配内存空间，但是因为又执行了init方法，会导致property被重新初始化。

### 方法四

为了保证属性的初始化只执行一次，可以将属性的初始化或者默认值设置加上dispatch_once。

```
+ (instancetype)sharedInstance
{
    return [[Singleton alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [super allocWithZone:zone];
    });
    return instance;
}

- (instancetype)init
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [super init];
        if (instance) {
            instance.height = 10;
            instance.object = [[NSObject alloc] init];
            instance.array = [[NSMutableArray alloc] init];
        }
    });
    return instance;
}
```

这种方式保证了单例的唯一，也保证了属性初始化的唯一。


### 关于线程安全

**GCD的dispatch_once方式：**保证程序在运行过程中只会被运行一次，那么假设此时线程1先执行shareInstance方法，创建了一个实例对象，线程2就不会再去执行dispatch_once的代码了。从而保证了只会创建一个实例对象。

**互斥锁方式：**会把锁内的代码当做一个任务，这个任务执行完毕前，不会被其他线程访问。

但是这种简单的互斥锁方式在每次调用单例时都会锁一次，很影响性能，单例使用越频繁，影响越大。

### 优化互斥锁方式

**DCL（double check lock）：**双重检查模式是优化了的互斥锁方式，过程就是check-lock-check，是对静态变量instance的两次判空。第一次判空避免了不必要的同步，第二次判空是为了创建实例。

将上面的简单互斥锁方式修改一下：

```
 if (!instance) {
        @synchronized (self) {
            if (!instance) {
                instance = [super allocWithZone:zone];
            }
        }
    }
return instance;
```

DCL优点是资源利用率高，第一次执行时单例对象才被实例化，效率高。缺点是第一次加载时反应稍慢一些，在高并发环境下也有一定的缺陷，虽然发生的概率很小。

效率：
GCD > DCL > 简单互斥锁

### 使用+load或+initialize

load方法与initialize方法都会被Runtime自动调用一次，并且在Runtime情况下，这两个方法都是线程安全的。

根据这种特性，来实现单例类。

```
+ (void)initialize
{
    if ([self class] == [Singleton class] && instance == nil) {
        instance = [[Singleton alloc] init];
        instance.height = 10;
        instance.object = [[NSObject alloc] init];
        instance.array = [[NSMutableArray alloc] init];
    }
}

+ (instancetype)sharedInstance
{
    return instance;
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    if (instance == nil) {
        instance = [super allocWithZone:zone];
    }
    return instance;
}
```

1. `if([self class] == [Singleton class]...)` 是为了保证 initialize方法只有在本类而非subclass时才执行单例初始化方法。
2. ` if (... && instance == nil)` 是为了防止+initialize多次调用而产生多个实例（除了Runtime调用，我们也可以显示调用+initialize方法）。经过测试，当我们将+initialize方法本身作为class的第一个方法执行时，Runtime的+initialize会被先调用（这保证了线程安全），然后我们自己显示调用的+initialize函数再被调用。 由于+initialize方法的第一次调用一定是Runtime调用，而Runtime又保证了线程安全性，因此这里只简单的检测 singalObject == nil即可。


最好不用+load来做单例是因为它是在程序被装载时调用的，可能单例所依赖的环境尚未形成，它比较适合对Class做设置。(先知道更多关于+load和+initialize的知识，[看这里](load_initialize.md))

### 使用宏

如果我们需要在程序中创建多个单例，那么需要在每个类中都写上一次上述代码，非常繁琐。

我们可以使用宏来封装单例的创建，这样任何类需要创建单例，只需要一行代码就搞定了。

```
#define SingletonH(name) + (instancetype)shared##name;

#define SingletonM(name)    \
static id instance = nil;   \
+ (instancetype)sharedInstance  \
{   \
    static dispatch_once_t onceToken;   \
    dispatch_once(&onceToken, ^{    \
        instance = [[[self class] alloc] init];  \
    }); \
    return instance;    \
}   \
```

### 其他

当然单例如果实现了NSCopying和NSMutableCopying协议，可以补充下面的方法：

```
- (id)copyWithZone:(NSZone *)zone
{
    return instance;
}

- (id)mutableCopyWithZone:(NSZone *)zone
{
    return instance;
}
```

#### 参考

- [iOS中的单例你用对了么？](http://www.cocoachina.com/ios/20160713/17017.html)
- [iOS单例详解](http://www.jianshu.com/p/5226bc8ed784)
- [NSObject的load与initialize方法](http://blog.csdn.net/u013378438/article/details/52060925)