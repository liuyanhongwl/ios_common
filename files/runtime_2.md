## Runtime 2 动态方法解析和转发

当一个对象能接收一个消息时，就会走正常的方法调用流程，也就是上面的消息传递流程。但如果一个对象无法接收指定消息时，又会发生什么事呢？

当向someObject发送某消息，但runtime system在当前类和父类中都找不到对应方法的实现时，runtime system并不会立即报错使程序崩溃，而是依次执行下列步骤：

![Message forwarding2](../images/runtime/message_forwarding_1.png)

流程：

- 动态方法解析
- 快速消息转发
- 标准消息转发
    
#### 动态方法解析（Dynamic Method Resolution）   

对象在接收到未知的消息时，首先会调用所属类的类方法`+resolveInstanceMethod:`(实例方法)或者`+resolveClassMethod:`(类方法)。在这个方法中，我们有机会为该未知消息新增一个”处理方法””。

我们可以在这里添加对该消息的处理方法，并返回YES，则将重新objc_msgSend：

```
+ (BOOL)resolveInstanceMethod:(SEL)aSEL  
{  
    if (aSEL == @selector(resolveThisMethodDynamically)) {  
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");  
          return YES;  
    }  
    return [super resolveInstanceMethod:aSEL];  
} 
```

#### 快速消息转发

调用`-forwardingTargetForSelector:`方法，尝试找到一个能响应该消息的对象。如果获取到，则直接把消息转发给它，返回非 nil 对象。否则返回 nil ，继续下面的动作。注意，这里不要返回 self ，否则会形成死循环。

```
- (id)forwardingTargetForSelector:(SEL)aSelector  
{  
    Doctor *doctor = [[Doctor alloc]init];  
    if ([doctor respondsToSelector:aSelector]) {  
        return doctor;  
    }  
    return nil;  
} 
```

这一步我们只想将消息转发到另一个能处理该消息的对象上。但这一步无法对消息进行处理，如操作消息的参数和返回值。

#### 标准消息转发

如果想要对消息进行处理，就可以在这一步操作消息的参数和返回值，还可以让多个对象响应，甚至把消息吞掉。

runtime system会调用`-methodSignatureForSelector:`方法，尝试获得一个方法签名。如果获取不到，则直接调用`-doesNotRecognizeSelector`抛出异常。如果能获取，则返回非nil：runtime system会创建一个 NSlnvocation 并传给`-forwardInvocation:`。

也就是说如果本类没有能响应的方法，`-methodSignatureForSelector:`方法本来应该返回nil，需要重写该方法想办法返回需要的签名，好让runtime system可以调用`-forwardInvocation:`。重写`-forwardInvocation:`，来对消息进行处理（交给其它对象处理、处理消息参数等）。

需要重写`-methodSignatureForSelector:`和`-forwardInvocation:`方法：

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector  
{  
    NSMethodSignature* signature = [super methodSignatureForSelector:aSelector];  
    if (signature==nil) {  
        signature = [someObj methodSignatureForSelector:aSelector];  
    }  
    NSUInteger argCount = [signature numberOfArguments];  
    for (NSInteger i=0 ; i<argCount ; i++) {  
    	//操作参数
    }  
      
    return signature;  
}  
  
- (void)forwardInvocation:(NSInvocation *)anInvocation  
{  
    SEL seletor = [anInvocation selector];  
    if ([someObj respondsToSelector:seletor]) {  
        [anInvocation invokeWithTarget:someObj];  
    }  
      
}  
```

#### 两种消息转发方式的比较

- 快速消息转发：简单、快速、但仅能转发给一个对象。
- 标准消息转发：稍复杂、较慢、但转发操作实现可控，可以实现多对象转发。

#### 消息转发与多继承

转发类似继承，可以用来支持Objective-C 编程的多重继承的某些效果。一个对象通过转发消息来响应消息，该对象似乎是借或者“继承”另一个类中实现的方法。如下图所示：

![Message forwarding2](../images/runtime/message_forwarding_2.gif)

不过消息转发虽然类似于继承，但NSObject的一些方法还是能区分两者。如`-respondsToSelector:`和`-isKindOfClass:`只能用于继承体系，而不能用于转发链。便如果我们想让这种消息转发看起来像是继承，则可以重写这些方法。

#### 消息转发与代理对象

转发不仅模仿多重继承，还可以开发轻量级对象代表原来的对象。代理代替原来的对象并传送消息给原来的对象。

苹果给了更为纯净的类[NSProxy](https://github.com/liuyanhongwl/ios_common/blob/master/files/NSProxy.md)，专门处理消息转发。
