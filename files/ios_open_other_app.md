## iOS open other app

#### 一. 应用程序间通信

例如，要在Safari应用程序中打开Google主页，我们可以编写如下代码：

```
NSURL *url = [NSURL URLWithString:@"http://google.com"]; 
[[UIApplication sharedApplication] openURL:url];
```

这里的http://部分叫做URL方案（URL scheme），它表示想要载入的应用程序。

还有几种用于本地iPhone应用程序的URL方案，并且可以使用类似的方式来启动它们。[About Apple URL Schemes](https://developer.apple.com/library/prerelease/ios/featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007899)


#### 二. A应用打开B应用。

原理： A应用根据URL Schemes来启动B应用。

- B应用 注册一个自定义的URL Schemes
- A应用 通过B注册的URL Schemes启动B

##### 1. B应用 注册URL Schemes

在B应用的info.plist文件下加入 URL Types

```
<key>CFBundleURLTypes</key>
<array>
	<dict>
		<key>CFBundleURLName</key>
		<string>com.wlnana17.B</string>
		<key>CFBundleURLSchemes</key>
		<array>
			<string>myapp</string>
		</array>
	</dict>
</array>

```

`CFBundleURLName`: 是URL Identifier，是自定义的URL Scheme的名字，建议采用反转域名的方法保证该名字的唯一性，比如come.yourCompany.yourApp。

`CFBundleURLSchemes`: 是一个数组，可以定义多个URL Schemes。这里不需要再后面追加://，如果设置为myapp， 那么自定义的url就是myapp://。

##### 2. A应用 使用B注册的URL Schemes启动B

```
 BOOL canOpen = [[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"myapp://"]];
            BOOL isOpen = NO;
            if (canOpen) {
                isOpen = [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"myapp://openedapp.wlnana17.com?property1=10"]];
            }
```

##### 3. iOS9 URL Schemes

在iOS9以后，如果使用 canOpenURL: 方法，该方法所涉及到的 URL scheme 必须在"Info.plist"中将它们列为白名单，否则不能使用。key叫做LSApplicationQueriesSchemes ，键值内容是

```
<key>LSApplicationQueriesSchemes</key>
<array>
 <string>urlscheme</string>
 <string>urlscheme2</string>
 <string>urlscheme3</string>
 <string>urlscheme4</string>
</array> 

```

所以在iOS9以后，我们需要在A应用的info.plist里加入如下，才能打开B应用

```
<key>LSApplicationQueriesSchemes</key>
<array>
 <string>myapp</string>
</array> 

```

苹果为什么要这么做？

在 iOS9 之前，你可以使用 canOpenURL: 监测用户手机里到底装没装微信，装没装微博。但是也有一些别有用心的 App ，这些 App 有一张常用 App 的 URL scheme，然后他们会多次调用canOpenURL: 遍历该表，来监测用户手机都装了什么 App ，比如这个用户装了叫“大姨妈”的App，你就可以知道这个用户是女性，你就可以只推给这个用户女性用品的广告。这是侵犯用户隐私的行为。

##### 4. 打开B应用

被打开的B应用会调用**application:openURL:options:**方法

- 如果B应用不在后台：会走**application:willFinishLaunchingWithOptions:** 和 **application:didFinishLaunchingWithOptions:**方法，如果这两个方法有一个返回NO，将不会走**application:openURL:options:**方法。

- 如果B应用在后台，不会走**application:willFinishLaunchingWithOptions:** 和 **application:didFinishLaunchingWithOptions:**方法，所以会调用**application:openURL:options:**


#### 三. 问题

```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options NS_AVAILABLE_IOS(9_0); // no equiv. notification. return NO if the application can't open for some reason

```

为什么返回NO无效果？有知道的小伙伴指点一下。

#### 四. 应用程序之间共享数据

这里的共享数据指的是APP跳转时，通过URL传递数据。

#### 五. 其他

也可以在Safari中，键入使用定制模式的URL（myapp://），确认是否启动B应用，**就可以从Safari中打开注册的B应用**。

详情请看 [App Programming Guide for iOS](https://developer.apple.com/library/prerelease/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html#//apple_ref/doc/uid/TP40007072-CH6-SW2)

