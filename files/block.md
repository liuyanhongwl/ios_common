# Block

block内部，栈是红灯区，堆是绿灯区。
     
>Block不允许修改外部变量的值。Apple这样设计，应该是考虑到了block的特殊性，block也属于“函数”的范畴，变量进入block，实际就是已经改变了作用域。在几个作用域之间进行切换时，如果不加上这样的限制，变量的可维护性将大大降低。又比如我想在block内声明了一个与外部同名的变量，此时是允许呢还是不允许呢？只有加上了这样的限制，这样的情景才能实现。于是栈区变成了红灯区，堆区变成了绿灯区。

###几种演算

- block调用 基本数据类型

``` 
    {
        NSLog(@"\n--------------------block调用 基本数据类型---------------------\n");

        int a = 10;
        NSLog(@"block定义前a地址=%p", &a);
        void (^aBlock)() = ^(){
            NSLog(@"block定义内部a地址=%p", &a);
        };
        NSLog(@"block定义后a地址=%p", &a);
        aBlock();
    }
    
    /*
     结果：
     block定义前a地址=0x7fff5bdcea8c
     block定义后a地址=0x7fff5bdcea8c
     block定义内部a地址=0x7fa87150b850
     */
    
    /*
     流程：
     1. block定义前：a在栈区
     2. block定义内部：里面的a是根据外面的a拷贝到堆中的，不是一个a
     3. block定义后：a在栈区
     */
    
    {
        NSLog(@"\n--------------------block调用 __block修饰的基本数据类型---------------------\n");
        
        __block int b = 10;
        NSLog(@"block定义前b地址=%p", &b);
        void (^bBlock)() = ^(){
            b = 20;
            NSLog(@"block定义内部b地址=%p", &b);
        };
        NSLog(@"block定义后b地址=%p", &b);
        NSLog(@"调用block前 b=%d", b);
        bBlock();
        NSLog(@"调用block后 b=%d", b);
    }
    
    /*
     结果：
     block定义前b地址=0x7fff5bdcea50
     block定义后b地址=0x7fa873b016d8
     调用block前 b=10
     block定义内部b地址=0x7fa873b016d8
     调用block后 b=20
     */
    
    /*
     流程：
     1. 声明 b 为 __block （__block 所起到的作用就是只要观察到该变量被 block 所持有，就将“外部变量”在栈中的内存地址放到了堆中。）
     2. block定义前：b在栈中。
     3. block定义内部： 将外面的b拷贝到堆中，并且使外面的b和里面的b是一个。
     4. block定义后：外面的b和里面的b是一个。
     5. block调用前：b的值还未被修改。
     6. block调用后：b的值在block内部被修改。
     */
    
    {
        NSLog(@"\n--------------------block调用 指针---------------------\n");
        
        NSString *c = @"ccc";
        NSLog(@"block定义前：c=%@, c指向的地址=%p, c本身的地址=%p", c, c, &c);
        void (^cBlock)() = ^{
            NSLog(@"block定义内部：c=%@, c指向的地址=%p, c本身的地址=%p", c, c, &c);
        };
        NSLog(@"block定义后：c=%@, c指向的地址=%p, c本身的地址=%p", c, c, &c);
        cBlock();
        NSLog(@"block调用后：c=%@, c指向的地址=%p, c本身的地址=%p", c, c, &c);
    }
    
    /*
     c指针本身在block定义中和外面不是一个，但是c指向的地址一直保持不变。
     1. block定义前：c指向的地址在堆中， c指针本身的地址在栈中。
     2. block定义内部：c指向的地址再堆中， c指针本身的地址在堆中（c指针本身和外面的不是一个，但是指向的地址和外面指向的地址是一样的）。
     3. block定义后：c不变，c指向的地址在堆中， c指针本身的地址在栈中。
     4. block调用后：c不变，c指向的地址在堆中， c指针本身的地址在栈中。
     */

    {
        NSLog(@"\n--------------------block调用 指针并修改值---------------------\n");
        
        NSMutableString *d = [NSMutableString stringWithFormat:@"ddd"];
        NSLog(@"block定义前：d=%@, d指向的地址=%p, d本身的地址=%p", d, d, &d);
        void (^dBlock)() = ^{
            NSLog(@"block定义内部：d=%@, d指向的地址=%p, d本身的地址=%p", d, d, &d);
            d.string = @"dddddd";
        };
        NSLog(@"block定义后：d=%@, d指向的地址=%p, d本身的地址=%p", d, d, &d);
        dBlock();
        NSLog(@"block调用后：d=%@, d指向的地址=%p, d本身的地址=%p", d, d, &d);
    }
    
    /*
     d指针本身在block定义中和外面不是一个，但是d指向的地址一直保持不变。
     在block调用后，d指向的堆中存储的值发生了变化。
     */
    
    {
        NSLog(@"\n--------------------block调用 __block修饰的指针---------------------\n");
        
        __block NSMutableString *e = [NSMutableString stringWithFormat:@"eee"];
        NSLog(@"block定义前：e=%@, e指向的地址=%p, e本身的地址=%p", e, e, &e);
        void (^eBlock)() = ^{
            NSLog(@"block定义内部：e=%@, e指向的地址=%p, e本身的地址=%p", e, e, &e);
            e = [NSMutableString stringWithFormat:@"new-eeeeee"];
        };
        NSLog(@"block定义后：e=%@, e指向的地址=%p, e本身的地址=%p", e, e, &e);
        eBlock();
        NSLog(@"block调用后：e=%@, e指向的地址=%p, e本身的地址=%p", e, e, &e);
    }
    
    /*
     从block定义内部使用__block修饰的e指针开始，e指针本身的地址由栈中改变到堆中，即使出了block，也在堆中。
     在block调用后，e在block内部重新指向一个新对象,e指向的堆中的地址发生了变化。
     */
    
    {
        NSLog(@"\n--------------------block调用 retain cycle---------------------\n");
        
        View *v = [[View alloc] init];
        v.tag = 1;
        v.frame = CGRectMake(100, 100, 100, 100);
        [self.view addSubview:v];      //self->view->v
        void (^block)() = ^{
            v.backgroundColor = [UIColor orangeColor]; //定义内部：block->v
        };
        v.block = block;    //v->block
        block();   
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            //预计3秒后释放v对象。
            [v removeFromSuperview];
        });
    }
    
    /*
     结果：
     不会输出 dealloc.
     */
    
    /*
     流程：
     1. self->view->v
     2. block定义内部：block->v 因为block定义里面调用了v
     3. v->block
     
     结论：
     引起循环引用的是block->v->block，切断其中一个线即可解决循环引用，跟self->view->v这根线无关
     */
    
    {
        NSLog(@"\n--------------------block调用self---------------------\n");
        
        View *v = [[View alloc] init];
        v.tag = 2;
        v.frame = CGRectMake(100, 220, 100, 100);
        [self.view addSubview:v];      //self->view->v
        void (^block)() = ^{
            self.view.backgroundColor = [UIColor redColor]; //定义内部：block->self
            _count ++;   //调用self的实例变量，也会让block强引用self。
            
        };
        v.block = block;    //v->block
        block();
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            //预计3秒后释放self这个对象。
            AppDelegate *appDelegate = [UIApplication sharedApplication].delegate;
            appDelegate.window.rootViewController = nil;
        });
    }

    /*
     结果：
     不会输出 dealloc.
     */
    
    /*
     流程：
     1. self->view->v
     2. v->block
     3. block->self 因为block定义里面调用了self
     
     结论：
     在block内引用实例变量，该实例变量会被block强引用。
     引起循环引用的是self->view->v->block->self，切断一个线即可解决循环引用。
     */
```


###总结

1. 在block内部，栈是红灯区，堆是绿灯区。
2. 在block内部使用的是将外部变量的拷贝到堆中的（基本数据类型直接拷贝一份到堆中，对象类型只将在栈中的指针拷贝到堆中并且指针所指向的地址不变。）
3. __block修饰符的作用：是将block中用到的变量，拷贝到堆中，并且外部的变量本身地址也改变到堆中。
4. 循环引用：分析实际的引用关系，block中直接引用self也不一定会造成循环引用。
	
