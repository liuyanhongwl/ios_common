## Reachability

iOS开发中，一说到Reachability，就能马上想到第三方的Reachability。但是最近苹果审核中，会因使用第三方的Reachability而被拒绝：

>WARNING there have been reports of apps being rejected when Reachability is used in a framework. The only solution to this so far is to rename the class.

原因可能是[第三方的Reachability](https://github.com/tonymillion/Reachability)和苹果官方[Sample Code “Reachability”](https://developer.apple.com/library/content/samplecode/Reachability/Introduction/Intro.html)重名导致审核不通过，第三方Reachability在Github上也声明了需要更改名字，来解决审核被拒的问题。

因为只略读了一下苹果官方Reachability，所以就略述下代码内的使用方法。

### Sample Code Reachability

苹果给的Reachability支持Ipv6。

Reachability类只是对SystemConfiguration.framework里面的SCNetworkReachability.h的一个简单封装示例。

Reachability.h 使用很简单，只暴露了如下接口：

```
+ (instancetype)reachabilityWithHostName:(NSString *)hostName;
+ (instancetype)reachabilityWithAddress:(const struct sockaddr *)hostAddress;
+ (instancetype)reachabilityForInternetConnection;

- (BOOL)startNotifier;
- (void)stopNotifier;

- (NetworkStatus)currentReachabilityStatus;
- (BOOL)connectionRequired;
```

使用：

- **构建：**构建一个Reachability对象
- **监听：**startNotifier监听网络变化。stopNotifier停止监听。
- **状态：**currentReachabilityStatus和connectionRequired方法获取网络状态

如何封装的：

- **构建：**通过SCNetworkReachabilityCreateWithName或者SCNetworkReachabilityCreateWithAddress构建一个SCNetworkReachabilityRef类型的结构体。
- **监听：** startNotifier：通过SCNetworkReachabilitySetCallback方法，设置监听某个SCNetworkReachabilityRef的变化。
- - 通过SCNetworkReachabilityScheduleWithRunLoop方法，给SCNetworkReachabilityRef安排RunLoop模式。
- - stopNotifier：通过SCNetworkReachabilityUnscheduleFromRunLoop方法，取消SCNetworkReachabilityRef的RunLoop模式。
- **状态：**通过SCNetworkReachabilityGetFlags方法获取网络标志位，判断网络标志位，确定当前网络状态。

### 第三方Reachability

主要方法跟苹果demo是一样的，并在起基础上增加更多block和方法。

