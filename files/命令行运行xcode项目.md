 d## 命令行运行Xcode项目

【参考】[通过Xcode命令行编译](http://www.jianshu.com/p/55b80e746e38)

### 运行命令行程序

```
$ xcodebuild
```

在项目目录下生成名为build路径

在build/release/目录下有MyiOSApp的可执行文件

```
$ ./MyiOSApp
``` 

### 测试iOS程序

测试MyiOSApp用7.1iPhone Retina (4-inch 64-bit)模拟器

```
$ xcodebuild test -scheme MyiOSApp -destination 'platform=iOS Simulator,name=iPhone Retina (4-inch 64-bit),OS=7.1'
```

测试MyiOSApp用7.1iPhone Retina (4-inch 64-bit)模拟器

```
$ xcodebuild test -scheme MyiOSApp -destination 'platform=iOS Simulator,name=iPad'
```

