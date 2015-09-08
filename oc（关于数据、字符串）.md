### OC（关于数据、字符串）

##### URL 中有汉字：
<code>
[urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
</code>


##### NSJSONSerializationw
<code>
NSDictionary *exception = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingAllowFragments error:&error];
</code>

1. **NSJSONReadingMutableContainers**：返回可变容器，NSMutableDictionary或NSMutableArray。

2. **NSJSONReadingMutableLeaves**：返回的JSON对象中字符串的值为NSMutableString，目前在iOS 7上测试不好用，应该是个bug，参见：

3. **NSJSONReadingAllowFragments**：允许JSON字符串最外层既不是NSArray也不是NSDictionary，但必须是有效的JSON Fragment。例如使用这个选项可以解析 @“123” 这样的字符串。

<code>
NSData *jsonData = [NSJSONSerialization dataWithJSONObject:parameters options:NSJSONWritingPrettyPrinted error:&error];
</code>

**NSJSONWritingPrettyPrinted**：的意思是将生成的json数据格式化输出，这样可读性高，不设置则输出的json字符串就是一整行。