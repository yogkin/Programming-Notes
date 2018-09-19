# 本地依赖aar文件

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



