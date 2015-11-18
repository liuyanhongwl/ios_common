## node.js 

### 一. 创建Hello World服务器

##### 步骤

- 引入 required 模块
- 创建服务器

```
var http = require('http');
http.createServer(function (request, response)
{
	//发送HTTP头部
	//HTTP 状态值: 200 : OK
	//内容类型
	response.writeHead(200, {'Content-Type':'text/plain'});

	//发送响应数据
	response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');

```

##### 解释
- 第一行请求（require）Node.js 自带的 http 模块，并且把它赋值给 http 变量。
- 接下来我们调用 http 模块提供的函数： createServer 。这个函数会返回 一个对象，这个对象有一个叫做 listen 的方法，这个方法有一个数值参数， 指定这个 HTTP 服务器监听的端口号。


### 二. 