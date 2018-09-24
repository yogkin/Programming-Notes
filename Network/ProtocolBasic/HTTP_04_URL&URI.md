# URI & URL

---
## 1 RFC

RFC是征求意见稿（英语：`Request For Comments`，缩写为RFC），是由互联网工程任务组（IETF）发布的一系列备忘录。文件收集了有关互联网相关信息，以及UNIX和互联网社区的软件文件，以编号排定。目前RFC文件是由互联网协会（ISOC）赞助发行。

RFC始于1969年，由斯蒂芬·克罗克用来记录有关ARPANET开发的非正式文档，最终演变为用来记录互联网规范、协议、过程等的标准文件。基本的互联网通信协议都有在RFC文件内详细说明。RFC文件还额外加入许多的论题在标准内，例如对于互联网新开发的协议及发展中所有的记录。

---
## 2 URI

在电脑术语中，**统一资源标识符**（`Uniform Resource Identifier`，或URI)是一个**用于标识某一互联网资源名称的字符串**。 该种标识允许用户对网络中（一般指万维网）的资源通过特定的协议进行交互操作。URI的最常见的形式是**统一资源定位符（URL）**，经常指定为非正式的网址。更罕见的用法是统一资源名称（URN），其目的是通过提供一种途径。用于在特定的命名空间资源的标识，以补充网址。

### URI与URL和URN的关系

URI可被视为定位符（URL），名称（URN）或两者兼备。

- 统一资源名（URN）如同一个人的名称，URN定义某事物的身份。
- 而统一资源定位符（URL）代表一个人的住址，URL提供查找该事物的方法。

>用于标识唯一书目的ISBN系统是一个典型的URN使用范例。例如，ISBN 0-486-27557-4( urn:isbn:0-486-27557-4 )无二义性地标识出莎士比亚的戏剧《罗密欧与朱丽叶》的某一特定版本。为获得该资源并阅读该书，人们需要它的位置，也就是一个URL地址。在类Unix操作系统中，一个典型的URL地址可能是一个文件目录，例如`file:///home/username/RomeoAndJuliet.pdf`。该URL标识出存储于本地硬盘中的电子书文件。因此，URL和URN有着互补的作用。

![](index_files/05ebc02a-3fc7-4bcb-b7ba-cb65c956d567.png)


### RFC对URI所做的定义

- Uniform：规定统一的格式可方便处理多种不同类型的资源，而不同根据上下文环境来识别资源指定的访问方式，此外加入新的协议方案(如：http/ftp)也更加容易
- Resource:资源的定义是**刻标识的任何东西**，除了文档文件，图像或者服务(如当天天气预报)等能够区别于其他类型的，全都可以作为资源，另外，资源不仅可以是单一的，也可以是多少的集合体
- Identifier：表示可标识的对象，也被称为标识符。

综上所述：**URI就是由某个协议方案表示的资源的定位标识符**，协议方案是指访问资源所使用的协议类型名称，比如使用HTTP协议时，协议方案就是http。

>URL 是 URI 的子集。任何东西，只要能够唯一地标识出来，都可以说这个标识是 URI 。如果这个标识是一个可获取到上述对象的路径，那么同时它也可以是一个 URL ；但如果这个标识不提供获取到对象的路径，那么它就必然不是 URL 。

URI用字符串标识某一互网资源，而URL表示资源的地点(互联网上所处的位置)，所以URL是URI的子集。

RFC3986规定URI的通用语法示例如下：

```
 ftp://ftp.is.co.za/rfc/rfc1808.txt
 http://www.ietf.org/rfc/rfc2396.txt
 ldap://[2001:db8::7]/c=GB?objectClass?one
 mailto:John.Doe@example.com
 news:comp.infosystems.www.servers.unix
 tel:+1-816-555-1212
 telnet://192.0.2.16:80/
 urn:oasis:names:specification:docbook:dtd:xml:4.1.2
```

URI是个纯粹的语法解构，包含用来指定Web资源的各种组成成分，URL是URI的一个特例，其包含了用于定位一个资源需要的足够的信息，比如URI:`mailto:cay@horstmann.com`就不是URL，无法通过它定位一个资源。


### URI格式

URI的绝对格式如下：
```
 [scheme:]scheme-specific-part[#fragment]
 http://user:pass@www.example.jp:80/dir/index.html?uid=1#ch1

 http表示协议方案名
 user:pass表示登陆认证信息
 @www.example.jp表示服务器地址
 80表示服务器端口号
 /dir/index.html表示带层次的文件目录
 uid=1表示查询的字符串
 ch1片段标识符
```
- 登陆信息(认证)：用户名和密码作为服务器端获取资源时必要的登陆信息(身份认证)，可选项
- 服务器地址：使用绝对URI必须指定待访问的服务器地址，地址可以是DNS可解析的域名也可以是ip地址
- 服务器端口号：指定服务器连接的网络端口号，可选项，没有指定则使用默认的端口号
- 带层次的文件路径：指定服务器上的文件来定位特指的资源，与Unix系统的文件目录接口类似
- 查询字符串：针对已指定的文件路径内的资源，可使用查询字符串传入任意参数，可选项
- 片段标识符：使用片段标识符通常可标记处已经获去资源中的子资源(文档内的某一个位置)，但是在RFC中并没有明确规定其使用方法，可选项

### Java中的URI实现

```
URI uri = URI.create("https://user:pass@zh.wikipedia.org:80/zh/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6?abc=bcd#safe=strict&q=uri");
System.out.println("uri.getScheme() = " + uri.getScheme());
System.out.println("uri.getUserInfo() = " + uri.getUserInfo());
System.out.println("uri.getAuthority() = " + uri.getAuthority());
System.out.println("uri.getHost() = " + uri.getHost());
System.out.println("uri.getPort() = " + uri.getPort());
System.out.println("uri.getPath() = " + uri.getPath());
System.out.println("uri.getQuery() =  " + uri.getQuery());
System.out.println("uri.getFragment() = " + uri.getFragment());

System.out.println("--------------------------------------------------------------------------");
System.out.println("uri.getRawUserInfo() = " + uri.getRawUserInfo());
System.out.println("uri.getRawAuthority() = " + uri.getRawAuthority());
System.out.println("uri.getRawPath() =  " + uri.getRawPath());
System.out.println("uri.getRawQuery() =  " + uri.getRawQuery());
System.out.println("uri.getRawFragment() =   " + uri.getRawFragment());
System.out.println("uri.getRawSchemeSpecificPart() = " + uri.getRawSchemeSpecificPart());
```

打印结果为：

```
uri.getScheme() = https
uri.getUserInfo() = user:pass
uri.getAuthority() = user:pass@zh.wikipedia.org:80
uri.getHost() = zh.wikipedia.org
uri.getPort() = 80
uri.getPath() = /zh/统一资源标志符
uri.getQuery() =  abc=bcd
uri.getFragment() = safe=strict&q=uri
--------------------------------------------------------------------------
uri.getRawUserInfo() = user:pass
uri.getRawAuthority() = user:pass@zh.wikipedia.org:80
uri.getRawPath() =  /zh/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6
uri.getRawQuery() =  abc=bcd
uri.getRawFragment() =   safe=strict&q=uri
uri.getRawSchemeSpecificPart() = //user:pass@zh.wikipedia.org:80/zh/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6?abc=bcd
```

由打印的信息可以清晰的看出Uri中的各个定义所表示的部分。


---
## 3 URL

 URL既**统一资源定位符**（或称**统一资源定位器/定位地址、URL地址等**，英语：Uniform / Universal Resource Locator，常缩写为URL）

统一资源定位符的标准格式如下：

```
 协议类型://服务器地址（必要时需加上端口号）/路径/文件名
```

### Java中的URL实现

```
URL url = new URL("http://mrfu.me/2016/02/28/Glide_Series_Roundup");
System.out.println("url.getProtocol() = " + url.getProtocol());
System.out.println("url.getUserInfo() = " + url.getUserInfo());
System.out.println("url.getAuthority() = " + url.getAuthority());
System.out.println("url.getHost() = " + url.getHost());
System.out.println("url.getPort() = " + url.getPort());
System.out.println("url.getDefaultPort() = " + url.getDefaultPort());
System.out.println("url.getPath() = " + url.getPath());
System.out.println("url.getQuery() =  " + url.getQuery());
System.out.println("url.getFile() = " + url.getFile());
System.out.println("url.getRef() = " + url.getRef());
System.out.println("url.getContent() = " + url.getContent());
```

打印结果为：

```
url.getProtocol() = http
url.getUserInfo() = null
url.getAuthority() = mrfu.me
url.getHost() = mrfu.me
url.getPort() = -1
url.getDefaultPort() = 80
url.getPath() = /2016/02/28/Glide_Series_Roundup
url.getQuery() =  null
url.getFile() = /2016/02/28/Glide_Series_Roundup
url.getRef() = null
Disconnected from the target VM, address: '127.0.0.1:55167', transport: 'socket'
url.getContent() = sun.net.www.protocol.http.HttpURLConnection$HttpInputStream@18e8568
```

### URL编码

一般来说，UR L只能使用英文字母、阿拉伯数字和某些标点符号，不能使用其他文字和符号。比如，世界上有英文字母的网址`"http://www.abc.com"`。这是因为网络标准RFC 1738做了硬性规定：**只有字母和数字`[0-9a-zA-Z]`、一些特殊符号`"$-_.+!*'(),`以及某些保留字是安全字符，才可以不经过编码直接用于URL**。这意味着，如果URL中有汉字，就必须编码后使用。

下面列出了需要进行 URL 编码的地方：

- URL地址
- URL后面的参数
- 响应头和请求头中的参数
- 多部分请求体中的 part 描述参数

---
## 引用

- [关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)
- [维基百科：统一资源标识符](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)

