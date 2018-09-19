# ndkbuild开发方式

ndkbuild开发方式需要在jni目录下配置Android.mk和Application.md等文件，然后进行编译。

---
## 1 Android.mk文件说明

Android.mk是Android提供的一种makefile文件，用来指定诸如编译生成so库名、引用的头文件目录、需要编译的.c/.cpp文件和.a静态库文件等。

Android.mk文件位于项目`jni/` 目录的子目录中，用于向构建系统描述源文件和共享库。 它实际上是构建系统解析一次或多次的微小**GNU makefile**片段。 Android.mk 文件用于定义 Application.mk、构建系统和环境变量所未定义的项目范围设置。 它还可替换特定模块的项目范围设置。

Android.mk文件语法将资源分组为“模块”。模块可以是以下之一：

- A static library.
- A shared library.
- A standalone executable.

### Android.mk基本变量说明

-  `$`代表调用一个makefile的函数

下面是一个最简单的Android.mk文件配置

```
//一个Android.mk文件必须以LOCAL_PATH变量的定义开头。它用于在开发树中定位源文件。
//在这个例子中，由构建系统提供的宏函数'my-dir'用于返回当前目录的路径（即包含Android.mk文件的目录本身）。
LOCAL_PATH := $(call my-dir)//call my-dir这个函数的作用就是获取当前文件所在的目录赋给 LOCAL_PATH变量

//CLEAR_VARS 变量指向特殊 GNU Makefile，可为您清除许多 LOCAL_XXX 变量，例如 LOCAL_MODULE、LOCAL_SRC_FILES 和 LOCAL_STATIC_LIBRARIES。
//请注意，它不会清除 LOCAL_PATH。此变量必须保留其值，因为系统在单一 GNU Make 执行环境（其中所有变量都是全局的）中解析所有构建控制文件。 在描述每个模块之前，必须声明（重新声明）此变量。
include $(CLEAR_VARS)//重新初始化 gnu make的脚本环境，这里会把所有的编译环境的变量重新初始化

//必须定义LOCAL_MODULE变量来标识在Android.mk中描述的每个模块。该名称必须是唯一的，不包含任何空格。
//请注意，构建系统将自动为相应生成的文件添加正确的前缀和后缀。换句话说，名为'foo'的共享库模块将生成'libfoo.so'。
LOCAL_MODULE    := Hello//告诉编译器编译出来的可执行性文件叫什么名字，编译完以后会变成libHello.so

//LOCAL_SRC_FILES变量必须包含将被构建并组装到模块中的C和/或C ++源文件的列表。
//请注意，不应该在这里列出标题和包含的文件，因为构建系统将为您自动计算依赖关系;只列出将直接传递给编译器的源文件。
LOCAL_SRC_FILES := Hello.c//告诉编译器 编译的源文件的名称

//BUILD_SHARED_LIBRARY 变量指向 GNU Makefile 脚本，用于收集您自最近 include 后在 LOCAL_XXX 变量中定义的所有信息。 此脚本确定要构建的内容及其操作方法。
//简单的说就是：包含编译所需的核心库文件，帮助系统将所有内容连接到一起
include $(BUILD_SHARED_LIBRARY)//代表将该c代码最终会变成一个动态库 扩展名为.so
```

### 静态库与动态库

- 如果在android.mk文件中放置下面这一句`include$(BUILD_SHARED_LIBRARY) `，代表c代码最终会变成一个动态库，扩展名为.so

- 如果在android.mk文件中放置下面这一句`include $(BUILD_STATIC_LIBRARY)`，代表最终c代码会编译成一个静态库，扩展名为.a


---
## 2 Application.mk

Application.mk用于描述应用需要的原生模块。 模块可以是静态库、共享库或可执行文件。Application.mk 文件实际上是定义要编译的多个变量的微小 GNU Makefile 片段。
它通常位于 `$PROJECT/jni/` 下，其中 `$PROJECT` 指向应用的项目目录。 另一种方式是将其放在顶级 `$NDK/apps/` 目录的子目录下。

### Application.mk基本变量说明

```
 //默认情况下，NDK 构建系统为 armeabi ABI 生成机器代码。 此机器代码对应于基于 ARMv5TE、采用软件浮点运算的 CPU。 您可以使用 APP_ABI 选择不同的 ABI
 //指定特定的平台则可以使用APP_ABI := armeabi armeabi-v7a x86 mips
 APP_ABI := all
```

---
## 3 更多指令说明

Android.mk和Application.mk的更多指令说明可以参考官方文档。

---
## 4 引入其他库

使用ndkbuild开发方式，如何引入其他第三方native库呢？参考我的github项目。
