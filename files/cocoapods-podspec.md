## 使用Cocoapods创建私有podspec

[原文地址](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

1. 创建并设置一个私有的Spec Repo。
2. 创建Pod的所需要的项目工程文件，并且有可访问的项目版本控制地址。
3. 创建Pod所对应的podspec文件。
4. 本地测试配置好的podspec文件是否可用。
5. 向私有的Spec Repo中提交podspec。
6. 在个人项目中的Podfile中增加刚刚制作的好的Pod并使用。
7. 更新维护podspec。

遇到的问题：

1. pod lib lint 时要加上需要的sources
2. `$ pod repo push WTSpecs PodTestLibrary.podspec` 时也需要加上需要的sources  