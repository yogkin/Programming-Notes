# Android进程线程和内存优化

---
## 1 Android进程和线程

进程是程序的一个运行实例，程序本身是静态的，而线程是CPU调度的基本单位，大部分操作系统都支持多任务运行，这一特性让用户感觉好像计算机可以同时处理多个任务。这种同时是一种假象，是操作系统采用分时的方法。

一个应用程序的入口一般都是main函数，这基本成立程序开发的规范，即它是一切事物的起源，而main函数的工作也基本是千篇一律，总结如下：

- 初始化：比如在Windows环境下通常要常见窗口，向系统申请资源
- 进入死循环，并在循环种处理各种事件，直到进程退出。

这种模型是**以事件为驱动**的软件系统必然结果。因此几乎存在于任何操作系统和编程语言中，Android也不例外。不过一切从零开始的开发模式显得太过费时费力，Android系统已经封装的笔记完善，加上IDE就可以自动生成各种应用原型。

让IDE创建一个没有任何功能的项目，运行后使用DDMS工具查看程序进程中的线程数，发现有三个线程：

- main
- Binder_1
- Binder_2

一个没有任何功能的程序就创建了三个线程，那么这些线程是如何创建的呢？通过Debug工具查看main线程的堆栈信息，可以追溯到ActivityThread类，没错，这就是每个Android应用程序的入口，代表着Android 应用程序的UI主线程。


---
## 2 Hander、MessageQueue、Runnable和Lopper

- Runnable和Message可以被压入MessageQueue中，形成一个消息队列
- Lopper不断的从MessageQueue中获取Message，然后交由Handler处理，Lopper是一个死循环
- Handler是真正处理Message的地方
- 如果MessageQueue中的消息队列没有消息了，则会进入睡眠以让出CPU资源



---
## 3 UI主线程ActivityThread

- ActivityThread中初始化了main线程的Looper，通过Looper.mainLooper即可获取主线程的Looper。这中方式巧妙的区分开各种线程的Looper，并界定了它们的访问权限。
- ActivityThread中的Handler类型mH是一个事件管家

ActivityThread中初始化Looper让主线程进入死循环，从而形成一个**以事件驱动**为模型的经典模型，**消息是推动整个系统运行起来的基础，即便是操作系统本身也是如此**。

---
## 4 Thread类

**线程类**与线程是两个不同的概念，线程是操作系统CPU资源分配出来的基本调度单元。处于抽象范畴，而线程类本身知识一段可执行的代码。Java中Thread的start方法最终调用VMThread.create方法，这是一个本地方法。


---
## 5 Android应用程序如何利用CPU的多核能力

- Thread
- AsyncTask(`java.util.concurrent`)
- IntentService

---
## 6 Android应用程序的典型启动流程

Android系统基于Linux，原则上说它的应用程序并不知识APK一种类型，换句话说，所有Linux支持的应用程序都可以通过一定方式运行在Android系统上，比如一些系统级应用。

---
## 7 Android应用程序的内存管理与优化

```
getprop | grep dalvik.vm.heapsize
```

`dalvik.vm.heapsize`值指定了一个进程可以使用的最大HeapSize，用过`dumpsys meminfo package_name`可以查看进程的Heap信息。Heap分为两大类：

- Native
- Dalvik