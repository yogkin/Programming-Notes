# Android

---
## 1 系统架构

![](index_files/android-stack_2x.png)

硬件抽象层 (HAL) 层：第一次亮相是在2008年的Google IO大会上。HAL提供标准界面，向更高级别的 Java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。

---
## 2 编译 Android 系统

- 环境要求：Linux、Mac
- 下载源码：
    - [按官方要求配置](https://source.android.com/source/downloading?hl=zh-cn)
    - [清华AOSP镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
- 使用Repo进行源码管理

其他扩展：

- 原生Android系统编译
- 定制产品的编译和烧录
- Android Multilib Build
- Android系统映像文件，比如system.img为安卓系统的运行程序包(framework)
- ODEX即Optimized Dakvik Executable的缩写，是针对Dalvik虚拟机上dex文件的优化产物
- OAT(Over the Air)系统升级，即增量式系统升级
- Android 反编译：APK的反编译；Android系统包的反编译，smali语法
- NDK build
- 第三方ROM的移植，比如CyanogenMod
- Makefile
- SDK编译