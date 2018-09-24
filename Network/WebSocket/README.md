# WebSocket

---
## 1 WebSocket介绍

WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持。当然WebSocket不仅可以用于浏览器，还可以应用与移动端。

### 与 Http 的区别

已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？因为 HTTP 协议有一个缺陷：**通信只能由客户端发起**。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用 **轮询**，每隔一段时候，就发出一个询问，了解服务器有没有新的信息。轮询的效率低，非常浪费资源，WebSocket的出现就是用于解决这个问题的。

WebSocket协议是一种建立在TCP连接基础上的全双工通信的协议。建立在TCP的基础上，是一个**全双工通信**(客户端和服务端可以同时进行双向通信)协议

### WebSocket特点

1. 建立在 TCP 协议之上，服务器端的实现比较容易。
2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
3. 数据格式比较轻量，性能开销小，通信高效。
4. 可以发送文本，也可以发送二进制数据。
5. 没有同源限制，客户端可以与任意服务器通信。
6. 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。比如`ws://example.com:80/some/path`

### 应用

- 网页聊天室
- 移动端消息推送
- 直播中的弹幕

---
## 2 WebSocket框架

- [OkHttp](http://square.github.io/okhttp/)，OkHttp在 3.4.1 版本中提供了对WebScoket的支持，还提供了用于测试的模拟WebSocketServer库。另外可以参考 [OkHttp实现WebSocket分析](http://www.jianshu.com/p/13ceb541ade9)
- [AndroidAsyn](https://github.com/koush/AndroidAsync)：Android端Socket封装，支持WebSocket
- [autobahn-java](https://github.com/crossbario/autobahn-java)：Android和Java8的WebSocket、WAMP`(Web Application Messaging Protocol`)支持
- [java-webSocket](https://github.com/TooTallNate/Java-WebSocket)
- [Scarlet](https://github.com/Tinder/Scarlet)

---
## 引用

- [阮一峰：WebSocket 教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
- [Android最佳实践——深入浅出WebSocket协议](http://blog.csdn.net/sbsujjbcy/article/details/52839540)
- [WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)
- [websocket 协议解析](http://imweb.io/topic/59f93bdfb72024f03c7f49d9)