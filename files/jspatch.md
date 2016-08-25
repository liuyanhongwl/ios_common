## JSPatch

[github JSPatch 文档](https://github.com/bang590/JSPatch/wiki)

#### 基本流程：

- 判断是否在审核中
- 从服务端拉取需要下载JS.zip文件的版本
- 下载JS.zip文件
- 从服务端下来的字段判断是下载后立即执行还是下次启动再执行
- 立即执行（适用于替换还未发生的函数，已经生成的界面只有重新生成才会改变）
- 下次启动执行下载下来并解压解密的JS文件（保证所有方法都是替换后才执行）

#### 特殊流程：

- JPCleaner 即时撤回脚本
- safari 断点调试

#### 不适用的地方：

- 因JavaScriptCore导致死锁问题（执行JavaScriptCore会加JS锁）
- - webView
- - JS锁和其他锁导致死锁
- block引起 （不支持JS 封装的 block 传到 OC 再传回 JS）


### 开发以及使用Loader流程

- #### 开发js代码，先在本地执行调试

```
[JPEngine startEngine];
NSString *path = [[NSBundle mainBundle] pathForResource:@"main" ofType:@"js"];
[JPEngine evaluateScriptWithPath:path];
```

- #### 将Loader文件拷贝到项目中
- #### 修改JPLoader.h的rootUrl为服务器地址
- #### 生成 RSA 公钥私钥，替换 JPLoader.h 里的 publicKey 和 tools/pack.php 里的 privateKey。

在 Mac 终端上执行 openssl，再执行以下三句命令，生成 PKCS8 格式的 RSA 公私钥，执行过程中提示输入密码，密码为空（直接回车）就行。

```
$ openssl
```

```
openssl >

genrsa -out rsa_private_key.pem 1024

pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM –nocrypt

rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem

```

这样在执行的目录下就有了 rsa_private_key.pem 和 rsa_public_key.pem 这两个文件。这里生成了长度为 1024 的私钥，长度可选 1024 / 2048 / 3072 / 4096 ...。

- #### 使用 Loader/packer.php 脚本打包JS文件

```
$ php pack.php main.js other.js 
```

- #### 放到服务器

服务器的存放路径是 ${rootUrl}/${appVersion}/${patchFile}

- #### 下载/更新脚本

举个例子，客户端当前 App 版本号为 1.0，上述配置 rootUrl 变量配为 http://localhost/JSPatch/，服务端告诉客户端最新脚本版本号为2，于是调用 [JPLoader updateToVersion:2 callback:nil]，这时会去请求 http://localhost/JSPatch/1.0/v2.zip 这个文件并解压验证，保存到本地目录等待执行。

- #### 安全性

关于JSPatch Loader 的RSA加密只处理脚本校验，防止传输过程被第三方篡改，但不会对脚本内容进行加密传输和存储，对脚本内容有加密需求的可以自行加上加密逻辑。

[JSPatch的安全问题](http://jspatch.com/Docs/security)

