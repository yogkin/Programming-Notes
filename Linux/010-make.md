# MakeFile

make是一个命令工具，它解释Makefile中的指令（应该说是规则）。在Makefile文件中描述了整个工程所有文件的编译顺序、编译规则。Makefile有自己的书写格式、关键字、函数。像C语言有自己的格式、关键字和函数一样。而且在Makefile中可以使用系统shell所提供的任何命令来完成想要的工作。

- make在执行时，需要一个命名为Makefile的文件。这个文件告诉make以何种方式编译源代码和链接程序。
- make通过比较对应文件（规则的目标和依赖，）的最后修改时间，来决定哪些文件需要更新、那些文件不需要更新。
- 在执行make之前，需要一个命名为Makefile的特殊文件来告诉make需要做什么，该怎么做

---
## 1 GNU make介绍

### Makefile规则介绍

一个简单的Makefile描述规则组成：

        TARGET... : PREREQUISITES...
           COMMAND
           ...
           ...

- target：规则的目标。通常是最后需要生成的文件名或者为了实现这个目的而必需的中间过程文件名，目标也可以是一个make执行的动作的名称，如目标“clean”，可以make整个目录，也可以针对单个目录进行make
- make中的第一个target也是该MakeFile文件的最终目标
- prerequisites：规则的依赖。生成规则目标所需要的文件名列表。通常一个目标依赖于一个或者多个文件
- command：规则的命令行。是规则所要执行的动作（任意的shell命令或者是可在shell下执行的程序）。它限定了make执行这条规则时所需要的动作
- 一个规则可以有多个命令行，每一条命令占一行。注意：**每一个命令行必须以[Tab]字符开始**，[Tab]字符告诉make此行是一个命令行
- 命令就是在任何一个目标的依赖文件发生变化后重建目标的动作描述
- 一个目标可以没有依赖而只有动作（指定的命令）。比如Makefile中的目标“clean”，此目标没有依赖，只有命令。“clean”用来执行清理工作
- Makefile中把那些没有任何依赖只有执行动作的目标称为“伪目标”（phony targets）

一个Make示例

```MakeFile
#sample Makefile
myapp:main.o plus.o minus.o \
multi.o divi.o
    gcc main.o plus.o minus.o \
    multi.o divi.o -o myapp
main.o:main.c
    gcc -c main.c
plus.o:plus.c
    gcc -c plus.c
minus.o:minus.c
    gcc -c minus.c
multi.o:multi.c
    gcc -c multi.c
divi.o:divi.c
    gcc -c divi.c

#没有依赖，clean清除所有的.o中间文件，伪目标
.PHONY:clean
clean:
    rm -f *.o
    rm -f myapp
```
注意：

- 一行写不下，可以使用`\`换号，但是\后面不能有空格，回车后继续写后续代码即可
- Make的三要素：目标，依赖，命令

### make如何工作

默认的情况下，make执行的是Makefile中的第一个规则，此规则的第一个目标称之为“最终目的”或者“终极目标”，当在shell提示符下输入“make”命令以后。make读取当前目录下的Makefile文件，并将Makefile文件中的第一个目标作为其执行的“终极目标”，开始处理第一个规则（终极目标所在的规则）

上面示例的执行步骤为：

1\. 目标.o文件不存在，使用其描述规则创建它
2\. 目标.o文件存在，目标.o文件所依赖的.c源文件、.h文件中的任何一个比目标.o文件“更新”（在上一次make之后被修改）。则根据规则重新编译生成它
3\. 目标.o文件存在，目标.o文件比它的任何一个依赖文件（的.c源文件、.h文件）“更新”（它的依赖文件在上一次make之后没有被修改），则什么也不做。

### 定义变量和使用函数

上门示例虽然可以正常执行，但是MakeFile编写的过于繁琐，比如myapp的依赖和myapp下面的命令参数时一样的，这样就不利于后期修改，比如添加了新的依赖却忘记了在gcc命令中也加上，为了避免这个问题，在实际工作中大家都比较认同的方法是，使用一个变量“objects”、“OBJECTS”、“objs”、“OBJS”、“obj”或者“OBJ”来作为所有的.o文件的列表的替代。

make提供了一些函数和默认规则，可以用来简化MakeFile的编写

```MakeFile
#所有.c源文件，wildcard用于获取所有的.c文件
SOURCES=$(wildcard  *.c)
#patsubst把.c后缀，替换成.o后缀
OBJECTS=$(patsubst  %.c,%.o,$(SOURCES))

myapp:$(OBJECTS)
#自动化变量 $^表示所有依赖，$@表示目标
    gcc $^ -o $@

#通配符，比如：
#o:%.c -> main.o:main.c
#gcc -c $^ -o $@ -> gcc -c main.c -o main.o
%.o:%.c
    gcc -c $^ -o $@

.PHONY:clean
clean:
    - rm -f *.o
    - rm -f myapp
```

- wildcard
- patsubst
- `*`和`%`通配符
- 通过“.PHONY”特殊目标将“clean”目标声明为伪目标。避免当磁盘上存在一个名为“clean”文件时，目标“clean”所在规则的命令无法执行
- 在命令行之前使用“-”，意思是忽略命令“rm”的执行错误

### 递归展开式

示例：

```MakeFile
#所有.c源文件，wildcard用于获取所有的.c文件
SOURCES=$(wildcard  *.c)
#patsubst把.c后缀，替换成.o后缀
OBJECTS=$(patsubst  %.c,%.o,$(SOURCES))

#递归展开式
#可以引用还没有定义的变量，展开是引用时展开
str2=$(str1)
str1=hello

#直接展开式
#必须引用定义好了的变量，定义之后就会展开
str3 := android
str4 := $(str3)
str5 := $(str1) world

#变量的值追加
str5 += hello

#自定义函数
myfun=$2  $1
#变量等于函数的执行结构
myfun_ret=$(call myfun,20,10)

#加上`@`符号可以避免输出命令本身
test:
    @echo $(SOURCES)
    @echo $(OBJECTS)
    @echo $(str2)
    @echo $(str4)
    @echo $(str5)
    @echo $(myfun_ret)
    @echo $(call myfun,30,40)
```

---
## 2 Makefile总述

### Makefile的内容

在一个完整的Makefile中，包含了5个东西：**显式规则、隐含规则、变量定义、指示符和注释。**

- 显式规则：它描述了在何种情况下如何更新一个或者多个被称为目标的文件（Makefile的目标文件）。
- 隐含规则：它是make根据一类目标文件（典型的是根据文件名的后缀）而自动推导出来的规则。make根据目标文件的名，自动产生目标的依赖文件并使用默认的命令来对目标进行更新（建立一个规则）
- 变量定义：使用一个字符或字符串代表一段文本串，当定义了一个变量以后，Makefile后续在需要使用此文本串的地方，通过引用这个变量来实现对文本串的使用
- Makefile指示符：指示符指明在make程序读取makefile文件过程中所要执行的一个动作。其中包括：
- 读取一个文件，读取给定文件名的文件，将其内容作为makefile文件的一部分
- 决定（通常是根据一个变量的得值）处理或者忽略Makefile中的某一特定部分
- 定义一个多行变量
- 注释：Makefile中“#”字符后的内容被作为是注释内容

### makefile文件的命名

默认的情况下，make会在工作目录（执行make的目录）下按照文件名顺序寻找makefile文件读取并执行，查找的文件名顺序为：“GNUmakefile”、“makefile”、“Makefile”。
通常应该使用“Makefile”作为一个makefile的文件名。不推荐使用“GNUmakefile”，因为以此命名的文件只有“GNU make”才可以识别，而其他版本的make程序只会在工作目录下“makefile”和“Makefile”这两个文件。

当makefile文件的命名不是这三个任何一个时，需要通过make的“-f”或者“--file”选项来指定make读取的makefile文件

### 包含其它makefile文件

Makefile中包含其它文件所需要使用的关键字是“include”，和c语言对头文件的包含方式一致。"include”指示符告诉make暂停读取当前的Makefile，而转去读取“include”指定的一个或者多个文件，完成以后再继续当前Makefile的读取。Makefile中指示符“include”书写在独立的一行。

```MakeFile
# FILENAMES是shell所支持的文件名（可以使用通配符）
include FILENAMES...
```

通常指示符“include”用在以下场合：

- 有多个不同的程序，由不同目录下的几个独立的Makefile来描述其重建规则。它们需要使用一组通用的变量定义或者模式规则
- 当根据源文件自动产生依赖文件时；我们可以将自动产生的依赖关系保存在另外一个文件中，主Makefile使用指示符“include”包含这些文件。这样的做法比直接在主Makefile中追加依赖文件的方法要明智的多。

如果指示符“include”指定的文件不是以斜线开始（绝对路径，如/usr/src/Makefile...），而且当前目录下也不存在此文件；make将根据文件名试图在以下几个目录下查找：

- 首先，查找使用命令行选项“-I”或者“--include-dir”指定的目录
- 依此搜索以下几个目录（如果其存在）：“/usr/gnu/include”、“/usr/local/include”和“/usr/include”。
- 当在这些目录下都没有找到“include”指定的文件时，make将会提示一个包含文件未找到的告警提示，但是不会立刻退出。而是继续处理Makefile的后续内容。当完成读取整个Makefile后，make将试图使用规则来创建通过指示符“include”指定的但未找到的文件，当不能创建它时（没有创建这个文件的规则），make将提示致命错误并退出。

### 变量

#### MAKEFILES

如果在当前环境定义了一个“MAKEFILES”环境变量，make执行时首先将此变量的值作为需要读入的Makefile文件，多个文件之间使用空格分开。

如果文件名非绝对路径而且当前目录也不存在此文件，make会在一些默认的目录去寻找，参数上面**包含其它makefile文件**，另外：

- 环境变量指定的makefile文件中的“目标”不会被作为make执行的“终极目标”
- 环境变量所定义的文件列表，在执行make时，如果不能找到其中某一个文件（不存在或者无法创建）。make不会提示错误，也不退出
- make在执行时，首先读取的是环境变量“MAKEFILES”所指定的文件列表，之后才是工作目录下的makefile文件，“include”所指定的文件是在make发现此关键字的时、暂停正在读取的文件而转去读取“include”所指定的文件。

#### MAKEFILE_LIST

make程序在读取多个makefile文件时，包括由环境变量“MAKEFILES”指定、命令行指、当前工作下的默认的以及使用指示符“include”指定包含的，在对这些文件进行解析执行之前make读取的文件名将会被自动依次追加到变量“MAKEFILE_LIST”的定义域中。
这样我们就可以通过测试此变量的最后一个字来获取当前make程序正在处理的makefile文件名。具体地说就是在一个makefile文件中如果使用指示符“include”包含另外一个文件之后，变量“MAKEFILE_LIST”的最后一个字只可能是指示符“include”指定所要包含的那个文件的名字。

```MakeFile
name1 := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
include inc.mk
name2 := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
all:
@echo name1 = $(name1)
@echo name2 = $(name2)
```

执行make，则看到的将是如下的结果`name1 = Makefile name2 = inc.mk`

words函数说明

- 函数名称：统计单词数目函数—words。
- 函数功能：字算字串“TEXT”中单词的数目。
- 返回值：“TEXT”字串中的单词数。
- 示例：`$(words, foo bar)` 返回值是“2”。所以字串“TEXT”的最后一个单词就是：`$(word $(words TEXT),TEXT)`。

#### .VARIABLES

GNU make支持一个特殊的变量，此变量不能通过任何途经给它赋值。它被展开为一个特定的值。一个重要的特殊的变量是“.VARIABLES”。它被展开以后是此引用点之前、makefile文件中所定义的所有全局变量列表。包括：空变量（未赋值的变量）和make的内嵌变量，但不包含目标指定的变量，目标指定变量值在特定目标的上下文有效。

---
## 引用

- GNU make中文手册
- [跟我一起写Makefile](https://seisman.github.io/how-to-write-makefile/overview.html)