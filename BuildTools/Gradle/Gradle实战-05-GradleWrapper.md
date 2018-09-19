# GradleWrapper

---
## 1 GradleWrapper介绍


GradleWrapper是gradle的核心特性，他能够让机器在没有安装gradle的运行时的情况下运行gradle构建，它让脚本运行在一个指定的gradle版本上，它是通过自动从中心仓库下载gradle运行时，解压和使用来实现的，最终的目标是创建一个独立于系统，系统配置和gradle版本的可靠的和可重复的构建。


**使用Gradle被认为是最佳实践，对于每一个gradle项目都是必需的，由包装器支持的gradle版本非常适合作为自动化发布过程的一部分。**


关键是由gradle包装器执行的构建脚本提供了与本地已安装的Gradle完全相同的任务、特性和行为。


---
## 2 使用GradleWrapper


使用GradleWrapper只需要在脚本中配置wrapper任务：


    task wrapper(type:Wrapper){//指定一个Wrapper类型的任务
        gradleVersion = '2.4'   //指定gradle版本
    }


然后运行`gradle wrapper`命令，第一次运行会下载指定版本的gradle版本，在项目中会生存gradle wrapper的相关配置，**这些配置文件应该包含到版本控制系统中去**


```
    │  build.gradle
    │  gradlew
    │  gradlew.bat //执行gradle命令的包装器脚本
    │
    └─gradle
        └─wrapper           //wrapper配置目录
                gradle-wrapper.jar    //包含下载和解压gradle运行时的逻辑
                gradle-wrapper.properties//包装器源信息包含以下载gradle运行时的存储位置和原始url
```


配置好gradle wraper之后，就可以使用`gradlew`命令来执行构建任务了。


### GradleWrapper作用

当其他人从版本库中拉取你的项目时，即使他们没有安装gradle，但是使用`./gradlew`相关命令时，**gradleWrapper**会检查正确版本的gradle是否被安装，如有必要会帮你下载正确的版本，这就是

---
## 2 定制包装器


上面说到第一次运行`gradle wrapper`会从gradle项目中心服务器下载gradle的压缩文件，默认是存储在`$HOME_DIR/.gradle/wraper/dists`目录下


这也是可以定制的


```
    task wrapper(type:Wrapper){
        gradleVersion = '2.4'
        distributionUrl="http://myServer.com/gradle/dists"    //指定过去gradle包装器的url
        distributionPath = 'gradle-dists'    //包装器被解压缩后存放的相对路径
    }
```



---
## 3 Android中配置GradleWrapper缓存目录(扩展)

关于Gradle要说明的是，假如你是用 **android studio新建的项目**，它会自动为新建的工程添加 wrapper。也就是你就不必自己再去创建 gradle wrapper 了，直接在命令行或者你的编译脚本中使用就好了。

- [Gradle 缓存的文件名是如何生成的？](http://gold.xitu.io/entry/57cbb20cc4c971005435147f)
- [Android-Studio 缓存文件夹配置](http://blog.csdn.net/qiujuer/article/details/44160127)
- [Android Studio Gradle 缓存文件夹设置](http://blog.csdn.net/qiujuer/article/details/44257993)
- [Gradle的一些总结](http://www.zircon.me/06-25-2015/about-gradle.html)

