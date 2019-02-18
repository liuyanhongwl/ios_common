## iOS vs JS


### JavaScriptCore

iOS7 以后引进的

- JSContext：JS执行的环境，同时也通过JSVirtualMachine管理着所有对象的生命周期，每个JSValue都和JSContext相关联并且强引用context。
- JSValue：封装了JS与ObjC中的对应的类型，以及调用JS的API等
- JSManagedValue：JS和OC对象的内存管理辅助对象。由于JS内存管理是垃圾回收，并且JS中的对象都是强引用，而OC是引用计数。如果双方相互引用，势必会造成循环引用，而导致内存泄露。我们可以用JSManagedValue保存JSValue来避免。
- JSVirtualMachine：JS运行的虚拟机，有独立的堆空间和垃圾回收机制。
- JSExport：一个协议，如果JS对象想直接调用OC对象里面的方法和属性，那么这个OC对象只要实现这个JSExport协议就可以了。


直接从 JSContext 对象中执行 JS 语句

```
//oc 调用 js 方法
[self.jsContext evaluateScript:@"showAlert('Hello', 'JavaScriptCore')"];
```

或者从 JSContext 对象中获取 JS 方法，进行调用

```
//oc 调用 js 方法
JSValue *showAlert = self.jsContext[@"showAlert"];
[showAlert callWithArguments:@[@"hello", @"JavaScriptCore"]];
```

把实现 JSExport 协议的 OC 方法注入到 web 中， 供 JS 调用。

如果声明了一个UIWebView，也可以使用UIWebView获取到JSContext对象，就可以使用JavaScriptCore的Api了，在UIWebView中获取JSContext的方法是：

```
self.jsContext = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
```

**WKWebView 不支持JavaScriptCore的方式但提供message handler的方式为JavaScript 与Objective-C 通信**

```
//把实现了 JSObjcDelegate 代理的方法封装成对象（BridgeObject）注入到 web 页面中
self.jsContext[@"BridgeObject"] = self;
self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
    context.exception = exceptionValue;
    NSLog(@"异常信息：%@", exceptionValue);
};
```

### UIWebView 的 stringByEvaluatingJavaScriptFromString 方法

```
[webView stringByEvaluatingJavaScriptFromString:@"showAlert('hello simple js')"];
```


### WebKit

#### 1. WKWebView 的 evaluateJavaScript:completionHandler: 方法

```
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^ _Nullable)(_Nullable id, NSError * _Nullable error))completionHandler;
```

```
    NSString *js = @"document.title";
    [self.webView evaluateJavaScript:js completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        //@param result 是调用 js 方法的返回值
    }];
```

#### 2. WKUserContentController 的 addUserScript: 方法

```
- (void)addUserScript:(WKUserScript *)userScript;
```

注入 JS 代码

```
    NSString *js = @"var meta = document.createElement('meta'); \
                    meta.setAttribute('name', 'viewport'); \
                    meta.setAttribute('content', 'width=device-width'); \
                    document.getElementsByTagName('head')[0].appendChild(meta);";

    WKUserScript *script = [[WKUserScript alloc] initWithSource:js injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];
    
    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    [configuration.userContentController addUserScript:script];
```

#### 3. WKUserContentController 的 addScriptMessageHandler:name: 方法

```
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
```

把 oc 可响应的 name 注入到 web 页面中
```
[configuration.userContentController addScriptMessageHandler:self name:@"callCamera"];
```

在 js 中可以通过下面方式调用：
```
window.webkit.messageHandlers.<name>.postMessage(<messageBody>)
```

//self 需要实现 WKScriptMessageHandler 协议的方法
```
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    if ([message.name isEqualToString:@"callCamera"]) {
        ...do simething
    }
}
```

### 参考

- [Objective-C与JavaScript交互的那些事](http://www.cocoachina.com/ios/20160127/15105.html)(简单介绍两种交互方式：JavaScriptCore和拦截URL协议)
- [JavaScriptCore 使用](https://www.jianshu.com/p/a329cd4a67ee)
- [【github】WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)
- [优秀开源代码解读之JS与iOS Native Code互调的优雅实现方案](http://blog.csdn.net/yanghua_kobe/article/details/8209751)(解读WebViewJavascriptBridge)
- [UIWebView和WKWebView的使用及js交互](http://liuyanwei.jumppo.com/2015/10/17/ios-webView.html)
- [【腾讯云】JavaScriptCore全面解析 （上篇）](https://cloud.tencent.com/developer/article/1004875)