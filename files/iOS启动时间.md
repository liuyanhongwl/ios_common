## iOS启动时间

启动过程可分为main()之前部分和main()之后部分, 之前的部分主要工作是加载可执行文件和动态库(dylib)，而之后的工作是构建第一个界面，并完成渲染展示。


在WWDC 2016上首次提到了关于App应用启动速度优化的话题:[Session 406 Optimizing App Startup Time](https://developer.apple.com/videos/play/wwdc2016/406/) .该Session上Apple建议一个App完整的启动时间应该保证400ms之内,而若超过20s后还未完全启动App,那么App进程就会被系统杀死.而如何Debug和优化应用启动的时间,官方提出一系列方法来关注应用启动时执行main()前究竟干了些什么.而通过这个Session,你会了解到以下内容:


测量耗时的方法分为**热启动(warm launch)**和**冷启动(cold launch)**。 热启动的时候由于系统核心缓存中有之前加载的动态库，所以会快于冷启动；而冷启动是指的是每次都重启设备后进行的启动，通常取冷启动的时长作为测量数据。


App是以镜像(image)为单位进行加载的，镜像的类型有:

1. executable可执行文件
2. dylib 动态链接库 framework就是动态链接库和相应资源包含在一起的一个文件夹结构
2. bundle 资源文件 只能用dlopen加载，不推荐使用这种方式加载

App开始启动后， 系统首先加载可执行文件，然后加载动态链接库Dyld，Dyld是一个专门用来加载动态链接库的库。 执行从Dyld开始，Dyld从可执行文件的依赖开始, 递归加载所有的依赖动态链接库。

动态链接库的加载步骤分为5步：

1. load dylibs image 读取库镜像文件
2. Rebase image
3. Bind image
4. Objc setup
5. initializers

### 1. load dylibs image

在每个动态库的加载过程中， Dyld需要：

1. 分析所依赖的动态库
2. 找到动态库的mach-o文件
3. 打开文件
4. 验证文件
5. 在系统核心注册文件签名
6. 对动态库的每一个segment调用mmap()

通常的，一个app需要加载100到400个dylibs， 但是其中的系统库被优化，可以很快的加载。

针对这一步骤的优化有：

1. 减少非系统库的依赖。
2. 合并非系统库
3. 使用静态资源，比如把代码加入主程序

### 2. rebase/bind

由于ASLR(address space layout randomization)的存在，可执行文件和动态链接库在虚拟内存中的加载地址每次启动都不固定，所以需要这2步来修复镜像中的资源指针，来指向正确的地址。

rebase修复的是指向当前镜像内部的资源指针； 而bind指向的是镜像外部的资源指针。

rebase步骤先进行，需要把镜像读入内存，并以page为单位进行加密验证，保证不会被篡改，所以这一步的瓶颈在IO。bind在其后进行，由于要查询符号表，来指向跨镜像的资源，加上在rebase阶段，镜像已被读入和加密验证，所以这一步的瓶颈在于CPU计算。

通过命令行可以查看相关的资源指针:

```
xcrun dyldinfo -rebase -bind -lazy_bind myapp.app/myapp
```

命令后面加上下面参数，可以统计__DATA数量：

```
| grep "__DATA" | wc -l
```


优化该阶段的关键在于减少__DATA segment中的指针数量。我们可以：

1. 减少Objc类数量， 减少selector数量
2. 减少C++虚函数数量
3. 转而使用swift stuct

### 3. Objc setup

这一步主要工作是:

1. 注册Objc类 (class registration)
2. 把category的定义插入方法列表 (category registration)
3. 保证每一个selector唯一 (selctor uniquing)

由于之前2步骤的优化，这一步实际上没有什么可做的。

### 4. initializers

以上三步属于静态调整(fix-up)，都是在修改__DATA segment中的内容，而这里则开始动态调整，开始在堆和堆栈中写入内容。

在这里的工作有：

1. Objc的+load()函数
2. C++的构造函数属性函数 形如__attribute__((constructor)) void DoSomeInitializationWork()
3. 非基本类型的C++静态全局变量的创建(通常是类或结构体)(non-trivial initializer) 比如一个全局静态结构体的构建，如果在构造函数中有繁重的工作，那么会拖慢启动速度

Objc的load函数和C++的静态构造函数采用由底向上的方式执行，来保证每个执行的方法，都可以找到所依赖的动态库。

优化方法：

1. 使用+(void)initialize()替换+(void)load()
2. 把__attribute__((constructor))替换为dispatch_once(), pthread_once(), std::once()。
3. 添加编译器选项-Wglobal-constructors，查找C++的非简单类型全局变量的创建
4. 替换代码为swift

注意： 由于在该步骤没有使用锁，所以不要在这里创建线程或调用dlopen，否则会引起性能问题和多线程问题

### 5. 其他

不要在必要的库上设置optional属性，可能会增加工作量。

#### 查看运行时间

Edit Scheme -> Arguments -> Run -> 环境变量 -> DYLD_PRINT_STATISTICS 设置 1。

```
Total pre-main time: 1.5 seconds (100.0%)
         dylib loading time: 216.41 milliseconds (14.2%)
        rebase/binding time: 221.66 milliseconds (14.6%)
            ObjC setup time: 211.34 milliseconds (13.9%)
           initializer time: 865.85 milliseconds (57.1%)
           slowest intializers :
             libSystem.B.dylib :  29.96 milliseconds (1.9%)
   libBacktraceRecording.dylib :  63.25 milliseconds (4.1%)
          libglInterpose.dylib : 245.11 milliseconds (16.1%)
         libMTLInterpose.dylib :  97.61 milliseconds (6.4%)
                     DailyCast : 823.20 milliseconds (54.3%)
```

### main()后的启动部分

在main()函数之后，App的主要工作就是读取必要设置，显示首页内容。而我们的优化也是围绕如何能够快速展现首页来开展。

#### 测量时长方法

这部分的测试方法需要写一些代码：

```
// main.m文件中
static CGFloat StartTime;
int main(int argc, char **argv) {
    StartTime = CFAbsoluteTimeGetCurrent();
}

// appDelegate.m 文件中
extern CGFloat StartTime;
- (void)applicationDidFinishLaunching:(UIApplication *)app {
    dispatch_async(dispatch_get_main_queue(), ^{
        NSlog(@"Launched in %f sec", CFAbsoluteTimeGetCurrent() - StartTime);
    }); 
}
```

#### 1. 配置文件

NSUserdefaults中的配置文件需要在这里加载，由于是一次性全部加载，所以不要在NSUserdefaults中写入过大的内容。


#### 2. 首页的创建

如果首页使用了nib文件，那么要尽可能的减小nib文件。

在加上首页创建所需视图、必要服务的启动、必要数据的创建和读取，这些就是我们可以尝试优化的地方。




#### 参考

- [iOS App启动优化](https://www.v2fs.com/ios-app-startup-optimize/)
- [解决 Swift + CocoaPods 因动态库导致启动时间过长
](http://kittenyang.com/dyld-image-loading-performance/)
- [iOS性能优化系列二：iOS应用启动速度优化](http://ewangke.github.io/2012/08/20/ios-app-launch-time-optimization/)