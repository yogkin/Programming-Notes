# Uri介绍

##  Android中的 Uri 与 Java 中的 URI 的区别

*  所属的包不同：URI位置在 `java.net.URI`，显然是Java提供的一个类。而Uri位置在`android.net.Uri`，是由 Android 提供的一个类。所以初步可以判断，`Uri`是URI的“扩展”以适应Android系统的需要。
*  作用的不同：
    - URI类代表了一个URI（这个URI不是类，而是其本来的意义：通用资源标志符——Uniform Resource Identifier)实例。
    - Uri类是一个不可改变的URI引用，包括一个URI和一些碎片，URI跟在“#”后面。建立并且转换URI引用。而且Uri类对无效的行为不敏感，对于无效的输入没有定义相应的行为，如果没有另外制定，它将返回垃圾而不是抛出一个异常。Uri遵循的是RFC 2396标准

##  Uri结构

具体参考：

- [1、《Uri详解之-Uri结构与代码提取》](http://blog.csdn.net/harvic880925/article/details/44679239)
- [2、《Uri详解之-通过自定义Uri外部启动APP与Notification启动》](http://blog.csdn.net/harvic880925/article/details/44781557)

## 常用Uri

```
    assets中的文件：file:///android_asset/index.html
    资源文件：android.resource://+context.getPackageName()+"/"+resourceId
```

