# Android 网络请求

---
## 1 API

从 Android 2.3 到现在一共经历了 3 套 API，分别是：

- HttpURLConnection（Java 官方）
- ApacheHttpClient （6.0 已经废弃）
- OkHttp （现在 Android 开发的标配）

其他的框架都是基于以上 API 的封装，现在 OkHttp + Retrofit + RxJava 可以说是非常完美的网络请求解决方案，可以应对大部分需求。

### 网络权限

永远都不要忘记，网络请求是需要权限的：

```xml
<uses-permission android:name="android.permission.INTERNET" /><-- 允许应用程序打开网络套接字 -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /><-- 允许应用程序访问网络连接信息 -->
```

---
## 2 离线支持

移动网络的特点：流量费用、不稳定、高延迟以及低带宽等情况。对于一个没有离线支持的 APP 来说，离开了网络，它将毫无是处，对移动应用的离线支持可以理解为应用在网络连接不稳定甚至不可以的情况下能够给用户提供一个较好的体验，一个能使用不能网络情况的应用应该包含以下特点：

- 在网络调用失败的情况下显示适当的错误信息并且添加重试操作
- 在 UI 上明确地显示出网络连接状态的信息（连接模式/离线模式）
- 在没有网络连接的情况下也允许用户进行数据查询与操作（离线数据访问）
- 在网络不可用的情况下禁用某些控件
- 支持在不同的网络连接条件下对应用进行测试

### 本地缓存

要使得应用程序在没有网络连接的情况下依然能够显示信息，而在连上网络的情况下需要刷新数据。需要在移动设备上对数据进行一定程度的持久化，并且通常需要保留一段时间。缓存方式通常包括以下几个：

- HTTP 缓存：使用 HTTP 首部控制缓存，只用于 GET 方式。
- 本地文件缓存：不依赖于 HTTP 客户端，发现一些 HTTP 客户端框架实现提供了对这种缓存方式的支持，我感觉本地文件缓存应该由开发者自行实现，HTTP 客户端实现应该只专注于实现 HTTP 规定的协议。
- SQLite 数据库缓存：简单的数据缓存不推荐使用数据库，但对于关系型数据就应该使用关系型数据库存储，比如聊天记录等。


### 缓存策略

对网络数据进行了缓存之后，就需要制定相应的策略，因为这时有两个数据源了（本地和缓存），如果对多个数据源进行组合就是缓存策略的事，一般包括以下策略：

- **网络优先**：总是尝试从服务器上获取数据，如果无法从服务器上获取，再转而从本地缓存中获取数据。适用于总是希望显示最近的、经过更新的信息的场景。
- **本地优先**：在一段指定的时间之内完全不会从网络上获取数据，而是通过本地缓存进行获取。这种策略存在一定的风险，即服务端的数据更新了，而本地依然`展示了过期的数据`，好处是用户体验往往更好，因为通常来说不会产生任何延迟。一般对这种场景，应该提供让用户进行`强制刷新`的按钮。
- **多个数据源混合**：这种方式在从服务端获取数据之前先从本地缓存中返回结果，然后可以在后台更新数据（等待服务端的通知或是在后台主动对服务进行查询），等待服务端影响之后对再本地的缓存数据进行刷新。这种机制能够在性能与用户体验之间取得一种良好的平衡，让用户不必等待数据加载。又能保证展示的是最新的数据。

HTTP 的条件验证：通过某种服务端缓存的支持，可以有效地弥补本地缓存的不足之处。正如 HTTP 缓存一样，当需要从服务器获取数据的时候，客户端可以通过发送一个`修订号`以确认该数据是否已经被更新了。服务端将检查客户端所发送的修订号是否与服务器上的数据一致，根据结果不同，或者通知客户端不必更新数据，或者将最新的数据返回。

### 其他缓存相关

- 缓存大小设置
- 缓存加密、数据库加密

---
## 3 网络管理与优化

网络管理：

- 通过 ConnectivityManager 获取当前网络状态。
- 使用 BroadcastReceiver 监听网络状态的变化，然后根据不同的状态及时调整网络策略。
- 不同的状态状态，自动调整不同的同步策略。

可配置的网络操作：

- 同步数据的频率，是否只在连接到 WiFi 才进行下载与上传操作。
- 不同网络状态下图片的加载策略，比如无图模式。

优化：

- 预加载
- 连接复用
- 数据压缩传输

### 网络操作队列

对于网络操作，可以有两种响应方式：积极的响应方式和消极的响应方式，比如一个点赞，不管当前网络是否可能，都积极的响应用户的操作，如果当前网络可用则立即进行网络请求，如果网络不可以，则将请求加入请求队列，等待网络可用时再进行网络操作。并将最后的请求结果反馈到 UI 上（UI 响应往往不依赖于具体的回调，而是基于数据变更的通知）。某些业务场景下，甚至可以对网络队列做持久化。以防止应用奔溃导致队列丢失。

这样做可以带来很好的用户体验，实现期望却更加复杂，可以针对特定的常见做特定的优化。


---
## 4 HTTPS

### HTTPS 简述

HTTPS 是安全的 HTTP 通信，在应该层和传输层之间添加了一层 SSL(Secure Socket Layer)，SSL 负责对数据进行加密和解密，为了提高网站的安全性，一般会在比较敏感的部分页面采用https传输，比如注册、登录、控制台等。像Gmail、网银、icloud等则全部采用 https传输。

如何正确的实施 HTTPS：

- 尽量申请权威的 CA 证书（付费的）。
- 如果使用自签名的证书，一定要做好证书校验，切忌信任所有证书。
- 采用双向认证

业界就发生了很多关于证书验证不严格的问题，具体参考 [Android 证书信任问题与大表哥](http://www.91ri.org/12540.html)。


### TLS 1.2

Android API 16 开始支持 TLS 1.2，到了 API 20 才默认开启 TLS 1.2。对于那些使用 TLS 1.2 的服务器，在 API 16-20 的 Android 系统中，需要手动开启 TLS 1.2，参考 [/okhttp/issues/2372](https://github.com/square/okhttp/issues/2372)。

---
## 5 下载器

下载区别于一般的 HTTP 请求，往往 I/O 时间较长，涉及到任务管理、断点续传、任务持久化等逻辑处理，可以把下载功能单独封装成一个功能完备的框架，这样我们可以更加专注地实现，一般下载器应该具备如下特性：

- 任务管理，比如最大并发数、任务调度
- 任务持久化，一般使用数据库
- 断点续传，需要服务器支持范围请求
- 良好的 API 设计，提供下载状态、进度监听

可以参考 github 上优秀的开源项目。

---
## 引用

HttpURLConnection 和 Apache HttpClient

- [关于Client与Connection](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)
- [Apache HttpClient](http://hc.apache.org/downloads.cgi)
- [LiteHttp](https://github.com/litesuits/android-lite-http)
- [Android中HttpURLConnection使用详解](http://blog.csdn.net/iispring/article/details/51474529)


网络优化

- [为移动应用提供离线支持](http://www.infoq.com/cn/articles/mobile-apps-offline-support)
- [Android网络操作和优化相关](http://blog.csdn.net/sdkfjksf/article/details/51645315)

https

- [Android证书信任问题与大表哥](http://www.91ri.org/12540.html)
- [Android Https相关完全解析 当OkHttp遇到Https](http://blog.csdn.net/lmj623565791/article/details/48129405)
- [android 平台ssl 单双向验证](http://blog.csdn.net/hfeng101/article/details/10163627)
- [Android中SSL通信中使用的bks格式证书的生成](https://blog.csdn.net/bailyzheng/article/details/54313356)
- [HTTPS 理论基础及其在 Android 中的最佳实践](http://blog.csdn.net/iispring/article/details/51615631)
- [Android App 安全的HTTPS 通信](https://yq.aliyun.com/articles/64810?spm=5176.8067842.tagmain.41.0LO1b6)
- [Android HTTPS正传](https://www.jianshu.com/p/458c0ba92026)

防抓包

- [漫画：App 防止 Fiddler 抓包小技巧！](http://vlambda.com/wz_xby9j2HnOT.html)
