# ARouter 分析

---
## 反射在Android中的应用

Java反射其实就是把Java中的类拆分成各个部分，利用反射可以在运行时操作各个对象的字段和方法，反射在各种框架中也有大量使用，而在Android Framework层中，更是随处可见，就连Activity对象也是通过反射创建的，在应用开发中，使用反射可以达到一些特殊的需求，比如hook一些系统API，应用程序的入口是ActivityThread中的main方法，通过反射获取ActivityThread对象，可以hook系统中很多核心的对象，比如负责处理远程服务器信息的H类、比如负责创建各大组件的Instrumentation类等。

反射ActivityThread：

```
    private Object mActivityThread;
    private Class<?> mActivityThreadClass;

   mActivityThreadClass = Class.forName("android.app.ActivityThread");
   Method currentActivityThreadMethod = mActivityThreadClass.getDeclaredMethod("currentActivityThread");
   currentActivityThreadMethod.setAccessible(true);
   //获取主线程对象
   mActivityThread = currentActivityThreadMethod.invoke(null);
```

