### iOS打包与证书制作


##### 打包

 用`公司账号`打`开发包` ：set xcode Code Sign -> set xcode Provisioning Profile -> xcode archive -> archive show in finder -> xxxx.app -> drag to itunes -> drag out from itunes -> xxxx.ipa
 
##### 版本构建

这里我想解释一下，Xcode项目设置里面的Version选项和Build选项的区别。

`Version`：标识着App的版本号。那么为什么又多一个Build选项出来呢？其实这里Apple设计很巧妙，上传到iTunes Connect的构建版本，已经不能删除了，可能我没找到这样的功能，那么我们构建的版本有bug，想重新上传，那么App的版本号已经不能修改了，所以 就产生Build这个东西。

`Build`：标识着App的构建版本号，即是App二进制包的标识，这样重新上传iTunes Connect就不会发生冲突的情况。