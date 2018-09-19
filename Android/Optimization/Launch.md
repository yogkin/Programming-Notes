# 提升应用的启动速度 和 splash页面的设计

---
## 1 启动分为两种方式

- 冷启动：当直接从桌面上直接启动，同时后台没有该进程的缓存，这个时候系统就需要重新创建一个新的进程并且分配各种资源。
- 热启动：该app后台有该进程的缓存，这时候启动的进程就属于热启动。

热启动不需要重新分配进程，也不会重新创建Application了，直接走的就是app的入口Activity，这样就速度快很多

---
## 2 如何测量一个应用的启动时间

使用命令行来启动app，同时进行时间测量。单位：毫秒

```
    adb shell am start -W [PackageName]/[PackageName.MainActivity]
    adb shell am start -W com.gzsll.hupu/.ui.splash.SplashActivity
    adb shell am start -W com.example.applicationstartoptimizedemo/com.example.applicationstartoptimizedemo.SplashActivity
    adb shell am start -W com.dn.splashoptimize/com.dn.splashoptimize.MainActivity

    ThisTime: 165 指当前指定的MainActivity的启动时间
    TotalTime: 165 整个应用的启动时间，Application+Activity的使用的时间。
    WaitTime: 175 包括系统的影响时间---比较上面大。
```

---
## 3 应用启动的流程

- Application
 - 创建
 - attachBaseContext()
 - onCreate()
- Activity
 - 创建onCreate()--->设置显示界面布局，设置主题、背景等等属性
 - onStart()
 - onResume()
 - 显示里面的view（测量、布局、绘制，显示到界面上）

分析时间花在哪里

---
## 4 减少应用的启动时间的耗时

- 不要在Application的构造方法、attachBaseContext()、onCreate()里面进行初始化耗时操作。
- 由于用户只关心最后的显示的这一帧，对我们的布局的层次要求要减少，自定义控件的话考虑测量、布局、绘制的时间。
- 对于SharedPreference的初始化。因为初始化的时候是需要将数据全部读取出来放到内存当中。
 - 1.可以尽可能减少sp文件数量(IO需要时间)
 - 2.像这样的初始化最好放到线程里面
 - 3.大的数据缓存到数据库里面

app启动的耗时主要是在：Application初始化 + MainActivity的界面加载绘制时间。由于MainActivity的业务和布局复杂度非常高，甚至该界面必须要有一些初始化的数据才能显示。那么这个时候MainActivity就可能半天都出不来，这就给用户感觉app太卡了。我们要做的就是给用户赶紧利落的体验。点击app就立马弹出我们的界面。于是乎想到使用SplashActivity，用来显示非常简单的一个欢迎页面。但是SplashActivity启动之后，还是需要跳到MainActivity。MainActivity还是需要从头开始加载布局和数据。想到SplashActivity里面可以去做一些MainActivity的数据的预加载。然后需要通过意图传到MainActivity。可不可以再做一些更好的优化呢？

如果我们能让这两个时间重叠在一个时间段内并发地做这两个事情就省时间了。解决方案：**将SplashActivity和MainActivity合为一个。**一进来还是显示的MainActivity，SplashActivity可以变成一个SplashFragment，然后放一个FrameLayout作为根布局直接现实SplashFragment界面。SplashFragment里面非常之简单，就是现实一个图片，启动非常快。当SplashFragment显示完毕后再将它remove。同时在splash的2S的友好时间内进行网络数据缓存。这个时候我们才看到MainActivity，就不必再去等待网络数据返回了。

---
## 5 延迟加载DelayLoad

- viewStub：viewStub的设计就是为了防止MainActivity的启动加载资源太耗时了。对某些View延迟进行加载。
- onwindowfocuschange
- ViewTreeObserver

---
## 6 设置 Activity 启动背景

参考[解决启动Android应用程序时出现白屏或者黑屏的问题](https://blog.csdn.net/wangjiang_qianmo/article/details/51736418)

---
## 引用

- [App Startup Time](https://developer.android.google.cn/topic/performance/vitals/launch-time)
