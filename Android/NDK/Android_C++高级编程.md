# 《Android C++高级编程：使用NDK》笔记

---
## 2 深入理解Android NDK

### ndk工具目录说明


| 目录名 | 描述|
|---|---|
| build   | 存放和编译相关的脚本文件,最外面的ndk-build就是调用该目录下的makefile文件,其中makefile文件都存放在build/core目录 |
| docs  | 帮助文档 |
| platforms  | 存放不同android版本,不同平台架构的头文件和库文件 |
| prebuilt  | 存放和编译相关工具比如make.exe |
| samples | ndk代码例子,用根目录下的ndk-build即可编译 |
| source | 源码目录,有一些头文件和库文件,比如gnu-libstdc,stlport |
| test | 一些测试样例,有很多例子可以从里面学习 |
| toolchains | 不同平台的编译器链接器目录以及一些和编译连接相关的工具,gcc,ld等工具都在这个目录 |

### obj文件

obj是中间目录，编译源代码后所产生的目标文件都保存在该目录下。

### 掌握Android.mk和Application.mk语法

其实.mk文件都是make文件的片段，类似`include $(BUILD_SHARED_LIBRARY)`是从NDK目录把`build-shared-library.mk`文件引入到当前make文件中，其实并没有什么神奇之处。


---
## 5 日志与调试

### 控制台日志

将stdout和stderr输出到logcat

```
adb shell stop
adb shell setprop log.redirect-stdio true
adb shell start
```

### ndk-stack 排查错误

1. 保存logcat文件，分析日志
2. ndk-stack工具：`adb logcat | ndk-stack -sym xxx/obj/local/armeabi/`
3.addr2line定位错误行：
`arm-linux-androideabi-addr2line -e xxx/obj/local/armeabi/libmyffmpeg.so 0x580001d`，0x580001d为定位到的地址

### gdb调试

*   编译加上-g参数：`gcc test1.c -g -o test1`
*   进入调试：`gdb test1`
*   开始调试：`start`
*   显示代码：`list`-简写l
*   查看函数内容：`list 函数名称`
*   查看某行代码：`list 行数`
*   执行下一步：`next`-简写n
*   查看变量：`print 变量名`-简写p
*   进入到某个函数：`step`-简写s
*   设置断点：`break 行号`(gdb中的行号)-简写b
*   运行：`continue`-遇到断点会停止-简写b
*   查看断点信息：`info breakpoints`
*   删除断点：`delete breakpoints 断点编号`
*   修改变量的值：`set var 变量=值`
*   程序调用堆栈：`backtrace-简写bt`，当前函数之前的所有已调用函数列表，每一个都分配一个“帧”，最近调用的函数在0号帧里
*   切换栈帧：`frame 1`查看指定栈帧的变量
*   自动显示：`display 变量名`
*   取消自动显示：`undisplay 行号`（自动显示的行号）
*   查看内存布局：`x /20 buff`，查看buff数组的前20个元素
*   程序非正常退出，如何查看错误？
    1.  `ulimit -a` 查看core文件是否分配大小
    2.  `ulimit -c 1024` 创建的core文件大小为1024字节
    3.  `gcc test2.c -g -o test2` 编译链接得到带有-g选项的可执行程序
    4.  `./test2` 执行程序，会生成core日志文件
    5.  `gdb test2 core` 打开日志文件，定位错误信息到具体的代码行数

---
## 6 Binoic API

**Bionic**是Android平台为使用C和C++进行元素应用程序开发所提供的POSIX标准C库，Google希望用它来取代glibc(GNU C)，它的发展目标是达到轻量化以及高运行速度。Bionic不会支持全部的C标准库函数，Android NDK文档中提供了缺失功能的完整列表。

### 与进程交互

使用system函数可以向shell传递命令，system函数在stblib头文件中：
```
int result = system("mkdir /data/data/com.example.hellojni/temp");
if(result == 0 || result == 127){
   //shell fail
}
```

使用popen函数可以与子进程进行交互：

```c
static void shellPro(const char *command) {
    FILE *stream;
    stream = popen(command, "r");
    if (NULL == stream) {
        LOGE("popen(%s) fail", command);
    } else {
        char buf[1024 * 5];
        int status;
        while (NULL != fgets(buf, 1024 * 5, stream)) {
            LOGE("read: %s", buf);
        }
        status = pclose(stream);
        LOGI("pclose result: %d",status);
    }
}
```

### 获取系统属性

在`sys/system_properties.h`中，定义了下面三个函数用于获取系统属性

- `__system_property_get`
- `__system_property_find`
- `__system_property_read`

### 进程间通讯

为了避免拒绝服务攻击和内核资源泄露，Bionic不提供队System V的进程间通讯的支持。程序开发者应该使用Binder机制。

---
## 7 原生线程

Android支持Java线程和原生线程，原生线程API定义在`pthread`中。Pthread是线程的POSIX标准。

### 原生代码使用Java线程的优缺点

与使用原生线程相比，原生代码使用Java线程的优点：

- 更容易建立
- 原生代码不需要做任何修改
- 因为Java线程已经是Java平台的一部分，所以不需要显式地附着到JVM上，原生代码可以用提供的线程专业JNIEnv接口指针与Java代码通信
- 通过java.lang.Thread提供的API可以与Java代码中的线程实例无缝交互

与使用原生线程相比，原生代码使用Java线程有哪些不足：

- 因为原生空间没有创建Java线程的API，所以假设为线程分配任务的逻辑是Java代码的一部分
- 因为基于Java的线程队原生代码是透明的，所以假设原生代码是安全的
- 原生代码不能获益于其他并发程序的概念和组件，比如信号量等
- 在不同线程中的原生代码不同信息或直接共享资源


### 线程API：`pthread.h`

- `int pthread_create(pthread_t *thread, pthread_attr_t const * attr, void *(*start_routine)(void *), void * arg);`：用于创建线程

- `int pthread_join(pthread_t thid, void ** ret_val);`：用于等待子线程并获取运行结果

>在JNI中使用线程时，如果在底层线程中获取不到Java平台的对象或相关属性，可以尝试在创建线程之前获取，然后保存其为全局引用，共创建后的底层线程使用

### 同步

POSIX线程提供的两种最常用的同步工具为：互斥锁(mutexes)和信号量(semaphores)

#### 互斥锁

互斥锁保证代码的互斥，锁定的部分代码不同同时执行。

使用流程
```
//创建互斥锁
static pthread_mutex_t mutex;

//初始化
pthread_mutex_init(&mutex, NULL); //返回0表示失败

//上锁与解锁
pthread_mutex_lock(&mutex); //返回0表示失败
pthread_mutex_unlock(&mutex); //返回0表示失败

//销毁锁
 pthread_mutex_destroy(&mutex); //返回0表示失败
```

#### 信号量

信号量定义在`semaphore.h`中

使用流程：
```
//创建信号量
sem_t sem;

//初始化信号量：sem为指向信号量结构的一个指针；pshared不为０时此信号量在进程间共享(共享标识)，否则只能为当前进程的所有线程共享；value给出了信号量的初始值(最大线程运行数)。如果初始化成功，返回0，否则为-1
int sem_init(sem_t *sem, int pshared, unsigned int value);

//sem_wait函数的作用是减少信号量数量，传入的信号量指针将被赋值，如果大于0，表示上锁成功，并且信号量的值减去1，如果信号量的值是0，调用线程挂起，直到另一个线程通过解锁增加信号量的值，该函数执行成功返回0，否则-1
int sem_wait(sem_t * sem);

//解锁信号量，使用sem_post时，信号量的值会加1，调度策略会决定信号量解锁后执行哪个挂起的线程，该函数执行成功返回0，否则-1
int sem_post(sem_t * sem);

//销毁
int sem_destroy(sem_t* sem);
```

#### 调度策略

POSIX线程规范要求实现一组调度策略，最常使用的策略如下:：
- SCHED_FIFO：先进先出调度策略基于线程进入列表的时间，可以基于线程优先级
- SCHED_RR：循环轮询调度策略是线程执行时间加以限制的SCHED_FIFO，其目的是规避线程独占可用的CPU时间

这些调度策略定义在`sched.h`中，可以在调用`pthread_create()`创建线程时，用线程属性参数`pthread_attr_t`的`sched_policy`来定义，也可以在运行时调用`pthread_setschedparam`函数定义调度策略

#### 优先级

POSIX也提供了基于调度策略调整线程优先级的函数，可以在调用`pthread_create(`)创建线程时，用线程属性参数`pthread_attr_t`的`sched_policy`来定义线程优先级，也可以在运行时调用`pthread_setschedparam`函数提供优先级，应用程序可以使用`sched_get_priority_max`函数和`sched_get_priority_min`函数来查询这些数


---
## 8 POSI Socket API

### TCP：面向连接的通讯

- 1 创建socket`int socket(int domain, int type, int protocol);`，返回用一个名为socket描述符的整数。

- 2 `int bind(int, const struct sockaddr *, int);`，将socket与一个地址绑定

- 3 网络字节排序：在硬件层上，不同机器体系结构使用不同的数据排序和表示规则，这种称为机器字节排序或字节序。例如Big-endian(ARM、x86使用)和Little-endian(MIPS使用)，字节排序规则不同的机器不能直接交互数据，为了使字节排序不同的机器能在网络上通信，IP将Big-endian字节排序设置为官方的数据传输网络字节排序规则，像Java虚拟机已经使用了Big-endian字节排序，所以Java中不需要程序员作字节排序转换，而原生代码需要在机器字节排序和网络字节排序间做必要的转换，socket库提供了一组遍历函数，是原生代码可以透明地处理字节排序转换，这些函数定义在`sys/endian.h`中：
    - htons:将unsigned short 从主机字节排序转换到网络字节排序
    - ntons:与htons相反
    - htonl:将unsigned integer 从主机字节排序转换到网络字节排序
    - ntonl:与htonl相反


- 4 `int listen(int, int);`，监听socke是通过listen函数完成的

- 5 `int accept(int, struct sockaddr *, socklen_t *);`，接收传入的连接

- 6 `ssize_t  recv(int, void *, size_t, unsigned int);`，阻塞函数，从socket接收数据

- 7 `ssize_t  send(int, const void *, size_t, unsigned int);`，向socket发送数据

### UDP：面向无连接的通讯

UDP的其他API与TCP共用，而发送和接收数据的API不同

- `ssize_t sendto(int, const void *, size_t, int, const struct sockaddr *, socklen_t);`，用于发送数据
- `ssize_t recvfrom(int, void *, size_t, unsigned int, const struct sockaddr *, socklen_t *);`，用于接收数据

### 异步IO

大多数socket api阻塞函数调用，这些函数挂起调用进程直到满足某些条件，例如都操作时socket上有可读数据，socket通过select函数提供异步io，select可以操作多个socket描述符并同时监控它们的状态。

---
## 11 支持 C++

Android平台提供了一个微型的C++运行库，称为**系统运行库**，该运行库不支持以下特性：

- C++标准库
- 异常支持
- RTTI支持

Android NDK提供了用于补充系统运行库功能的一些额外的C++运行库，以完善缺失的特性：

- 系统库
- GAbi++
- STLport
- GNU STL

---
## 12 原生图形API

- JNI Graphice(原生Bitmap)
- OpenGL ES(1.x/2.0)
- 原生Window

---
## 13 原生音频API

- OpenSL ES

---
## 14 程序概要分析和NEON优化

- GNU Profiler
- ARM NEON Intrinsics技术