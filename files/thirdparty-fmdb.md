## thirdparty-FMDB
-----

[这里唐巧的有关FMDB技术博客](http://blog.devtang.com/blog/2012/04/22/use-fmdb/)

1. iOS使用  FMDB 的sql语句中 用 ‘?’ 来注入参数，  不要用格式化符(%@,%d等)拼接sql语句
2. sql语句中注入的整数，浮点数等用 NSNumber封装。


```
[db executeUpdate:@"insert into test (a, b, c, d, e) values (?, ?, ?, ?, ?)" ,
         @"hi again'", // look!  I put in a ', and I'm not escaping it!
         [NSString stringWithFormat:@"number %d", i],
         [NSNumber numberWithInt:i],
         [NSDate date],
         [NSNumber numberWithFloat:2.2f]];

```
