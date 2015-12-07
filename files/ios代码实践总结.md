### iOS代码实践总结
-----

#### [参考： iOS代码实践总结](http://www.cocoachina.com/ios/20150923/13531.html)

##### 1. 减少对象属性

- 头文件中尽可能少暴露变量或方法，而要使用extension或者category放在.m文件，或者专门的private头文件中
- 使用局部变量或者`__block变量代替`
- 可以尽可能避免循环引用

##### 2. 减少和模块化对象消息

- 减少对象消息     
就是减少target-action和protocol，多使用block，UI和action在一起，代码不用跳来跳去。

- 模块化     
多使用#pragma mark - XXX 分模块。
	
##### 3. MVVM & RAC

- MVVM 核心思想是 data binding, 就是使用KVO。    
- 数据部分的逻辑抽取放在ViewModel中，然后让UI和ViewModel中的数据binding，这个不会减少代码量，但是会大大简化逻辑的复度。
- RAC 影响性能。回调栈太深，但是影响有限。

[参考： RAC/MVVM](http://blog.csdn.net/colorapp/article/details/46524893)

##### 4. UI开发

- 重写setter方法和Code Block Evaluation C Extension语法 

	1. [参考： setter代码风格](http://casatwy.com/iosying-yong-jia-gou-tan-viewceng-de-zu-zhi-he-diao-yong-fang-an.html)    
	2. [参考： GCC Code Block Evaluation C Extension ({…})语法](http://blog.csdn.net/colorapp/article/details/47006771)			

```
self.searchBar = ({
	UISearchBar *searchBar = [[UISearchBar alloc] initWithFrame:({
        CGRect frame = self.tableView.frame;
        frame.size.height = 50.0f;
        frame;
    })];
    searchBar.delegate = self;
    searchBar;
});
```
语法优点：    
1. 结构会更加清晰    
2. 非常简洁的命名来命名局部变量（不怕重名）

- 复杂UI的开发
	1. 组合式view。view hierarchy
	2. 