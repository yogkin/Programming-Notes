# CMake简要学习

---
## 1 什么是 CMake

 GNU Make、qmake等Make工具遵循着不同的规范和标准，所执行的Makefile格式也是不同的。所以如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用这类Make工具，就得为每一种标准写一次 Makefile。这显然很低效，而CMake就是针对这个问题所设计的工具：它允许开发者编写一种平台无关的 CMakeList.txt 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile和工程文件，从而做到“Write once, run everywhere”，所以CMake 是一个比上述几种Make更高级的编译配置工具。总结**CMake是一个开源，跨平台的工具系列，旨在构建，测试和打包软件。**

---
## 2 示例与笔记

- Sample01：最简单的Cmake示例
- Sample02：编译多个文件
- Sample03：编译不同目录下的多个文件
- Sample04：自定义编译选项
- Sample05：构建时检查系统环境

参考 [github](https://github.com/Ztiany/Programming-Notes-Code/blob/master/C%26C%2B%2B/cmake/README.md) 仓库

---
## 引用

### 官方文档

- [CMake官网教程](https://cmake.org/documentation/)
- [CMake Wiki](https://cmake.org/Wiki/CMake)
- [CMake文档](https://cmake.org/cmake/help/v3.0/index.html#)
- [CMake Wiki](https://cmake.org/Wiki/CMake)
- [CMake_Useful_Variables](https://cmake.org/Wiki/CMake_Useful_Variables)

### 博客

- [make makefile cmake qmake都是什么，有什么区别？](https://www.zhihu.com/question/27455963)
- [CMake 入门实战](http://hahack.com/codes/cmake/)
- [cmake使用教程](https://juejin.im/post/5a6f32e86fb9a01ca6031230)
- [使用CMake组织C++工程](https://elloop.github.io/tools/2016-04-04/learning-cmake-0)
- [cmake手册](http://www.cnblogs.com/coderfenghc/tag/cmake/)
- [Android NDK 开发之 CMake 必知必会](https://juejin.im/post/5b9879976fb9a05d330aa206)