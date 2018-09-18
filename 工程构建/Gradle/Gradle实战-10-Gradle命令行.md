# Gradle命令行

Gradle命令行接口(CLI)包含三个部分：

- 探索和帮助相关的task
- 构建设置task
- 配置输入

gradle命令的用法：`gradle [option...] [task...]`

---
##  探索类


task|描述
------|------
dependencies |列出项目的依赖，包括传递性依赖
dependencyInsight|解释在依赖图中一个依赖如何被选择，为什么被选择，`--dependency`用于指定依赖，`--configuration`用于指定非compile的配置
help|显示Gradle命令行的基本用法
projects|列出参与构建的项目
properties|列出项目中所有可用的属性
tasks|列出所有的可运行的task，使用`--all`命令选项可以更详细的列出task之间的依赖

## 配置输入


构建配置信息可以通过CLI提供，不需要提供值的选项可以组合使用。比如同时使用`-is`

选项|描述
------|------
-?,-h,--help|打印出所有可用的命令选项
-a,--no-rebuild|避免重新构建多项目中的所有子项目，部分构建
-b,-build-file|Gradle构建脚本默认的命名约定是build.gralde,通过此选项可以改变
-c,--setting.file|同上，改变settings.gradle的默认名
--continue|在一个task执行失败后Gradle会继续构建，在多项目中很有用，用于发现所有的问题，并一起修复
--configure-on-demand|优化多项目初始化项目构建的配置时间，只配置正在请求的task相关项目，在grale.properties中使用`org.gradle.configure-ondemand`属性激活
-g,--gralde-user-home|配置gradle的home目录
--gui|运行一个基于Swing的图像化用户界面
-I,iiinit-script|设置一个初始化脚本用于构建，整个脚本在所有构建task之前被执行
-p,--project-dir|默认的gralde任务会在当前目录进行构建，通过此选项指定特定目录下的项目进行构建
--parallel|并行构建参与多项目构建的子项目，Gradle会自动确定最优的线程数量，在grale.properties中使用`org.gradle.parallel`属性激活
--parallel-threads|指定并行构建的线程数:`--parallel-threads=5`
-m,--dry-run|打印task的执行顺序，而不真的执行它们
--profile|列出所有task执行和阶段使用的时间
-v|版本信息
-x,--exclude-task|指定构建时按个task不执行`gradle -x test check build`表示不执行test和check进行构建


## 属性选项


选项|描述
------|------
-D,system-prop|指定系统属性`-Dkey=value`
-P,--project-prop|指定项目属性`-Pkey=value`


## 日志选项


选项|描述
------|------
-i,--info|默认情况下Gradle构建并不会输出大量的构建日志信息，通过选项把日志级别设置为INFO以便输出更多的信息
-d,--debug|以Debug日志级别运行Gradle构建
-q,--quite|减少构建时的错误信息
-s,--stacktrace|打印简单的构建时堆栈异常信息
-S,--fule-stacktrace|打印完整的堆栈异常信息

## 缓存选项

选项|描述
------|------
--offline|使用离线模式构建
--project-cahce-dir|指定依赖缓存目录
--recompile-scripts|Gradle默认编译所有的脚本并缓存以提高效率，此命令用于重新编译脚本
--refresh-dependencies|手动刷新缓存中的依赖，强制检查依赖版本

## 后台守护进程选项

选项|描述
------|------
--daemon|使用后台进程，在grale.properties中使用`org.gradle.daemon`属性激活
--foreground|在终端运行Gradle后台守护进程，用于调试和监控目的
--no-daemon|不使用已有的守护进程
--stop|停止守护进程

