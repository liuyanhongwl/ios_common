## iOS与服务器交互

### 一. 加密/解密

#### iOS AES 

问题： 服务器和安卓用"AES/CBC/PKCS5Padding"对请求参数进行加密，iOS如何做AES加密。

解决： 

1. 确定秘钥 (这里是`秘钥`再base64解码才得到`真正的秘钥`)。
2. 让已有加密方式（这里是服务器和安卓）加密一段数据，在iOS的AES加密方法中`调整参数`，直到与已有加密方式加密后的数据一致。

影响加密、解密处理的因素：

1. 秘钥参数
2. 秘钥长度参数

代码：（其他地方用，针对性修改即可）

加密

```
- (NSData *)AESEncryptWithKey:(NSString *)key
{
    //获取真正的秘钥
    NSData *keyData = [[NSData alloc] initWithBase64EncodedString:key options:NSDataBase64DecodingIgnoreUnknownCharacters];
    const unsigned char *ptr = [keyData bytes];
    for(int i=0; i<[keyData length]; ++i) {
        Byte c = *ptr++;
        NSLog(@"char=%c hex=%hhu", c, c);
    }
    
    NSUInteger dataLength = [self length];
    size_t bufferSize = dataLength + kCCBlockSizeAES128;
    void *buffer = malloc(bufferSize);
    size_t numBytesEncrypted = 0;
    
    char cIv[16];
    bzero(cIv, 16);
    
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                          kCCAlgorithmAES128,
                                          kCCOptionPKCS7Padding,
                                          [keyData bytes],
                                          kCCBlockSizeAES128,
                                          cIv,
                                          [self bytes],
                                          dataLength,
                                          buffer,
                                          bufferSize,
                                          &numBytesEncrypted);
    if (cryptStatus == kCCSuccess) {
        return [NSData dataWithBytesNoCopy:buffer length:numBytesEncrypted];
    }
    free(buffer);
    return nil;
}

```

解密

```
- (NSData *)AESDecryptWithKey:(NSString *)key
{
    NSData *keyData = [[NSData alloc] initWithBase64EncodedString:key options:NSDataBase64DecodingIgnoreUnknownCharacters];
    
    NSUInteger dataLength = [self length];
    size_t bufferSize = dataLength + kCCBlockSizeAES128;
    void *buffer = malloc(bufferSize);
    size_t numBytesDecrypted = 0;
    
    char cIv[kCCBlockSizeAES128];
    bzero(cIv, kCCBlockSizeAES128);
    
    CCCryptorStatus cryptStatus = CCCrypt(kCCDecrypt,
                                          kCCAlgorithmAES128,
                                          kCCOptionPKCS7Padding,
                                          [keyData bytes],
                                          kCCBlockSizeAES128,
                                          cIv,
                                          [self bytes],
                                          dataLength,
                                          buffer,
                                          bufferSize,
                                          &numBytesDecrypted);
    if (cryptStatus == kCCSuccess) {
        return [NSData dataWithBytesNoCopy:buffer length:numBytesDecrypted];
    }
    free(buffer);
    return nil;
}

```

参数解释

- CCOptions : kCCOptionPKCS7Padding、kCCOptionECBMode。补全方式。
- - iOS的这个参数设置里面没有PKCS5Padding，但是PKCS7Padding兼容PKCS5Padding。
- - 添加kCCOptionECBMode参数，表示是ECB方式，去掉这个参数，就表示是CBC方式。


#### iOS URL encode decode

##### URL 中有汉字：

```
[urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];

```

上面的方法iOS9过时，用下面代替

```
[urlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];

```

### 二. 发送的POST请求体，服务器接受不到

问题： 要发送给服务器的数据进行POST加密，放到请求体（httpBody）中，服务器接收不到。

解决： 

```
[request setValue:@"application/octet-stream; charset=UTF-8" forHTTPHeaderField:@"Content-Type"];
```

原因： 服务器是从stream中获取数据的。

这里DailyCast与服务器的交互是按照，传加密的数据的话设置 `Content-Type`为 `application/octet-stream; charset=UTF-8`, 其他情况设置为 `application/x-www-form-urlencoded; charset=UTF-8`。
