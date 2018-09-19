# Application

一个Application可以说代表一个App实例，每一个App只会初始化一个Application，我们可以继承Application实现一些特定的逻辑，Application也提供了一些方法开发者的方法。

---
## 1 registerComponentCallbacks（） & unregisterComponentCallbacks（）

**这两个方法在API14引入**。调用registerComponentCallbacks方法可以传入ComponentCallbacks2类型

registerComponentCallbacks有三个方法：

- onConfigurationChanged(Configuration newConfig)：配置改变时被回调
- onLowMemory()：监听 Android系统整体内存较低时刻，这是Android 4.0前 检测内存使用情况，从而避免被系统直接杀掉 & 优化应用程序的性能体验
- onTrimMemory(int level)：通知 应用程序 当前内存使用情况（以内存级别进行识别），应用程序可以根据当前内存使用情况进行自身的内存资源的不同程度释放，以避免被系统直接杀掉 & 优化应用程序的性能体验

---
##  2 registerActivityLifecycleCallbacks（） & unregisterActivityLifecycleCallbacks（）

**这两个方法在API14引入**。

registerActivityLifecycleCallbacks 用于监听应用程序内所有Activity的生命周期。registerActivityLifecycleCallbacks中所有的监听方法是在 Activity 的 supper.onXXX 中调用的，supper 中做了很多处理，比如 Fragment 的恢复，需要注意的是，在 onCreate 中，是先处理 Fragment 恢复相关逻辑，然后才分发生命周期事件到 registerActivityLifecycleCallbacks 中。

---
## 3 onTerminate()

onTerminate在应用程序结束时调用，但是**但该方法只用于Android仿真机测试，在Android产品机是不会调用的**

---
## 参考

- [Application解析](http://www.jianshu.com/p/f665366b2a47)