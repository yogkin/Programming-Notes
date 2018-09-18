# 如何依赖不同buildType的library

默认情况下，主工程依赖所以来的module的构建类型都是release的，这个不随着主工程的构建类型而改变，那么如果依赖不同buildType的library呢？

```groovy
    //library
    android {
        publishNonDefault true
    }
    //主工程
    dependencies {
        releaseCompile project(path: ':base', configuration: 'release')
        debugCompile project(path: ':base', configuration: 'debug')
    }
```

这样显得比较麻烦，可以通过代码让配置变得灵活一些

```groovy
        //library
        android {
            publishNonDefault true
        }
        //主工程
        dependencies {
               //基础类库
              compile project(path: ':base', configuration: libraryBuildType)
        }
```

然后我们可以在build.properties或者命令行中指定libraryBuildType

