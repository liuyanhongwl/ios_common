## iOS vs JS

### UIWebView 的 stringByEvaluatingJavaScriptFromString 方法

```
[webView stringByEvaluatingJavaScriptFromString:@"showAlert('hello simple js')"];
```

### JavaScriptCore

iOS7 以后引进的

>JSContext：给JavaScript提供运行的上下文环境,通过-evaluateScript:方法就可以执行一JS代码
JSValue：JavaScript和Objective-C数据和方法的桥梁,封装了JS与ObjC中的对应的类型，以及调用JS的API等
JSManagedValue：管理数据和方法的类
JSVirtualMachine：处理线程相关，使用较少
JSExport：这是一个协议，如果采用协议的方法交互，自己定义的协议必须遵守此协议

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

也可以把实现 JSExport 协议的 OC 方法注入到 web 中， 供 JS 调用。

```
//获取js环境对象
self.jsContext = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
//把实现了 JSObjcDelegate 代理的方法封装成对象（BridgeObject）注入到 web 页面中
self.jsContext[@"BridgeObject"] = self;
self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
    context.exception = exceptionValue;
    NSLog(@"异常信息：%@", exceptionValue);
};
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

