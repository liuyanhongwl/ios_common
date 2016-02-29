## CocoaPods
-----

### 什么是CocoaPods
#### 一. 为什么需要CocoaPods
在进行iOS开发的时候，总免不了使用第三方的开源库，比如SBJson、AFNetworking、Reachability等等。使用这些库的时候通常需要：

下载开源库的源代码并引入工程
向工程中添加开源库使用到的framework
解决开源库和开源库以及开源库和工程之间的依赖关系、检查重复添加的framework等问题
如果开源库有更新的时候，还需要将工程中使用的开源库删除，重新执行前面的三个步骤，顿时头都大了。。。
自从有了CocoaPods以后，这些繁杂的工作就不再需要我们亲力亲为了，只需要我们做好少量的配置工作，CocoaPods会为我们做好一切！

#### 二. 什么是CocoaPods
CocoaPods是一个用来帮助我们管理第三方依赖库的工具。它可以解决库与库之间的依赖关系，下载库的源代码，同时通过创建一个Xcode的workspace来将这些第三方库和我们的工程连接起来，供我们开发使用。

使用CocoaPods的目的是让我们能自动化的、集中的、直观的管理第三方开源库。


#### 三. 下载和安装CocoaPods

`sudo gem install cocoapods`

执行上面的命令后半天无反应，是因为需要翻墙。

我们可以使用淘宝的Ruby镜像来访问cocoapods。

`gem sources --remove https://rubygems.org/` 

`gem sources -a https://ruby.taobao.org/`

为了验证你的Ruby镜像是并且仅是taobao，可以用以下命令查看：

`$ gem sources -l`

只有在终端中出现下面文字才表明你上面的命令是成功的：

```
*** CURRENT SOURCES ***

http://ruby.taobao.org/

```

这时候，你再次在终端中运行：

`$ sudo gem install cocoapods`


#### 四. 使用CocoaPods

比如，在项目中导入AFNetworking类库

1、确定AFNetworking是否支持CocoaPods，可以用CocoaPods的搜索功能验证一下。

`$ pod search AFNetworking`

2、过几秒钟之后，你会在终端中看到关于AFNetworking类库的一些信息。（**注意：如果是第一次使用，一直卡在Setting up CocoaPods master repo界面，说明CocoaPods在将它的信息下载到~/.cocoapods里，cd到该目录，用`du -sh *`命令来查看文件的大小**）

3、在工程中创建一个Podfile文件。

`$ touch Podfile`

然后使用vim编辑Podfile文件，使用命令：

`$ vim Podfile`

在Podfile文件中加入如下：

`
pod ‘AFNetworking‘, ‘~> 2.3.1‘
`
4、然后在终端输入命令安装相应的第三方类库

`
$pod install
`

5、安装成功后，看到之后打开工程都需要从类型为工程名.xcworkspace文件打开。

 
 
#### 五. CocoaPods Podfile 语法

```
pod 'AFNetworking'      //不显式指定依赖库版本，表示每次都获取最新版本  
pod 'AFNetworking', '2.0'     //只使用2.0版本  
pod 'AFNetworking', '> 2.0'     //使用高于2.0的版本  
pod 'AFNetworking', '>= 2.0'     //使用大于或等于2.0的版本  
pod 'AFNetworking', '< 2.0'     //使用小于2.0的版本  
pod 'AFNetworking', '<= 2.0'     //使用小于或等于2.0的版本  
pod 'AFNetworking', '~> 0.1.2'     //使用大于等于0.1.2但小于0.2的版本  
pod 'AFNetworking', '~>0.1'     //使用大于等于0.1但小于1.0的版本  
pod 'AFNetworking', '~>0'     //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本  


脚本

vim publish

chmod u+x publish

./publish
```









