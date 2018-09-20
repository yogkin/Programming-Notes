# OkHttp 相关资料

---
## 1 OkHttp 介绍

首先OkHttp并不是市面上的一些网络框架，它和**HttpUrlConnection**和**HttpClient**一样，是一套基于Tcp/Ip协议，实现对底层Socket封装的一套网络编程API，OkHttp 具备如下特点：

- 自动重试
- 自动重定向
- 连接池复用
- okio高效io操作
- 拦截器(类似AOP)
- 支持SPDY，允许连接同一主机的所有请求分享一个socket。
- 如果SPDY不可用，会使用连接池减少请求延迟。
- 使用GZIP压缩下载内容，且压缩操作对用户是透明的。
- 利用响应缓存来避免重复的网络请求

当网络出现问题的时候，OKHttp会依然有效，它将从常见的连接问题当中恢复。如果你的服务端有多个IP地址，当第一个地址连接失败时，OKHttp会尝试连接其他的地址，这对IPV4和IPV6以及寄宿在多个数据中心的服务而言，是非常有必要的。OKHttp利用TLS的特性初始化新的连接，如果握手失败便退回到SSLV3。OKHttp的使用很简单。其2.0API拥有流畅的构建器和稳定性。它支持同步阻塞请求和异步回调请求。你可以试试OKHttp而不重写网络代码。`okhttp-urlconnection` 模块实现了都很熟悉 `java.net.HttpURLConnection` 的API，okhttp-apache 模块实现了 Apache 的 HttpClient 的 API。OKHttp 支持 Android2.3 以上，Java 支持最低版本 1.7。

---

## 2 相关资料

### 官网

- [官方WIKI](https://github.com/square/okhttp/wiki)
- [samples](https://github.com/square/okhttp/tree/master/samples)
- [模拟服务器](https://github.com/square/okhttp/tree/master/mockwebserver)

### 源码解析

- [ frodoking  OKHttp源码解析 ]( http://frodoking.github.io/2015/03/12/android-okhttp/ )
- [ OKHttp源码解析，缓存解析 ]( http://blog.csdn.net/chenzujie/article/details/46994073   )
- [OKHttp是怎样工作的](http://blog.csdn.net/marktheone/article/details/49402077)
- [OKHttp源码解析](http://frodoking.github.io/2015/03/12/android-okhttp/)
- [OkHttp3源码分析[Beta]](http://www.jianshu.com/p/aad5aacd79bf)
- [OKHttp 源码解析](http://frodoking.github.io/2015/03/12/android-okhttp/)
- [OkHttp设计解析](https://wingjay.com/2016/07/21/%E5%B8%A6%E4%BD%A0%E5%AD%A6%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%EF%BC%9AOkHttp-%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%AE%9E%E7%8E%B0okhttp/)

### 使用教程

- [部分WIKI翻译](http://www.cnblogs.com/ct2011/p/3997368.html)
- [如何高效的使用OkHttp](http://brucezz.github.io/articles/2016/02/21/effective-okhttp/)
- [学习okhttp wiki--HTTPS](http://www.cnblogs.com/yuanchongjie/p/4971665.html)
- [学习OkHttp wiki--Interceptors](http://www.cnblogs.com/yuanchongjie/p/4969326.html)
- [学习okhttp wiki--Connections.](http://www.cnblogs.com/yuanchongjie/p/4962310.html)
- [Http AUTH](http://blog.csdn.net/wwwsq/article/details/7255062)
- [okhttp中文注释](https://github.com/dunwen/okhttp)
- [Cookie](https://segmentfault.com/a/1190000004345545)
- [Understand-OkHttp](http://blog.piasy.com/2016/07/11/Understand-OkHttp/)
- [OkHttp翻译](http://www.jianshu.com/p/7621c6f63fd7)

### Sample

- [websocket-sample](https://github.com/fedepaol/websocket-sample)
- [Java WebSocket教程](http://colobu.com/2015/02/27/WebSockets-tutorial-on-Wildfly-8/)

### 拓展

- [鸿洋博客1](http://blog.csdn.net/lmj623565791/article/details/47911083)
- [ 鸿洋博客2](http://blog.csdn.net/lmj623565791/article/details/49734867)
- [上传多文件](http://blog.csdn.net/djk_dong/article/details/47861367   )
- [ 实现文件上传下载进度监听 ](http://blog.csdn.net/sbsujjbcy/article/details/48194701#comments)
- [PersistentCookieJar](https://github.com/franmontiel/PersistentCookieJar)

---
## 3 与OkHttp配合使用的库

参考：[Works-with-OkHttp](https://github.com/square/okhttp/wiki/Works-with-OkHttp)














