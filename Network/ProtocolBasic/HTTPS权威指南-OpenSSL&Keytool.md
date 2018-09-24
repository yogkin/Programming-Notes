# OpenSSL 和 keytool（未完成）

---
## 1 OpenSSL

在计算机网络上，OpenSSL是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连线者的身份。这个包广泛被应用在互联网的网页服务器上——维基百科。

OpenSSL 是一个健壮的、商业级和功能齐全的`传输层安全性 (TLS) 和安全套接字层 (SSL) 协议`实现工具包。它也是一个`通用的加密库`。它还是一个开源（Apache开源许可证授权）程序的套件、这个套件有三个部分组成：

- 一是 libcryto，这是一个具有通用功能的加密库，里面实现了众多的加密库；
- 二是 libssl，这个是实现ssl机制的，它是用于实现TLS/SSL的功能；
- 三是 openssl，是个多功能命令行工具，它可以实现加密解密，甚至还可以当CA来用，可以创建证书、吊销证书。

openssl 的命令格式为：`openssl command [ command_opts ] [ command_args ]`，查看 openssl 信息和支持的命令：

```
显示版本和编译参数：>openssl version -a
查看支持的标准命令：>openssl list-standard-commands
查看支持的加密命令：>openssl list-cipher-commands
查看支持的信息摘要命令：>list-message-digest-commands
查看密码组合列表：>openssl ciphers
```

从 openssl 命令的使用方式可以看出，openssl 包含很多的子命令，比如：

- `openssl ca` 命令用于 CA 的管理。
- `openssl genrsa` 命令用于生成RSA私钥。
- `openssl rsa` 命令用于 RSA 数据管理。
- `openssl req` 命令主要以 `PKCS＃10` 格式创建和处理证书请求。例如，它还可以创建自签名证书以用作根 CA。
- `openssl x509` 命令是一个多用途证书实用程序。它可用于显示证书信息，将证书转换为各种表单，像“迷你CA”一样签署证书请求或编辑证书信任设置。
- `openssl crl` 命令用于管理 CRL 列表。
- `openssl pkcs7` 命令用于管理 pkcs7 数据。
- `openssl crl2pkcs7` 用于CRL和PKCS#7之间的转换。
- `openssl pkcs12` 命令允许创建和解析PKCS＃12文件（有时称为PFX文件）。 PKCS＃12文件被包括 Netscape，MSIE和 MS Outlook在内的多个程序使用。


---
## 2 keytool

keytool是java提供一个有效的安全钥匙和证书的管理工具，可以用来创建数字证书。

---
## 3 证书相关概念于格式说明

证书由两种：`CA 证书`和`自签名证书`，CA 证书是收费的，通过 OpenSSL 可以创建自签名证书，关于证书需要掌握如下概念：

### X.509 的证书编码格式：PEM 和 DER

X.509 有不同的编码格式，比如 PEM 和 DER 。

- DER(Distinguished Encoding Rules)：即可辨别编码规则，二进制的格式，不可读，Java 和 Windows 服务器偏向于使用这种编码格式。
- PEM(Privacy Enhanced Mail)：即 (隐私增强型电子邮件)，DER编码的证书再进行Base64编码的数据存放在`"-----BEGIN CERTIFICATE-----"`和`"-----END CERTIFICATE-----"`之中，Apache和类Unix服务器偏向于使用这种编码格式。

```
查看 PEM 格式证书的信息:openssl x509 -in certificate.pem -text -noout
查看 DER 格式证书的信息:openssl x509 -in certificate.der -inform der -text -noout
```

PEM 和 DER 本质上是编码格式，但也用于扩展名。

### X.509 相关扩展名

X.509 有多种常用的扩展名。不过其中的一些还用于其它用途，就是说具有这个扩展名的文件可能并不是证书，比如说可能只是保存了私钥。需要注意的是，虽然 X.509 有 PEM 和 DER 这两种编码格式，但文件扩展名并不一定就叫”PEM”或者”DER”，不同的扩展名有不同的含义，内容也有差别，但大多数都能相互转换编码格式。

- **CRT**：常见于`类UNIX`系统，有可能是 PEM 编码，也有可能是 DER 编码，大多数应该是 PEM 编码。
- **CER**：常见于Windows系统，有可能是 PEM 编码，也有可能是 DER 编码，大多数应该是 DER 编码。
- **KEY**：单独存放的pem格式的密钥，它不是证书，编码可能是 PEM，也可能是 DER。
- **CSR**：Certificate Signing Request，证书签名请求，不是证书，而是获得签名证书的申请，一般是向权威证书颁发机构提交CSR，CSR包含了证书持有人的信息，比如国家、邮件、域名等信息，CSR 需要通过一个私钥生成。
- **PKCS1 - PKCS12**：PKCS 即 Public Key Cryptography Standards，公钥加密标准，此一标准的设计与发布皆由RSA信息安全公司所制定。一般存储为 `*.pN`，而 `*.p21`是包含证书的同时可能还有带密码保护的私钥。
- **PFX**：是 `PKCS#12` 之前的格式，是 Windows IIS（互联网信息服务 Internet Information Services） 的实现 ，在`类UNIX`系统上，一般 CRT 和 KEY 是分开存放在不同文件中的，但 Windows 的 IIS 则将它们存在一个 PFX 文件中，PFX 通常会有一个提取密码操作，需要提供提取密码才能把里面的信息读取出来的话，PFX使用的是 DER 编码。
- **PEM**：证书或密钥的 Base64 文本存储格式，可以单独存储证书和密钥，也可以同时存储证书和密钥。
- **DER**：证书或密钥的二进制存储格式
- **CRL**：Certificate Revocation List，即证书吊销列表

相关命令：

```
# 查看 KEY
openssl rsa -in key_name.key -text -noout
openssl rsa -in key_name.key -text -noout -inform der # DER格式

# 查看 CSR
openssl req -noout -text -in csr_name.csr
openssl req -noout -text -in csr_name.csr -inform der # DER格式

# 生成pfx，其中 CACert.crt 是 CA 根证书，没有则省略 -certfile CACert.cr
openssl pkcs12 -export -in certificate.crt -inkey privateKey.key -out certificate.pfx -certfile CACert.crt
# PFX转换为PEM编码
openssl pkcs12 -in pfx_name.pfx -out pem_name.pem -nodes

# PEM 转 DER
openssl x509 -in cert.crt -outform der -out cert.der # 证书
openssl rsa -in pem_key.key -outform der -out der_key.key # 密钥
openssl req -in pem_csr.csr -outform der -out der_csr.csr # 证书签名请求
# DER 转 PEM
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
openssl rsa -in der_key.key -inform der -outform pem -out pem_key.key
openssl req -in der_csr.csr -inform der -outform pem -out pem_csr.csr
```

### JKS

JSK 即 JavaKeysotre，这是 Java 发明的证书格式，本身于 OpenSSL 关系不大。


---
## 4 创建自签名证书及相关命令

生产环境一般使用付费的 CA 证书，创建自签名证书可以用于测试和学习。在此之前要明确证书的几个角色：

- CA 证书：包括 CA 的私钥和CA 的证书，如果使用付费的证书那么这些东西在权威机构那边，不需要我们生成，如果是自签名的证书，那么我们需要生成 CA 的私钥和CA 的证书来充当证书颁发机构。
- 服务端证书：包括 服务端的私钥和服务端的证书。
- 客户端证书：包括 客户端的私钥和客户端的证书。

---
### 4.1 创建自签名证书

#### 生成密钥

方式 2：

```
# 生成私钥(没有加密)，2048 指 RSA 密钥长度位数
openssl genrsa -out private.key 2048
# 导出公钥
openssl rsa -in private.key -pubout -out public.key
```

方式 2：

```
# 生成私钥，使用 3des 进行加密，该命令会要求输入密码
openssl genrsa -des3 -out private.key 1024
# 从秘钥中移除密码，即回到第一种方式生成的私钥，该命令会要求输入密码
cp private.key private.key.org
openssl rsa -in private.key.org -out private.key
```

使用加密的私钥还是不加密的私钥？如果使用加密的私钥，则需要在使用过程中输入密码。这对于一些特定场景自动化来说会带来一些问题。如果使用不加密的私钥，则如果私钥泄漏给其他人了，对应的证书也需要被吊销，以避免带来危害。

#### 生成 CSR (Certificate Signing Request)

CSR 的意思就是证书签名请求，通常我们将 CSR 发送到 CA，CA 会做身份验证，并颁发签名证书。也就是说证书需要 CA私钥对申请者提交的 CSR 进行签名才能生成。

现在模拟一个场景，Server A 要申请一个证书，那么步骤是：

- 使用 openssl 生成一个密钥，这是 Server A 的密钥。
- Server A 通过它的密钥生成一个 CSR，然后提交给 CA。
- CA 使用自己的私钥和证书对 Server A 提交的 CSR 进行签名后生成证书，颁发给 Server A。

Server A 通过它的密钥生成一个 CSR 时需要输入相关信息，这些是证书的 X.509 属性。其中一个提示是 `Common Name (e.g., YOUR name)`，这个非常重要，这一项应该填入 FQDN(Fully Qualified Domain Name，完全限定域名)，这个 FQDN 会被 SSL 保护。如果要被保护的网站是 `https://www.ztiany.com`，那么输入 `www.ztiany.com`。

生成过程：

```
# 生成服务端的私钥
openssl genrsa -out server.key 2048
# 根据私钥生成 CSR
openssl req -new -key server.key -out server.csr
        Country Name (2 letter code) [AU]:CN
        State or Province Name (full name) [Some-State]:GD
        Locality Name (eg, city) []:SZ
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:it
        Organizational Unit Name (eg, section) []:android
        Common Name (e.g. server FQDN or YOUR name) []:www.fakeca.com
        Email Address []:god@ca.com
        A challenge password []:ca123456
        An optional company name []:
```

#### 生成 CA 证书

知道了怎么生成私钥和 CSR ，那么生成证书就相对简单了。

首先模拟一个 CA，我们需要生成 CA 的私钥和 CA 的证书。

```
# 生成 CA 的私钥
 openssl genrsa -out ca.key 2048
# 生成 CA 证书的 csr
 openssl req -new -key ca.key -out ca.csr
# 使用 CA 的私钥给 ca.csr 签名生成 CA 证书
 openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt

# 以上三个步骤可以可并为
 openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout ca.key -out ca.crt

# 生成证书后可以查看自签名CA证书
 openssl x509 -text -in ca.cert
```

#### 颁发证书

颁发证书就是用 CA 的秘钥给其他人签名证书，该过程输入需要证书请求、CA的私钥、CA的证书三部分，输出的是签名好的证书，证书将会交给申请者。

模拟 Server 向 CA 提交 CSR，然后得到证书：
```
# 生成 Server 密钥
 openssl genrsa -out server.key 2048
# Server 生成请求:
 openssl req -new -key server.key -out server.csr
        Country Name (2 letter code) [AU]:CN
        State or Province Name (full name) [Some-State]:GD
        Locality Name (eg, city) []:SZ
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:s1
        Organizational Unit Name (eg, section) []:s1
        Common Name (e.g. server FQDN or YOUR name) []:www.s1.com
        Email Address []:server@s1.com
        A challenge password []:s1123456
        An optional company name []:
# CA 给 Server 签发证书:
 openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

# 以 PEM 格式输出证书的 SubjectPublicKeyInfo 块
 openssl x509 -in server.crt -pubkey
# 验证server证书
 openssl verify -CAfile ca.crt server.crt
```

相关参数说明可以查看 openssl 官方文档。


### 4.2 其他证书操作(todo)

- 生成证书链多级证书
- 吊销证书
- 证书转换相关命令
- openssl.cnf 扩展

---
## 引用

OpenSSL

- [OpenSSL Command-Line HOWTO](https://www.madboa.com/geek/openssl/)
- [openssl-wiki](https://wiki.openssl.org/index.php/Command_Line_Utilities)
- [openssl 系列](http://www.cnblogs.com/f-ck-need-u/p/7048359.html#auto_id_3)
- [OpenSSL命令详解（一）——标准命令](https://blog.csdn.net/scuyxi/article/details/54884976)
- [OpenSSL命令详解（二）——摘要算法、签名、验签](https://blog.csdn.net/scuyxi/article/details/54884976)
- [那些证书相关的玩意儿(SSL, X.509, PEM, DER, CRT, CER, KEY, CSR, P12等)](https://mp.weixin.qq.com/s?__biz=MzAxMjY5NDU2Ng==&mid=2651851825&idx=1&sn=1e6861d961442dab239ccfa29a63fc7e&chksm=80494978b73ec06e46c90c152a64871bbe765c34c8585aab58f13c52e53221471f0064d8976d&mpshare=1&scene=1&srcid=0807InpYeOF4ucKgsXVDbyng#rd)

keytool

- [常用的Java Keytool Keystore命令](https://www.chinassl.net/ssltools/keytool-commands.html)
- [java keytool 工具](https://blog.csdn.net/bailyzheng/article/details/80006344)
- [Java Keytool工具简介](https://blog.csdn.net/liumiaocn/article/details/61921014)

openssl 与 keytool

- [OpenSSL与KeyStore指令小集](https://blog.csdn.net/kimylrong/article/details/43525333)
- [数字证书管理工具openssl和keytool的区别](https://www.cnblogs.com/zhangshitong/p/9015482.html)

实战

- [OpenSSL 自建CA及签发证书](https://blog.csdn.net/scuyxi/article/details/54898870)
- [如何创建自签名的 SSL 证书](https://www.jianshu.com/p/e5f46dcf4664)：多级证书、证书吊销
- [基于 OpenSSL 自建 CA 和颁发 SSL 证书](http://seanlook.com/2015/01/18/openssl-self-sign-ca/)：nginx 配置
- [基于 OpenSSL 自建 CA 和颁发 SSL 证书](https://deepzz.com/post/based-on-openssl-privateCA-issuer-cert.html)：openssl.cof
- [使用 openssl 生成证书](https://www.cnblogs.com/littleatp/p/5878763.html)：证书转换