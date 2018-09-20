## 1 预备知识

### Common
- Android 类加载机制

### 插件

- [Android动态加载技术三个关键问题详解](http://mp.weixin.qq.com/s?__biz=MzA4MjA0MTc4NQ==&mid=504089665&idx=2&sn=3a1811844c3833ef6b4c9c286116b0a3#rd)
- [2017插件化开发小结](https://www.jianshu.com/p/71bd20eb5ec4)
- [Android插件化：从入门到放弃](http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up)
- [Android插件化开发](https://github.com/carl1990/Android-DynamicAPK-Plugin)
- [Android动态加载技术 简单易懂的介绍方式](https://segmentfault.com/a/1190000004062866)
- [插件化从放弃到捡起](https://kymjs.com/column/plugin.html)
- [Android博客周刊专题之＃插件化开发＃](http://www.androidblog.cn/index.php/Index/detail/id/16#)

### 热修复

- [Android热修复技术专题：来自微信、淘宝、支付宝、QQ空间的热修复方案](https://github.com/DiyCodes/code_news/blob/master/dialy_news/2016/06/%E7%AC%AC34%E6%9C%9F%EF%BC%9AAndroid%E7%83%AD%E4%BF%AE%E5%A4%8D%E6%8A%80%E6%9C%AF%E4%B8%93%E9%A2%98%EF%BC%9A%E6%9D%A5%E8%87%AA%E5%BE%AE%E4%BF%A1%E3%80%81%E6%B7%98%E5%AE%9D%E3%80%81%E6%94%AF%E4%BB%98%E5%AE%9D%E3%80%81QQ%E7%A9%BA%E9%97%B4%E7%9A%84%E7%83%AD%E4%BF%AE%E5%A4%8D%E6%96%B9%E6%A1%88.md)
- [干货满满，Android热修复方案介绍](https://yq.aliyun.com/articles/231111?utm_content=m_34179)
- [Android 热修复你想知道的一切都在这里了](https://juejin.im/entry/58df77baac502e4957872346)
- [最全面的Android热修复技术](https://blog.csdn.net/u010299178/article/details/52031505)

---
## 2 框架

### 插件化

- [任玉刚——dynamic-load-apk](https://github.com/singwhatiwanna/dynamic-load-apk)
- [360——DroidPlugin](https://github.com/Qihoo360/DroidPlugin)
- [android-pluginmgr](https://github.com/houkx/android-pluginmgr/tree/dev)
- [ACDD](https://github.com/bunnyblue/ACDD)
- [滴滴——VirtualAPK](https://github.com/didi/VirtualAPK)
- [携程——DynamicAPK](https://github.com/CtripMobile/DynamicAPK)
- [Small——做最轻巧的跨平台插件化框架](https://github.com/wequick/Small)、[Small文档](http://code.wequick.net/Small/cn/home)
- [360——RePlugin](https://github.com/Qihoo360/RePlugin)
- [Android-Plugin-Framework](https://github.com/limpoxe/Android-Plugin-Framework)：Android插件开发框架完整源码及示例

### 热修复

- [Nuwa](https://github.com/jasonross/Nuwa)
- [腾讯——tinker](https://github.com/Tencent/tinker)
- [美团——Robust](https://github.com/Meituan-Dianping/Robust)
- [支付宝——AndFix](https://github.com/alibaba/AndFix)
- [Aliyun-HotFix(Sophix)](https://cn.aliyun.com/product/hotfix)

### 其他

- [Alibaba——atlas](https://github.com/alibaba/atlas)
- [Alibaba——dexposed](https://github.com/alibaba/dexposed)

---
## 3 MultiDex

- [MultiDex文档](https://developer.android.com/studio/build/multidex.html?hl=zh-cn)
- [包括65536原因、multidex解决、NoClassDefFoundError、Application Not Responding、App首次启动加载方案等](http://yydcdut.com/2016/03/20/split-dex/)
- [MultiDex安装过程源码分析](http://mouxuejie.com/blog/2016-06-11/multidex-source-analysis/)
- [Android分包原理](http://souly.cn/%E6%8A%80%E6%9C%AF%E5%8D%9A%E6%96%87/2016/02/25/android%E5%88%86%E5%8C%85%E5%8E%9F%E7%90%86/)
- [其实你不知道MultiDex到底有多坑](http://blog.zongwu233.com/the-touble-of-multidex)
- [由Android 65K方法数限制引发的思考](http://jayfeng.com/2016/03/10/%E7%94%B1Android-65K%E6%96%B9%E6%B3%95%E6%95%B0%E9%99%90%E5%88%B6%E5%BC%95%E5%8F%91%E7%9A%84%E6%80%9D%E8%80%83/)

### MultiDex Lib

- [快速加载Dex](https://github.com/asLody/TurboDex)
- [Dex65536-ant](https://github.com/mmin18/Dex65536)