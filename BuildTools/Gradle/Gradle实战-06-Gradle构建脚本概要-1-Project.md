# Gradle构建脚本概要

---
## 1 构件块


每个Gradle构建都包括三个基本构件块：`project、task、property`。每一个构建至少包含一个project，进而由包含一个或多个task。


Gradle中最基本的两个概念是 **project和task**，在多项目构建中，一个project可能依赖于另一个project。


---
## 2 Project


在Gradle术语中，一个项目(project)代表一个正在构建的组件(比如一个jar文件)，或者一个想要完成的目标。Gradle基于`build.gradle`中的配置实例化`org.gradle.api.Porject`类。并且通过project变量使其隐式可用,**Project可以插创建新的task，添加依赖关系和配置，并且应用插件和其他构建脚本**。下面是Project类一些重要的API:


构建脚本配置

```
    project.apply()
    project.buildscript {}
```

依赖管理

```
    project.dependencies {}
    project.configurations {}
    project.getDependencies()
    project.getConfigurations()
```

属性

```
    project.getAnt()
    project.getName()
    project.getDependencies()
    project.getGroup()
    project.getPath()
    project.getVersion()
    project.getLogger()
    project.setVersion()
    project.setDescription()
```

创建文件

```
    project.file() project提供file方法，它会创建相对于项目目录的java.io.File实例
    project.files()
    project.fileTree()
```

创建task

```
    Task task(String name) throws InvalidUserDataException;
    Task task(Map<String, ?> args, String name) throws InvalidUserDataException;
    Task task(Map<String, ?> args, String name, Closure configureClosure);
    Task task(String name, Closure configureClosure);
```

在`build.gradle`脚本中，不需要显式的使用project变量就可以访问Project的API，由于Groovy的可动态编程，实际运行过程Gradle会把这些API调用转移到Project对象上。


```
setDescription("Gradle构件块示例")//设置Project的描述，实际调用的时Project对象的void setDescription(String description);方法
```

