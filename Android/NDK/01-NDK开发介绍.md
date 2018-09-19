# Android NDK开发

---
## 1 NDK开发介绍

### NDK

Native Development Kit（NDK）是一系列工具的集合。它提供了一系列的工具，帮助开发者快速开发C/C++的动态库，并能自动将so和java一起打包成apk。

### JNI

Java Native Interface（JNI）标准是java平台的一部分，JNI是Java语言提供的Java和C/C++相互沟通的机制，Java可以通过JNI调用C/C++代码，C/C++的代码也可以调用java代码。

### 为什么使用NDK

1. 项目需要调用底层的一些C/C++的一些东西（java无法直接访问到操作系统底层（如系统硬件等）），或者已经在C/C++环境下实现了功能代码（大部分现存的开源库都是用C/C++代码编写的。），直接使用即可。NDK开发常用于驱动开发、无线热点共享、数学运算、实时渲染的游戏、音视频处理、文件压缩、人脸识别、图片处理等。
2. 为了效率更加高效些。将要求高性能的应用逻辑使用C/C++开发，从而提高应用程序的执行效率。但是C/C++代码虽然是高效的，在java与C/C++相互调用时却增大了开销；
3. 基于安全性的考虑。防止代码被反编译，为了安全起见，使用C/C++语言来编写重要的部分以增大系统的安全性，最后生成so库（用过第三方库的应该都不陌生）便于给人提供方便。（任何有效的代码混淆对于会smail语法反编译你apk是分分钟的事，即使你加壳也不能幸免高手的攻击）
4. 便于移植。用C/C++写得库可以方便在其他的嵌入式平台上再次使用。


### NDK使用

配置Android.mk和Application.mk文件，编写好相关代码，然后使用ndk-build命令进行编译。

示例：
```
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=./jni/Application.mk NDK_LIBS_OUT=./jniLibs
```
- `NDK_PROJECT_PATH` 指定项目路径, 会自动读取这个目录下的 jni/Android.mk 文件
- `NDK_APPLICATION_MK` 指定 Application.mk 的位置
- `NDK_LIBS_OUT` 指定将生成的 .so 文件放到哪个目录，默认 Android Studio 会读取 jniLibs 目录下的 .so 文件, 所以我们把 .so 文件生成到这


关于ndk-build的命令选项可以通过`ndk-build -h`获取。


---
## 2 一些概念

### 编译

把一个源文件 源代码 翻译(编译)成一个二进制文件的过程

### 连接

把编译生成的二进制，根据操作系统，根据当前处理器的类型．把这个二进制文件转化成一个真正可以执行的二进制文件．

### 交叉编译

使用交叉编译工具链在一个平台下(一种cpu的平台)编译出来另外一个平台(另一种cpu的平台)可以运行的二进制代码

### 动态库，静态库

- 静态库 文件名.a   包含所有的函数并且函数运行的依赖，体积大，包含所有的API
- 动态库 文件名.so  包含函数，不包含函数运行的依赖，体积小，运行的时候，去操作系统寻找需要的API

### make

>维基百科：在软件开发中，make是一个工具程序（Utility software），经由读取叫做“makefile”的文件，自动化建构软件。它是一种转化文件形式的工具，转换的目标称为“target”；与此同时，它也检查文件的依赖关系，如果需要的话，它会调用一些外部软件来完成任务。它的依赖关系检查系统非常简单，主要根据依赖文件的修改时间进行判断。大多数情况下，它被用来编译源代码，生成结果代码，然后把结果代码连接起来生成可执行文件或者库文件。它使用叫做“makefile”的文件来确定一个target文件的依赖关系，然后把生成这个target的相关命令传给shell去执行。

---
## 3 NDK 开发方式

1. 使用使用eclipse开发
2. 在AndroidStudio上，徒手编写Android.mk然后ndk-build编译，这种方式需要在`gradle.properties`中添加`android.useDeprecatedNdk = true`
3. 使用gradle-experimental NDK插件进行开发，这个方式只是一个试验性功能，现在已经废弃
4. 使用AndroidStudio2.2及更高版本，这个版本增强了C++的开发能力，能够使用 ndk-build 或 CMake 去编译和调试项目里的 C++ 代码，此时AndroidStudio用于构建原生库的默认工具是 CMake，但是仍然支持使用ndkBuild
    - 在`externalNativeBuild中配置ndkBuild`
    - 在`externalNativeBuild中配置cmake`

### 关于gcc与clang

- Android NDK从r11开始建议大家切换到clang。并且把GCC标记为deprecated，将GCC版本锁定在GCC 4.9不再更新。
- Android NDK从r13起，默认使用Clang进行编译。

---
## 4 NDK开发学习内容

- c与c++语言
- ndk-build方式nkd开发
- cmake方式nkd开发
- api选择与适配
- android平台提供的本地api
- 引入第三方的本地库，例如ffmpeg