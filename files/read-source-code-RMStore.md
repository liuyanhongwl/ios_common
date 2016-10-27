## RMStore（in-app purchase）(内购)

### 结构：

- store
- - product delegates 实现请求商品的代理
- - payment params	存储付款成功、失败的回调。交易的状态、行为由store监听。
- - - success
- - - failure

### 功能点：

- 请求产品信息（Request products）
- 添加付款（Add payment）
- 恢复交易（Restore transactions）
- 验证凭证
- 下载内容

#### 验证凭证

增加验证可以判断应用的信息等是否合法，合法则解锁功能，防止应用被拷贝。
可以看这里了解苹果凭证验证，[Receipt Validation Programming Guide](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Introduction.html#//apple_ref/doc/uid/TP40010573-CH105-SW1)。

```
NSURL *receiptURL = [[NSBundle mainBundle] appStoreReceiptURL];
NSData *receiptData = [NSData dataWithContentsOfURL:receiptURL];
 
// Custom method to work with receipts
BOOL rocketCarEnabled = [self receipt:receiptData
        includesProductID:@"com.example.rocketCar"];
```

#### 下载内容

用户购买以解锁功能后，有两种方法开启应用的功能。

- 功能本来就包含在应用内，但是未购买的情况下不能使用。
- 功能要在需要的时候下载下来。

两种方法各有优缺。比如包大小问题： 功能本来在包内，一开始包会较大，下载时间较长。功能在需要的时候下载，用户得等待下载即使是很小的功能。

解决：集成小尤其特别希望用户购买的功能。需要时下载大的功能，并且这样的功能增加时不需要重新打包上线。


在需要的时候下载这种方法里，也有两种方案。

- 苹果托管内容（Apple-hosted content）
- 自己托管内容（Self-hosted content）

**苹果托管内容（Apple-hosted content）**，在苹果服务器上管理内容，不需要提供自己的服务器，并且内容可以自动在后台下载，应用不需要运行也可以。

**自己托管内容（Self-hosted content）**，需要有自己的服务器进行管理。








