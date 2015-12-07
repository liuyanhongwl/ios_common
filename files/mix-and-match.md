## 项目中同时有objective-c 和swift

### Mix and Match 

Swift 与 Objective-C 的兼容能力使你可以在同一个工程中同时使用两种语言。你可以用这种叫做 `mix and match` 的特性来开发基于混合语言的应用，可以用 Swfit 的最新特性实现应用的一部分功能，并无缝地并入已有的 Objective-C 的代码中。



在objective-c项目中接入 swift

#import "NSURLSession_test-Swift.h"


NSURLSession-test-Bridging-Header.h
#import "RNCryptor.h"




### 参考

[【中文参考】在一个工程中同时使用Swift和Objective-C](http://c.biancheng.net/cpp/html/2268.html)

[【developer.apple参考】MixandMatch](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)
