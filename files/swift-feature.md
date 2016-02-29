## Swift 特点

>Swift has a nifty feature that allows us to skip the `respondsToSelector:` dance and just make the method invocation optional. If the method doesn't exist, then pretend it returned nil or void. You may not believe it, but this is one case where Swift makes it easier to be dynamic.

上述大致意思是swift不用判断是否有某个方法，如果方法不存在将会返回nil或void。



