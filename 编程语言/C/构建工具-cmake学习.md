# CMake笔记


## 1 基础语法


### 1.1 最简单的CmakeLists

一个最简单的`CmakeLists.txt`文件包括以下信息：

```Shell
#指定要求的版本
cmake_minimum_required(VERSION 3.7)

#指定项目信息
project(Sample01)

#指定C标准版本(可选)
set(CMAKE_C_STANDARD 99)

#指定需要编译的源，默认与CmakeLists.txt在同意目录下
set(SOURCE_FILES main.c)
add_executable(Sample01 ${SOURCE_FILES})
```

### 1.2 编译多个文件

```Shell
cmake_minimum_required(VERSION 3.7)
project(Sample02)

set(CMAKE_C_STANDARD 99)

#方式1
#set(SOURCE_FILES Source/main.c Source/Lib.c)
#add_executable(Sample02 ${SOURCE_FILES})

#方式2
# 查找Source目录下的所有源文件，并将名称保存到 DIR_SRCS 变量
aux_source_directory(Source DIR_SRCS)
add_executable(Sample02 ${DIR_SRCS})    
```


### 1.3 编译不同目录下的多个文件

这是在子目录下需要新建一个`CmakeLists.txt`文件，用于编译子目录下的源文件，然后在根目录下的`CmakeLists.txt`可以包含子模块

子目录构建：

```Shell
# 子目录下需要一个新的CMakeList文件

#设置源码路径
set(SOURCE_FILES Lib.c)

#add_library的意思是将目录中的源文件编译为静态链接库。SubLib为静态库的名称
add_library(SubLib ${SOURCE_FILES})
```

根目录构建：
```Shell
cmake_minimum_required(VERSION 3.7)
project(Sample03)

set(CMAKE_C_STANDARD 99)

#添加子目录
add_subdirectory(Lib)

set(SOURCE_FILES main.c)
add_executable(Sample03 ${SOURCE_FILES})

#添加链接库
target_link_libraries(Sample03 SubLib)
```

### 1.4 自定义编译选项

在当前目录下建立一个`config.h.in`的文件，文件内容如下：

```Shell
#cmakedefine USE_MYMATH
```

然后在`CMakeList.txt`中可以引用`config.h.in`，根据其配置内容生成不同内容的头文件

```Shell
cmake_minimum_required(VERSION 3.7)
project(Sample4)

set(CMAKE_C_STANDARD 99)

#包含当前目录，查找头文件
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# 是否使用自己的 MathFunctions 库
# option的作用是给编译过程中提供一个选项供用户选择，值为ON或OFF。如果没有提供初始值，则默认使用OFF。
option (USE_MYMATH "Use provided math implementation" OFF)
#option (USE_MYMATH "Use provided math implementation" ON)


# configure_file 命令用于加入一个配置头文件 config.h ，这个文件由 CMake 从 config.h.in 生成，通过这样的机制，将可以通过预定义一些参数和变量来控制代码的生成。
configure_file (
        "${PROJECT_SOURCE_DIR}/config.h.in"
        "${PROJECT_BINARY_DIR}/config.h"
)


# 是否加入 MathLib 库
# 如果上面配置的USE_MYMATH为ON，则进入下面流程，引用子目录下的库
if (USE_MYMATH)
    include_directories ("${PROJECT_SOURCE_DIR}/Libs")
    add_subdirectory (Libs)
    set (EXTRA_LIBS ${EXTRA_LIBS} MathLib)
endif (USE_MYMATH)

#构建
#set(SOURCE_FILES main.c)
aux_source_directory(. SOURCE_FILES)
add_executable(Sample4 ${SOURCE_FILES} )
target_link_libraries (Sample4  ${EXTRA_LIBS})
```


注意：`config.h.in`中和`CMakeList.txt`中的USE_MYMATH变量是要求一致的


### 1.5 添加环境检查

有时候需要对系统环境进行检测，要使用到一个平台相关的函数，首先要检测此函数是否存在，存在则使用系统提供的函数，否则使用自己编写的函数。
CMake提供了环境检查的功能：

- 1 在`config.h.in`配置文件中定义下面变量
```
#cmakedefine HAVE_POW
```
- 2 在CMakeLlist中定义
```Shell

......

# 检查系统是否支持 pow 函数
# step1： 在顶层 CMakeLists 文件中添加 CheckFunctionExists.cmake 宏(固定写法)
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
# step2： 调用 check_function_exists 命令测试链接器是否能够在链接阶段找到 pow 函数
check_function_exists (pow HAVE_POW)

#调用check_function_exists函数检查后，HAVE_POW会被赋值，如果系统存在pow函数，则HAV#_POW为ON。
#同时由于在config.h.in中定义了#cmakedefine HAVE_POW，会在config.h中生成HAVE_POW宏定义


if (NOT HAVE_POW)
    #进到这里，表示系统没有提供pow函数，则需要加入自己的pow函数实现
endif (NOT HAVE_POW)

......
```



---
## 2 命令


### 2.1 [Cmake语言](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)

cmake能识别`CMakeLists.txt`和`*.cmake`格式的文件。cmake能够以三种方式 来组织文件：

- `CMakeLists.txt`(Directory)
- `<script>.cmake`(Script)
- `<module>.cmake`(Module)

#### Directory

当CMake处理一个项目时，入口点是一个名为`CMakeLists.txt`的源文件，这个一定是根目录下的`CMakeLists.txt`。这个文件包含整个工程的构建规范，当我们有多个子文件夹需要编译时，使用`add_subdirectory(<dir_name>)`命令来为构建添加子目录。添加的每个子目录也必须包含一个`CMakeLists.txt`文件作为该子目录的入口点。每个子目录的`CMakeLists.txt`文件被处理时，CMake在构建树中生成相应的目录作为默认的工作和输出目录。

#### Script

单个`<script> .cmake`源文件可以通过使用带有-P选项的cmake命令行工具以脚本模式处理。脚本模式只运行给定的CMake语言源文件中的命令，不会生成构建系统。它不允许定义构建目标或操作的CMake命令。


#### Module

Directory或Script中的CMake语言代码可以使用`include()`命令在包含上下文的范围内加载`<module> .cmake`源文件。比如上面用到的`CheckFunctionExists`。也可以提供自己的模块，并在CMAKE_MODULE_PATH变量中指定它们的位置。


### 2.2 CMake中预定义的变量

- `CMAKE_SOURCE_DIR`：这是包含顶级CMakeLists.txt的目录，即顶级源目录

### 2.3 常用命令

- `set(KEY VALUE)`：用于定义变量
- `unset(KEY)`：用于取消变量的定义
- `${xxx-variable}`：用于引用已定义的变量，可以嵌套引用，嵌套引用从内部进行替换，例如`${outer_${inner_variable}variable}`
- `message("message to display" ...)`：用于输入日志，比如`message("CMAKE_SOURCE_DIR : " ${CMAKE_SOURCE_DIR})`，message支持多种模式，比如`STATUS、WARNING：CMake`等，`message(STATUS ${PROJECT_NAME})`表示输出一个附带的信息
- `project(<PROJECT-NAME> [LANGUAGES] [<language-name>...])`：给工程命名,设置项目名称并将该名称存储在PROJECT_NAME变量中
- `aux_source_directory(<dir> <variable>)`：收集指定目录中所有源文件的名称，并将列表存储在提供的`<variable>`中。
- `include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])`：添加头文件路径 
- `link_directories(directory1 directory2 ...)`：添加库文件路径(`.so/.dll/.lib/.dylib/`)
- `link_libraries(library1 <debug | optimized> library2 ...)`：添加需要链接的库文件路径，比如`link_libraries(“xxx/lib/libcommon.a”)`
- `add_definitions(-DFOO -DBAR ...)`：将-D定义标志添加到源文件的编译中
- `find_library (<VAR> name1 [path1 path2 ...])`： 该命令用于查找库，由`<VAR>`命名的缓存条目被创建来存储该命令的结果。如果找到了库，结果将存储在变量中，除非变量被清除，否则搜索将不会重复。如果没有发现，结果将是`<VAR> -NOTFOUND`，并且下次find_library被同一个变量调用时，搜索将被再次尝试。
- `target_link_libraries(<target> [item1 [item2 [...]]] [[debug|optimized|general] <item>] ...)`：设置要链接的库文件的名称，即设置二进制目标之间的依赖关系，示例如下：
```Shell
# 以下写法都可以： 
target_link_libraries(myProject comm)       # 连接libhello.so库，默认优先链接动态库，即myProject依赖comm
target_link_libraries(myProject libcomm.a)  # 显示指定链接静态库
target_link_libraries(myProject libcomm.so) # 显示指定链接动态库

# 再如：
target_link_libraries(myProject libcomm.so)　　#这些库名写法都可以。
target_link_libraries(myProject comm)
target_link_libraries(myProject -lcomm)
```
- `add_executable(<name> [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] source1 [source2 ...])`：为工程生成目标文件
- `configure_file(<input> <output> [COPYONLY] [ESCAPE_QUOTES] [@ONLY] [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ]`：复制文件到另一个地方并修改文件内容，并在输入文件内容中替换`@VAR@`或`${VAR}`的变量值。每个变量引用将被替换为变量的当前值，如果变量的值未被定义，则为空字符串。VAR必须与cmakelist.txt中的变量保持一直，否则会生成注释。说明：
    - `CMAKE_CURRENT_BINARY_DIR`：项目根目录
    - `CMAKE_CURRENT_SOURCE_DIR`：项目构建目录
    - `COPYONLY`：只复制文件，不替换任何东西，不能和`NEWLINE_STYLE <style>`一起使用
    - `ESCAPE_QUOTES`：禁止为 `"` 转义
    - `@ONLY`：只允许替换@VAR@包裹的变量${VAR}则不会被替换；
    - `NEWLINE_STYLE <style>`：设置换行符格式
- `add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 [source2 ...])`：使用指定的源文件将库添加到项目。name在项目中必须是全局唯一的，STATIC，SHARED或MODULE可能会指定要创建的库的类型。
    - MODULE库是没有链接到其他目标的插件，但可以在运行时使用类似dlopen的功能动态加载
    - STATIC表示静态库
    - SHARED表示动态链接库


注意上面参数`<command>`为必填，`[command]`为选填。

### 2.4 常用参数

- `CMAKE_CXX_FLAGS`：是CMake传给C++编译器的编译选项，比如`set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")`


### 2.5 其他

- 支持gdb：需要指定 Debug 模式下开启 -g 选项
```
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

---
## 3 注意

- cmake的语法支持大小、小写和大小写混合，但是只有系统指令是不区分大小写的，但是变量和字符串是区分大小写的
- 传递参数：参数可以用引号包裹起来，双引号内部包括的内容表示一个参数，也可以不用引号，但是假如参数带有分隔符则必须用双引号包裹起来，否则将被认为是多个参数，假如一行不能写完，则用`\\`符号来表示连接成一行

--- 
## 4 todo

- 安装和测试
- 生成安装包
- 文件生成器