# Ztiany 的编程日志

>wiz --> github，笔记还在迁移中......

说明：

- 笔记内容来源于读书、开发实践总结、阅读他人博客、翻译等。
- 笔记中的引用一般会标明出处并加上链接、如有纰漏，请联系我及时补充。
- 笔记难免存在错误，如有发现，请帮我指出，不甚感激。
- 相关代码可以在这里找到：[Programming-Notes-Code](https://github.com/Ztiany/Programming-Notes-Code)

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

如何学习一门技术：

- 它是什么，自己能简单是叙述出来
- 它解决了什么问题，有什么意义
- 是如何解决问题的，内部如何实现
- 有何缺点，多角度分析
- 提问是有效学习一种方式，在一篇技术笔记后面加上自己的疑问，让学习变得主动

---
---
## Android

- [Android 开发常规技能总结](Android/README.md)

### 基础组件

- [Application](Android/Basic/Application.md)
- [Activity](Android/Basic/Activity.md)  
- [Service](Android/Basic/Service.md)  
- [Intent](Android/Basic/Intent.md)
- [IntentFlag](Android/Basic/IntentFlag.md)
- [Notification](Android/Basic/Notification.md)
- [PendingIntent](Android/Basic/PendingIntent.md)
- [RemoteViews](Android/Basic/RemoteViews.md)
- [桌面Widget](Android/Basic/桌面Widget.md)

### Fragment

- [Fragment 学习资料](Android/Fragment/Fragment学习资料.md)
- [Fragment相关问题记录](Android/Fragment/Fragment相关问题记录.md)  
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

### 开源框架

- [Dagger2 使用指南](Android/Libraries/Dagger2-使用指南.md)
- [okio 简介](Android/Libraries/okio-简介.md)
- [okio wiki](Android/Libraries/okio-wiki.md)

### NDK

- [01-NDK开发介绍](Android/NDK/01-NDK开发介绍.md)
- [02-ndkbuild](Android/NDK/02-ndkbuild.md)
- [03-cmake](Android/NDK/03-cmake.md)

### 优化

- [性能优化](Android/Optimization/Performance.md)

---
---
## 工具

- [Git 常用命令](Tools/Git基础命令.md)
- [AndroidStudio](Tools/Android.md)
- [IDEA&AndroidStudio插件](Tools/IDEA&AndroidStudio插件.md)
- [Windows 常用软件](Tools/Windows-Software-List.md)
- [Windows 操作技巧](Tools/Windows-operating-skill.md)

---
---
## 网络协议

- **协议**：一系列相关协议的集合称为一个协议族的集合称为一个协议族，指定一个协议族的各种协议之间的相互关系并划分需要完成的任务的设计，称为协议族的体系结构或者参考模型。TCP/IP 是一个实现 Internet 体系结构的协议族。其来源于 ARPANET 参考模型。
- **应用程序编程接口**：无论是 P2P 或是客户机/服务器，都需要表述其所需的网络操作(比如建立一个连接，写入或读取数据)，这通常由主机操作系统使用一个网络应用程序编程接口(API)来实现，最流行的编程接口被称为套接字( Socket )或者 Berkeley 套接字，它最初由 LJFK93 开发。可以说 TCP/IP 是规范。而 Socket 是其在编程上的实现。
- [网络相关学习资料](Network/ProtocolBasic/网络相关学习资料.md)

### 图解密码技术

- [《图解密码技术》第一部分](Network/ProtocolBasic/图解密码技术-part01.md)

### HTTP

- [HTTP 概述](Network/ProtocolBasic/HTTP_01_概述.md)
- [HTTP Content encoding 和 Transfer Encoding](Network/ProtocolBasic/HTTP_02_Content_encoding_Transfer_Encoding.md)
- [HTTP 范围请求](Network/ProtocolBasic/HTTP_03_范围请求.md)


### HTTPS

- [《HTTPS权威指南》-TLS/SSL](Network/ProtocolBasic/HTTPS权威指南01-SSL&TLS.md)
- [《HTTPS权威指南》-密码学简介](Network/ProtocolBasic/HTTPS权威指南02-密码学简介.md)
- [《HTTPS权威指南》-TLS 协议](Network/ProtocolBasic/HTTPS权威指南03-TLS协议.md)
- [《HTTPS权威指南》-PKI、证书](Network/ProtocolBasic/HTTPS权威指南04-PKI.md)
- [《HTTPS权威指南》-OpenSSL&Keytool](Network/ProtocolBasic/HTTPS权威指南-OpenSSL&Keytool.md)

---
---
## 工程构建

### Gradle

学习资料：

- [Gradle学习资料](BuildTools/Gradle/Gradle学习资料.md)

Gradle Android：

- [Android-Gradle 实践记录](BuildTools/Gradle/Android-Gradle实践记录.md)
- [Android-TransformAPI](BuildTools/Gradle/Android-TransformAPI.md)
- [Android-解决重复依赖的冲突](BuildTools/Gradle/Android-解决重复依赖的冲突.md)
- [Gradle-Windows 平台修改缓存位置](BuildTools/Gradle/Gradle-Windows平台修改缓存位置.md)
- [Android-《GradleForAndroid》 笔记](BuildTools/Gradle/Android-GradleForAndroid.md)
- [CI-Nexus 仓库](BuildTools/Gradle/CI-Nexus仓库.md)

Gradle 实战：

- [Gradle实战-00-概述](BuildTools/Gradle/Gradle实战-00-概述.md)
- [Gradle实战-01-项目自动化介绍](BuildTools/Gradle/Gradle实战-01-项目自动化介绍)
- [Gradle实战-02-Gradle基础](BuildTools/Gradle/Gradle实战-02-Gradle基础.md)
- [Gradle实战-03-构建java项目](BuildTools/Gradle/Gradle实战-03-构建java项目.md)
- [Gradle实战-04-构建web项目](BuildTools/Gradle/Gradle实战-04-构建web项目.md)
- [Gradle实战-05-GradleWrapper](BuildTools/Gradle/Gradle实战-05-GradleWrapper.md)
- [Gradle实战-06-Gradle构建脚本概要-1-Project](BuildTools/Gradle/Gradle实战-06-Gradle构建脚本概要-1-Project.md)
- [Gradle实战-07-Gradle构建脚本概要-2-Task](BuildTools/Gradle/Gradle实战-07-Gradle构建脚本概要-2-Task.md)
- [Gradle实战-08-Gradle依赖管理](BuildTools/Gradle/Gradle实战-08-Gradle依赖管理.md)
- [Gradle实战-09-扩展Gradle](BuildTools/Gradle/Gradle实战-09-扩展Gradle.md)
- [Gradle实战-10-Gradle命令行](BuildTools/Gradle/Gradle实战-10-Gradle命令行.md)

### Maven

- [Maven 入门](BuildTools/Maven/01-Maven入门.md)


---
---
## 编程语言

### Java

- [Java核心技术36讲](Languages/Java/Java核心技术36讲/Java核心技术36讲重点记录.md)，推荐订阅该[专栏](https://time.geekbang.org/column/intro/82)

### C语言

学习资料：

- [C学习资料](Languages/C/C学习资料.md)

编译与构建：

- [编译器-gcc学习](Languages/C/编译器-gcc学习.md)
- [构建工具-cmake简介](Languages/C/构建工具-cmake简介.md)  
- [构建工具-cmake学习](Languages/C/构建工具-cmake学习.md)
- [LLVM](https://llvm.org/) 项目是模块化和可重用的编译器和工具链技术的集合

C 语言基础与提高：

- [01-C语言简介](Languages/C/01-C语言简介.md)
- [02-C语言基础](Languages/C/02-C语言基础.md)
- [03-数据类型](Languages/C/03-数据类型.md)
- [04-sizeof和size_t类型](Languages/C/04-sizeof和size_t类型.md)
- [05-数组](Languages/C/05-数组.md)
- [06-函数](Languages/C/06-函数.md)
- [07-存储类别](Languages/C/07-存储类别.md)
- [08-字符串](Languages/C/08-字符串.md)
- [09-左值与右值](Languages/C/09-左值与右值.md)
- [10-指针](Languages/C/10-指针.md)
- [11-指针的大小](Languages/C/11-指针的大小.md)
- [12-输入和输出](Languages/C/12-输入和输出.md)
- [13-预处理](Languages/C/13-预处理.md)
- [14-头文件](Languages/C/14-头文件.md)
- [15-typedef](Languages/C/15-typedef.md)
- [16-结构体-枚举-联合体](Languages/C/16-结构体-枚举-联合体.md)
- [17-字节对齐与内存分配](Languages/C/17-字节对齐与内存分配.md)
- [20-文件读写](Languages/C/20-文件读写.md)
- [21-错误处理](Languages/C/21-错误处理.md)
- [22-动态内存管理](Languages/C/22-动态内存管理.md)
- [23-C语言标准库](Languages/C/23-C语言标准库.md)
- [24-C语言内存四区](Languages/C/24-C语言内存四区.md)
- [25-C 提高](Languages/C/25-C提高.md)
- [25-C 提高练习](Languages/C/25-C提高练习.md)

### Kotlin

学习资料：

- [Kotlin 学习资料](Languages/Kotlin/Kotlin学习资料.md)

Kotlin 基础：

- [Kotlin 基础语法](Languages/Kotlin/Kotlin01-基础语法.md)
- [Kotlin 面向对象](Languages/Kotlin/Kotlin02-面向对象.md)
- [Kotlin 函数与扩展](Languages/Kotlin/Kotlin03-函数与扩展.md)
- [Kotlin Lambda编程](Languages/Kotlin/Kotlin04-Lambda编程.md)
- [Kotlin 类型系统](Languages/Kotlin/Kotlin05-类型系统.md)
- [Kotlin 运算符重载](Languages/Kotlin/Kotlin06-运算符重载.md)
- [Kotlin 委托](Languages/Kotlin/Kotlin07-委托.md)
- [Kotlin 泛型](Languages/Kotlin/Kotlin08-泛型.md)
- [Kotlin 与 Java 互操作](Languages/Kotlin/Kotlin09-与Java互操作.md)
- [Kotlin 反射](Languages/Kotlin/Kotlin10-反射.md)
- [Kotlin API 总结](Languages/Kotlin/Kotlin-API总结.md)

Kotlin Android：

- [Kotlin Android 开发实践](Languages/Kotlin/Kotlin-Android-开发实践.md)
- [Kotlin Android 避坑](Languages/Kotlin/Kotlin-Android-避坑.md)

Kotlin 协程：

- [Kotlin 协程概述](Languages/Kotlin/KotlinCoroutine01-概述.md)
- [Kotlin Guide](Languages/Kotlin/KotlinCoroutine02-Guide.md)
- [Kotlin 协程总结](Languages/Kotlin/KotlinCoroutine03-总结.md)
- [Kotlin 协程实践](Languages/Kotlin/KotlinCoroutine04-实践.md)

### Groovy

- [Groovy 学习资料](Languages/Groovy/Groovy学习资料.md)
- [Groovy 介绍](Languages/Groovy/Groovy01-介绍.md)
- [Groovy 入门](Languages/Groovy/Groovy02-起步.md)
- [面向 Java 开发者的 Groovy](Languages/Groovy/Groovy03-面向Java开发者的Groovy.md)
- [Groovy 动态类型](Languages/Groovy/Groovy04-动态类型.md)
- [Groovy 闭包](Languages/Groovy/Groovy05-闭包.md)
- [Groovy 字符串](Languages/Groovy/Groovy06-字符串.md)
- [Groovy 使用集合](Languages/Groovy/Groovy07-使用集合.md)
- [Groovy 探索GDK](Languages/Groovy/Groovy08-探索GDK.md)
- [Groovy 处理XML](Languages/Groovy/Groovy09-XML.md)
- [Groovy 数据库](Languages/Groovy/Groovy10-数据库.md)
- [Groovy 使用脚本和类](Languages/Groovy/Groovy11-使用脚本和类.md)
- [Groovy 探索元对象协议](Languages/Groovy/Groovy12-探索元对象协议.md)
- [Groovy 使用MOP拦截方法](Languages/Groovy/Groovy13-使用MOP拦截方法.md)
- [Groovy MOP方法注入](Languages/Groovy/Groovy14-MOP方法注入.md)
- [Groovy MOP方法合成](Languages/Groovy/Groovy15-MOP方法合成.md)
- [Groovy MOP技术汇总](Languages/Groovy/Groovy16-MOP技术汇总.md)
- [Groovy 应用编译时元编程](Languages/Groovy/Groovy17-应用编译时元编程.md)
- [Groovy 生成器](Languages/Groovy/Groovy18-生成器.md)
- [Groovy 单元测试与模拟](Languages/Groovy/Groovy19-单元测试与模拟.md)
- [在 Groovy 中创建 DSL](Languages/Groovy/Groovy20-在Groovy中创建DSL.md)

### CPP

- [CPP 学习资料](Languages/CPP/CPP学习资料.md)
- [CPP 对 C 的增强](Languages/CPP/CPP对C的增强.md)
- [01-概述](Languages/CPP/01-概述.md)
- [02-基础语法](Languages/CPP/02-基础语法.md)
- [03-变量和基本类型](Languages/CPP/03-变量和基本类型.md)
- [04-字符串](Languages/CPP/04-字符串.md)
- [06-vector](Languages/CPP/06-vector.md)
- [07-迭代器](Languages/CPP/07-迭代器.md)
- [08-数组](Languages/CPP/08-数组.md)
- [09-表达式](Languages/CPP/09-表达式.md)
- [10-函数](Languages/CPP/10-函数.md)
- [11-类](Languages/CPP/11-类.md)
- [12-面向对象](Languages/CPP/12-面向对象.md)
- [13-运算符重载](Languages/CPP/13-运算符重载.md)
- [14-动态对象](Languages/CPP/14-动态对象.md)
- [15-IO](Languages/CPP/15-IO.md)
- [16-异常处理](Languages/CPP/16-异常处理.md)
- [17-顺序容器](Languages/CPP/17-顺序容器.md)
- [18-泛型算法](Languages/CPP/18-泛型算法.md)
- [19-关联容器](Languages/CPP/19-关联容器.md)
- [20-模板](Languages/CPP/20-模板.md)
- [21-stl综合](Languages/CPP/21-stl综合.md)

### Python

学习资料：

- [Python学习资料](Languages/Python/Python学习资料.md)
- [Python2&Python3](Languages/Python/Python2.md)
- [Python标准库](Languages/Python/Python标准库.md)

Python3基础：

- [01-Python 简介](Languages/Python/01-Python简介.md)
- [02-基础语法](Languages/Python/02-基础语法.md)
- [03-字符串](Languages/Python/03-字符串.md)
- [04-容器](Languages/Python/04-容器.md)
- [05-运算符与表达式](Languages/Python/05-运算符与表达式.md)
- [06-包与模块](Languages/Python/06-包与模块.md)
- [07-函数](Languages/Python/07-函数.md)
- [08-面向对象](Languages/Python/08-面向对象.md)
- [09-正则表达式](Languages/Python/09-正则表达式.md)
- [10-函数式编程与装饰器](Languages/Python/10-函数式编程与装饰器.md)
- [11-文件操作](Languages/Python/11-文件操作.md)
- [12-输入输出](Languages/Python/12-输入输出.md)
- [13-高级部分](Languages/Python/13-高级部分.md)
- [15-调试PDB](Languages/Python/15-调试PDB.md)
- [16-进程](Languages/Python/16-进程.md)
- [17-多线程](Languages/Python/17-多线程.md)
- [18-网络通信](Languages/Python/18-网络通信.md)


---
---
## Linux

- [001-认识Linux](Linux/001-认识Linux.md)
- [002-ubuntu安装与环境配置](Linux/002-ubuntu安装与环境配置.md)
- [003-Linux文件系统与文件权限](Linux/003-Linux文件系统与文件权限.md)
- [004-Linux命令获取帮助](Linux/004-Linux命令获取帮助.md)
- [005-Linux常用命令](Linux/005-Linux常用命令.md)
- [006-vim](Linux/006-vim.md)
- [007-Linux常用服务器构建](Linux/007-Linux常用服务器构建.md)
- [008-包管理](Linux/008-包管理.md)
- [009-Shell](Linux/009-Shell.md)
- [010-make](Linux/010-make.md)
- [Linux疑问记录](Linux/Linux疑问记录.md)
- [ubuntu下搭建Android开发环境](Linux/ubuntu下搭建Android开发环境.md)