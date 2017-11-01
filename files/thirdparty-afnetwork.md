## AFNetworking

在 AFHTTPSessionManager 中

1. requestSerializer 生成request请求，对于不同的request增加不同的参数， 增加了AFRequestSerializerDelegate，可以设置参数。
2. responseSerializer 写在initWithCoder和encodeWithCoder， 可以归档请求
3. 内部有个数组装代理，每个代理负责每个请求task



http://52.27.144.7:8080/api/v2/user/videos/favorite?tz=Asia%252FShanghailocale=%25E4%25B8%25AD%25E6%2596%2587%2520(%25E4%25B8%25AD%25E5%259B%25BD)from=com.jiandaola.dailycastvn=2.1.5lc=googleplayudid=744e21f38bce4ca3992652


http://52.27.144.7:8080/api/v2/user/videos/favorite?tz=Asia%252FShanghai&locale=%25E4%25B8%25AD%25E6%2596%2587%2520(%25E4%25B8%25AD%25E5%259B%25BD)&from=com.jiandaola.dailycast&vn=2.1.5&lc=googleplay&udid=744e21f38bce4ca399265261039a31d59cdcf2cc&sdk=22&vc=143