---
## productFlavors

3.0 的 productFlavors 必须属于一个 flavorDimensions

```groovy
flavorDimensions "tier", "minApi"

  productFlavors {
       free {
        // 这个 product flavor 属于 "tier" flavor dimension
        // 如果只有一个 dimension 那么这个属性就是可选的
        dimension "tier"
        ...
      }
      paid {
        dimension "tier"
        ...
      }
      minApi23 {
          dimension "minApi"
          ...
      }
      minApi18 {
          dimension "minApi"
          ...
      }
  }
```

---
## 如何依赖不同buildType的library

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

---
## 本地依赖aar文件

Android开发中，依赖本地aar文件与jar文件有所不同，gradle不能像直接依赖jar那样依赖aar文件，依赖aar文本需要把改问题天添加到本地仓库中，在进行依赖。


示例：

```groovy
apply plugin: 'com.android.application'

android {
     .....
}

//把libs设置为本地仓库
repositories{
    flatDir{
        dirs 'libs'
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    //依赖arr
    compile(name:'aar_name', ext:'aar')
}
```



