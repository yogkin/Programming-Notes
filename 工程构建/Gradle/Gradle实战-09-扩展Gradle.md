# 扩展Gradle

插件让重用和扩展的概念走的更远，它们通过引入某个特定的问题领域的约定和模式为项目添加新的能力。比如java、web、war插件。

---
## 1 插件分类

Gradle将插件分为两大类，**脚本插件和对象插件**。

- 脚本插件只不过是一个普通的Gradle构建脚本，它可以被导入到其他构建脚本中，就像平时我们写Gradle脚本一样。
- 对象插件需要是实现`org.gradle.api.Plugin`接口，对象插件的源代码通常放在buildSrc目录下。


### 应用仓库插件的方式

```groovy
    //buildscript模块用于配置构建脚本
    buildscript {
        //这里配置的构建脚本的类库仓库
        repositories {//使用mavenCentral解析已声明的依赖
            mavenCentral()
        }

        dependencies {
            classpath 'com.xxx:1.4.0'//将某某插件API添加到构建脚本的classpath中
        }
    }
```

---
## 2 脚本插件

### 2.1 直接编写脚本插件

脚本插件就是新建一个gradle脚本，在脚本中写入一些脚本代码，然后在项目中使用apply方法引用脚本。

plugin.gradle：

```groovy
    def funcA() {
        println "FuncA run"
    }

    task pluginTask {
        doLast {
            funcA()
        }
    }
```

build.gradle:

    apply from: 'plugin.gradle'//使用apply from来应用外脚本插件，可以是本地的脚本插件也可以是远程(url)的脚本插件


上面使用的`apply from: 'plugin.gradle'`表示应用当前目录下的一个文件名为`plugin.gradle`的脚本。


### 2.2 使用定制task实现

在buildsrc下定制任务。这种方式不需要发布插件，其他模块可以直接引用buildsrc下的插件。

---
## 3 对象插件

一般都是通过扩展`org.gradle.api.Plugin`实现对象插件的。然后可以通过maven发布到本地仓库或者远程仓库。

---
## 4 注意

对于`project.extensions.create()`获取的extension，应该在task执行s时才去获取。


---
## 引用

- [Gradle深入与实战（五）自定义插件](http://benweizhu.github.io/blog/2015/03/15/deep-into-gradle-in-action-5/)
- [用AndroidStudio开发自定义 Gradle plugin](https://github.com/helen-x/gradle-plugin-demo)

