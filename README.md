# 笔记迁移中（wiz --> github）...

说明：

- 笔记内容来源于读书、开发实践总结、阅读他人博客、翻译等。
- 笔记中的引用一般会标明出处并加上链接、如有纰漏，请联系我及时补充。
- 笔记难免存在错误，如有发现，请帮我指出，不甚感激。

如何解决问题?

- 切忌焦躁，静心思考
- 从日志找答案
- Debug
- 从源码分析
- Google
- StackOverflow

如何学习 GitHub 项目?

- 仔细阅读 README
- 查看 Wiki
- 查看 branch
- 使用 Chrome 相关插件，高效使用 Github。

---
## Android

### 基础组件

- [Application](Android/基础组件/Application.md)
- [Activity](Android/基础组件/Activity.md)
    - [上次发版我就改了一行代码！](http://blog.csdn.net/eclipsexys/article/details/53791818)
    - [利用 `<activity-alias>` 动态改变 App 桌面图标](http://yifeng.studio/2016/12/30/android-change-app-launcher-icon-dynamically/)
- [Service](Android/基础组件/Service.md)  
- [Intent](Android/基础组件/Intent.md)
- [IntentFlag](Android/基础组件/IntentFlag.md)
- [Notification](Android/基础组件/Notification.md)
- [PendingIntent](Android/基础组件/PendingIntent.md)
- [RemoteViews](Android/基础组件/RemoteViews.md)
- [桌面Widget](Android/基础组件/桌面Widget.md)

### Fragment

- [Fragment 学习资料](Android/Fragment/Fragment学习资料.md)
- [01-Fragment 入门](Android/Fragment/01-Fragment入门.md)
- [02-Fragment 使用](Android/Fragment/02-Fragment使用.md)
- [03-Fragment 回退栈](Android/Fragment/03-Fragment回退栈.md)
- [04-Fragment 转场动画](Android/Fragment/04-Fragment转场动画.md)
- [05-Fragment TabHost 使用](Android/Fragment/05-Fragment_TabHost.md)
- [06-Fragment 与 ViewPager](Android/Fragment/06-Fragment与ViewPager.md)
- [07-Fragment Dialog 使用](Android/Fragment/07-FragmentDialog使用.md)
- [08-Fragment Dialog 解析](Android/Fragment/08-FragmentDialog解析.md)
- [09-Fragment 状态丢失问题](Android/Fragment/09-Fragment状态丢失问题.md)
- [10-ActivityLifecycleCallbacks、FragmentLifecycleCallbacks、Activity 与 Fragment 生命周期交互](Android/Fragment/10-Fragment与其他组件生命周期交互.md)
- [11-XMl 中使用 Fragment](Android/Fragment/11-在xml中使用Fragment.md)
- [100-Fragment相关问题记录](Android/Fragment/100-Fragment相关问题记录.md)


---
## 网络协议

- **协议**：一系列相关协议的集合称为一个协议族的集合称为一个协议族，指定一个协议族的各种协议之间的相互关系并划分需要完成的任务的设计，称为协议族的体系结构或者参考模型。TCP/IP 是一个实现 Internet 体系结构的协议族。其来源于 ARPANET 参考模型。
- **应用程序编程接口**：无论是 P2P 或是客户机/服务器，都需要表述其所需的网络操作(比如建立一个连接，写入或读取数据)，这通常由主机操作系统使用一个网络应用程序编程接口(API)来实现，最流行的编程接口被称为套接字( Socket )或者 Berkeley 套接字，它最初由 LJFK93 开发。可以说 TCP/IP 是规范。而 Socket 是其在编程上的实现。
- [网络相关学习资料](计算机网络/网络通信基础/网络相关学习资料.md)

### 《图解密码技术》

- [《图解密码技术》第一部分](计算机网络/网络通信基础/图解密码技术-part01.md)

### HTTP

- [HTTP 概述](计算机网络/网络通信基础/HTTP_01_概述.md)
- [HTTP Content encoding 和 Transfer Encoding](计算机网络/网络通信基础/HTTP_02_Content_encoding_Transfer_Encoding.md)
- [HTTP 范围请求](计算机网络/网络通信基础/HTTP_03_范围请求.md)


### HTTPS

- [《HTTPS权威指南》-TLS/SSL](计算机网络/网络通信基础/HTTPS权威指南01-SSL&TLS.md)