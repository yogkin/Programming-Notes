# 如果解决重复依赖的冲突

有时候因为项目中依赖是开源项目较多，而多个开源项目中可能又有自己所依赖的其他库，这就是依赖传递，而有可能多个开源项目所依赖的其他库存在重复，这就造成了依赖冲突，默认情况下Gradle会聪明的寻找重复依赖中的最新版，比如：

我的项目的依赖配置如下：

```groovy
compile "com.android.support:appcompat-v7:25.1.0"
compile 'com.afollestad.material-dialogs:core:0.9.3.0'
...
...
此次省略30个其他的compile。
```
可以看到我的项目依赖了`appcompat-v7:25.1.0`和其他一些列项目，当我运行build后，应该IDEA中的`External Libraries`中看到`appcompat-v7:25.1.0`库，但是由于某个项目依赖的是`appcompat-v7:25.1.1`，所以造成了项目构建最终选择的是`appcompat-v7:25.1.1`，假如`appcompat-v7:25.1.1`有一个bug正好影响到了我的项目，我想强制要求构建使用`appcompat-v7:25.1.0`版本，如何解决呢？

## 1 强制所有依赖某个版本


```groovy
//强制某种配置
    configurations.compile {
        resolutionStrategy.force(
            "android.arch.core:runtime:1.1.1",
            "android.arch.lifecycle:livedata:1.1.1",
            "android.arch.lifecycle:viewmodel:1.1.1",
            "android.arch.lifecycle:extensions:1.1.1",
            "android.arch.lifecycle:livedata-core:1.1.1",
            "android.arch.lifecycle:runtime:1.1.1"
        )
    }
    
//强制所有配置
   configurations.all {
    resolutionStrategy.force(
            "android.arch.core:runtime:1.1.1",
            "android.arch.lifecycle:livedata:1.1.1",
            "android.arch.lifecycle:viewmodel:1.1.1",
            "android.arch.lifecycle:extensions:1.1.1",
            "android.arch.lifecycle:livedata-core:1.1.1",
            "android.arch.lifecycle:runtime:1.1.1"
    )
}
```

---
## 2 显式指定某个库的确定版本，让Gradle自动覆盖

```
//如果某一第三方库依赖，com.android.support:appcompat-v7:25.1.0
//但是现在使用的是27.1.0版本，则显式的在该模块中声明一次
implementation com.android.support:appcompat-v7:25.1.0
```

---
## 3 找出哪个项目依赖了`appcompat-v7:25.1.1`，然后排除它对`appcompat-v7:25.1.1`的依赖


### step 1


执行`gradlew dependences`命令，查看项目依赖的详细信息：


```groovy
    -----------------------------------------------------------
    Project :app
    ------------------------------------------------------------
    ......省略
    _debugApk - ## Internal use, do not manually configure ##
    +--- com.android.support:multidex:1.0.1
    +--- project :AndroidBase
    |    +--- com.jakewharton.timber:timber:4.5.1
    |    +--- com.android.support:appcompat-v7:25.1.0 -> 25.1.1
    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    +--- com.android.support:support-v4:25.1.1
    |    |    |    +--- com.android.support:support-compat:25.1.1
    |    |    |    |    \--- com.android.support:support-annotations:25.1.1
    |    |    |    +--- com.android.support:support-media-compat:25.1.1
    |    |    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    |    |    \--- com.android.support:support-compat:25.1.1 (*)
    |    |    |    +--- com.android.support:support-core-utils:25.1.1
    |    |    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    |    |    \--- com.android.support:support-compat:25.1.1 (*)
    |    |    |    +--- com.android.support:support-core-ui:25.1.1
    |    |    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    |    |    \--- com.android.support:support-compat:25.1.1 (*)
    |    |    |    \--- com.android.support:support-fragment:25.1.1
    |    |    |         +--- com.android.support:support-compat:25.1.1 (*)
    |    |    |         +--- com.android.support:support-media-compat:25.1.1 (*)
    |    |    |         +--- com.android.support:support-core-ui:25.1.1 (*)
    |    |    |         \--- com.android.support:support-core-utils:25.1.1 (*)
    |    |    +--- com.android.support:support-vector-drawable:25.1.1
    |    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    |    \--- com.android.support:support-compat:25.1.1 (*)
    |    |    \--- com.android.support:animated-vector-drawable:25.1.1
    |    |         \--- com.android.support:support-vector-drawable:25.1.1 (*)
    |    +--- com.android.support:recyclerview-v7:25.1.0 -> 25.1.1
    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    +--- com.android.support:support-compat:25.1.1 (*)
    |    |    \--- com.android.support:support-core-ui:25.1.1 (*)
    |    +--- com.android.support:design:25.1.0
    ......省略很多很多
```


当面多如此多的信息时，如果快速找到哪个库依赖了`support-compat:25.1.1`呢，可以看到上面的信息中有：
`com.android.support:appcompat-v7:25.1.0 -> 25.1.1`，这个标识`support:appcompat-v7`由25.1.0变为了25.1.1，导致下面级联的库都是用了25.1.1，我们找的信息是直接声明`com.android.support:support-annotations:25.1.1`而没有`25.1.0 -> 25.1.1`的信息，这就标识某个库直接依赖了`25.1.1`版本，比如：


```groovy
    +--- com.afollestad.material-dialogs:core:0.9.3.0
    |    +--- com.android.support:support-v13:25.1.1
    |    |    +--- com.android.support:support-annotations:25.1.1
    |    |    \--- com.android.support:support-v4:25.1.1 (*)
    |    +--- com.android.support:appcompat-v7:25.1.1 (*)
    |    +--- com.android.support:recyclerview-v7:25.1.1 (*)
    |    +--- com.android.support:support-annotations:25.1.1
```


这时就可以确认是`material-dialogs`这个库依赖了`25.1.1`版本，只需要排除他对support库的依赖即可。


### step 2


```groovy
        compile('com.afollestad.material-dialogs:core:0.9.3.0') {
            exclude group: "com.android.support"
        }
```

