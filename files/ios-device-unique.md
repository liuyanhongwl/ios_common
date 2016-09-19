## iOS唯一标识设备方法

### 一、iOS5以后不能获取手机IMEI
 iOS 2.0版本以后UIDevice提供一个获取设备唯一标识符的方法uniqueIdentifier，通过该方法我们可以获取设备的序列号，这个也是目前为止唯一可以确认唯一的标示符。好景不长，因为该唯一标识符与手机一一对应，苹果觉得可能会泄露用户隐私，**所以在iOS5之后该方法就被废弃掉**了，因此iOS5以后不能获取手机IMEI，但是也是可以**通过私有API**获取手机的IMEI号的，
 但是通过苹果私有API获取IMEI号，上架苹果商店会被拒掉的。
 
### 二、iOS7以后不能通过获得MAC地址来标识手机
应用在iOS6及以下时，可以正确取道Mac地址，在iOS7上，会返回固定值。

这样带来的问题是无法区分具体的iOS设备，有些产品就非常难搞了，目前没有找到可以区分不同iOS设备的方法。
测试过mac地址，确实会返回固定值02:00:00:00:00:00

### 三、通过IDFA标识手机

```
NSString *identifierForAdvertising = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
```

广告标示符是由系统存储着的。不过即使这是由系统存储的，但是有几种情况下，会重新生成广告标示符:

- 如果用户完全重置系统（(设置程序 ->通用 -> 还原 ->还原位置与隐私)，这个广告标示符会重新生成。
- 另外如果用户明确的还原广告(设置程序->通用 -> 关于本机 ->广告 ->还原广告标示符)，那么广告标示符也会重新生成

关于广告标示符的还原，有一点需要注意：如果程序在后台运行，此时用户“还原广告标示符”，然后再回到程序中，此时获取广告标示符并不会立即获得还原后的标示符。必须要终止程序，然后再重新启动程序，才能获得还原后的广告标示符。之所以会这样，因为ASIdentifierManager是一个单例。

### 四、iOS10以后获取IDFA有限制

```
In iOS 10.0 and later, the value of advertisingIdentifier is all zeroes when the user has limited ad tracking.
```

如果用户限制广告追踪，开发者获取IDFA将是 一串数字 0。

### 五、使用钥匙串保存唯一标识

#### 钥匙串的情况时机

一般是在设置里还原，才会清空钥匙串。一般版本升级不会清空。

#### 1.我们首先需要导入Security.frameWork（keychain依赖它），然后需要一个keychain管理器，一个uuid管理器，文件组成如下：

- KeyChainManager
- UUIDManager

#### 2.KeychainManager,其实就是对keychain的增、删、改、查，类似于数据库的处理。

```
#import <Foundation/Foundation.h>

@interface KeyChainManager : NSObject

+ (NSMutableDictionary *)getKeychainQuery:(NSString *)service;

+ (void)save:(NSString *)service data:(id)data;

+ (id)load:(NSString *)service;

+ (void)delete:(NSString *)service;

@end
```

```
#import "KeyChainManager.h"
#import <Security/Security.h>

@implementation KeyChainManager

+ (NSMutableDictionary *)getKeychainQuery:(NSString *)service
{
    return [NSMutableDictionary dictionaryWithObjectsAndKeys:(__bridge_transfer id)kSecClassGenericPassword,(__bridge_transfer id)kSecClass,
            service, (__bridge_transfer id)kSecAttrService,
            service, (__bridge_transfer id)kSecAttrAccount,
            (__bridge_transfer id)kSecAttrAccessibleAfterFirstUnlock, (__bridge_transfer id)kSecAttrAccessible, nil];
}

+ (void)save:(NSString *)service data:(id)data
{
    //Get search dictionary
    NSMutableDictionary *keychainQuery = [self getKeychainQuery:service];
    //Delete old item before add new item
    SecItemDelete((__bridge_retained CFDictionaryRef)keychainQuery);
    //Add new object to search dictionary(Attention:the data format)
    [keychainQuery setObject:[NSKeyedArchiver archivedDataWithRootObject:data] forKey:(__bridge_transfer id)kSecValueData];
    //Add item to keychain with the search dictionary
    SecItemAdd((__bridge_retained CFDictionaryRef)keychainQuery, NULL);
}

+ (id)load:(NSString *)service
{
    id ret = nil;
    NSMutableDictionary *keychainQuery = [self getKeychainQuery:service];
    //Configure the search setting
    [keychainQuery setObject:(id)kCFBooleanTrue forKey:(__bridge_transfer id)kSecReturnData];
    [keychainQuery setObject:(__bridge_transfer id)kSecMatchLimitOne forKey:(__bridge_transfer id)kSecMatchLimit];
    CFDataRef keyData = NULL;
    if (SecItemCopyMatching((__bridge_retained CFDictionaryRef)keychainQuery, (CFTypeRef *)&keyData) == noErr) {
        @try {
            ret = [NSKeyedUnarchiver unarchiveObjectWithData:(__bridge_transfer NSData *)keyData];
        } @catch (NSException *e) {
            NSLog(@"Unarchive of %@ failed: %@", service, e);
        } @finally {
        }
    }
    return ret;
}

+ (void)delete:(NSString *)service
{
    NSMutableDictionary *keychainQuery = [self getKeychainQuery:service];
    SecItemDelete((__bridge_retained CFDictionaryRef)keychainQuery);
}


@end
```

#### 3.再来看下MyUUIDManager文件，实现的是对UUID的增、删、改、查，其中save既是增也是改：

```
#import <Foundation/Foundation.h>

@interface UUIDManager : NSObject

+(void)saveUUID:(NSString *)uuid;

+(NSString *)getUUID;

+(void)deleteUUID;

@end

```

```
#import "UUIDManager.h"
#import "KeyChainManager.h"

static NSString * const KEY_IN_KEYCHAIN = @"com.sube.dailycast.uuid";

@implementation UUIDManager

+(void)saveUUID:(NSString *)uuid
{
    if (uuid && uuid.length > 0) {
        [KeyChainManager save:KEY_IN_KEYCHAIN data:uuid];
    }
}

+(NSString *)getUUID
{
    NSString *uuid = [KeyChainManager load:KEY_IN_KEYCHAIN];
    
    if (!uuid || uuid.length == 0) {
        NSString *uuid = [[NSUUID UUID] UUIDString];
        
        [self saveUUID:uuid];
    }
    return uuid;
}

+(void)deleteUUID
{
    [KeyChainManager delete:KEY_IN_KEYCHAIN];
}

@end
```

#### 4. 记下获取的uuid, 然后卸载APP重新安装一下，再获取uuid与之前一致。

