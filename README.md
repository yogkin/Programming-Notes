# 编程笔记

>wiz --> github，笔记还在迁移中......

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
## 工具

- [Git 常用命令](Tools/Git基础命令.md)
- [AndroidStudio](Tools/Android.md)
- [IDEA&AndroidStudio插件](Tools/IDEA&AndroidStudio插件.md)
- [Windows 常用软件](Tools/Windows-Software-List.md)
- [Windows 操作技巧](Tools/Windows-operating-skill.md)

---
## 计算机网络

- [网络相关学习资料](计算机网络/网络通信基础/网络相关学习资料.md)

### 图解密码技术

- [《图解密码技术》第一部分](计算机网络/网络通信基础/图解密码技术-part01.md)

### HTTP

- [HTTP 概述](计算机网络/网络通信基础/HTTP_01_概述.md)
- [HTTP Content encoding 和 Transfer Encoding](计算机网络/网络通信基础/HTTP_02_Content_encoding_Transfer_Encoding.md)
- [HTTP 范围请求](计算机网络/网络通信基础/HTTP_03_范围请求.md)


### HTTPS

- [《HTTPS权威指南》-TLS/SSL](计算机网络/网络通信基础/HTTPS权威指南01-SSL&TLS.md)


---
## 编程语言

### C语言

学习资料：

[C学习资料.md](编程语言/C/C学习资料.md)

编译与构建：

- [编译器-gcc学习.md](编程语言/C/编译器-gcc学习.md)
- [构建工具-cmake简介.md](编程语言/C/构建工具-cmake简介.md)  
- [构建工具-cmake学习.md](编程语言/C/构建工具-cmake学习.md)
- [LLVM](https://llvm.org/) 项目是模块化和可重用的编译器和工具链技术的集合

C 语言基础与提高：

- [01-C语言简介.md](编程语言/C/01-C语言简介.md)
- [02-C语言基础.md](编程语言/C/02-C语言基础.md)
- [03-数据类型.md](编程语言/C/03-数据类型.md)
- [04-sizeof和size_t类型.md](编程语言/C/04-sizeof和size_t类型.md)
- [05-数组.md](编程语言/C/05-数组.md)
- [06-函数.md](编程语言/C/06-函数.md)
- [07-存储类别.md](编程语言/C/07-存储类别.md)
- [08-字符串.md](编程语言/C/08-字符串.md)
- [09-左值与右值.md](编程语言/C/09-左值与右值.md)
- [10-指针.md](编程语言/C/10-指针.md)
- [11-指针的大小.md](编程语言/C/11-指针的大小.md)
- [12-输入和输出.md](编程语言/C/12-输入和输出.md)
- [13-预处理.md](编程语言/C/13-预处理.md)
- [14-头文件.md](编程语言/C/14-头文件.md)
- [15-typedef.md](编程语言/C/15-typedef.md)
- [16-结构体-枚举-联合体.md](编程语言/C/16-结构体-枚举-联合体.md)
- [17-字节对齐与内存分配.md](编程语言/C/17-字节对齐与内存分配.md)
- [20-文件读写.md](编程语言/C/20-文件读写.md)
- [21-错误处理.md](编程语言/C/21-错误处理.md)
- [22-动态内存管理.md](编程语言/C/22-动态内存管理.md)
- [23-C语言标准库.md](编程语言/C/23-C语言标准库.md)
- [24-C语言内存四区.md](编程语言/C/24-C语言内存四区.md)
- [25-C 提高](编程语言/C/25-C提高.md)
- [25-C 提高练习](编程语言/C/25-C提高练习.md)

### Kotlin

学习资料：

- [Kotlin 学习资料](编程语言/Kotlin/Kotlin学习资料.md)

Kotlin 基础：

- [Kotlin 基础语法](编程语言/Kotlin/Kotlin01-基础语法.md)
- [Kotlin 面向对象](编程语言/Kotlin/Kotlin02-面向对象.md)
- [Kotlin 函数与扩展](编程语言/Kotlin/Kotlin03-函数与扩展.md)
- [Kotlin Lambda编程](编程语言/Kotlin/Kotlin04-Lambda编程.md)
- [Kotlin 类型系统](编程语言/Kotlin/Kotlin05-类型系统.md)
- [Kotlin 运算符重载](编程语言/Kotlin/Kotlin06-运算符重载.md)
- [Kotlin 委托](编程语言/Kotlin/Kotlin07-委托.md)
- [Kotlin 泛型](编程语言/Kotlin/Kotlin08-泛型.md)
- [Kotlin 与 Java 互操作](编程语言/Kotlin/Kotlin09-与Java互操作.md)
- [Kotlin 反射](编程语言/Kotlin/Kotlin10-反射.md)
- [Kotlin API 总结](编程语言/Kotlin/Kotlin-API总结.md)

Kotlin Android：

- [Kotlin Android 开发实践](编程语言/Kotlin/Kotlin-Android-开发实践.md)
- [Kotlin Android 避坑](编程语言/Kotlin/Kotlin-Android-避坑.md)

Kotlin 协程：

- [Kotlin 协程概述](编程语言/Kotlin/KotlinCoroutine01-概述.md)
- [Kotlin Guide](编程语言/Kotlin/KotlinCoroutine02-Guide.md)
- [Kotlin 协程总结](编程语言/Kotlin/KotlinCoroutine03-总结.md)
- [Kotlin 协程实践](编程语言/Kotlin/KotlinCoroutine04-实践.md)

### Groovy

- [Groovy 学习资料](编程语言/Groovy/Groovy学习资料.md)
- [Groovy 介绍](编程语言/Groovy/Groovy01-介绍.md)
- [Groovy 入门](编程语言/Groovy/Groovy02-起步.md)
- [面向 Java 开发者的 Groovy](编程语言/Groovy/Groovy03-面向Java开发者的Groovy.md)
- [Groovy 动态类型](编程语言/Groovy/Groovy04-动态类型.md)
- [Groovy 闭包](编程语言/Groovy/Groovy05-闭包.md)
- [Groovy 字符串](编程语言/Groovy/Groovy06-字符串.md)
- [Groovy 使用集合](编程语言/Groovy/Groovy07-使用集合.md)
- [Groovy 探索GDK](编程语言/Groovy/Groovy08-探索GDK.md)
- [Groovy 处理XML](编程语言/Groovy/Groovy09-XML.md)
- [Groovy 数据库](编程语言/Groovy/Groovy10-数据库.md)
- [Groovy 使用脚本和类](编程语言/Groovy/Groovy11-使用脚本和类.md)
- [Groovy 探索元对象协议](编程语言/Groovy/Groovy12-探索元对象协议.md)
- [Groovy 使用MOP拦截方法](编程语言/Groovy/Groovy13-使用MOP拦截方法.md)
- [Groovy MOP方法注入](编程语言/Groovy/Groovy14-MOP方法注入.md)
- [Groovy MOP方法合成](编程语言/Groovy/Groovy15-MOP方法合成.md)
- [Groovy MOP技术汇总](编程语言/Groovy/Groovy16-MOP技术汇总.md)
- [Groovy 应用编译时元编程](编程语言/Groovy/Groovy17-应用编译时元编程.md)
- [Groovy 生成器](编程语言/Groovy/Groovy18-生成器.md)
- [Groovy 单元测试与模拟](编程语言/Groovy/Groovy19-单元测试与模拟.md)
- [在 Groovy 中创建 DSL](编程语言/Groovy/Groovy20-在Groovy中创建DSL.md)