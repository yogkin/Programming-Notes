## Glide

### Glide介绍

- [Glide]( https://github.com/bumptech/glide )
- [Glide 4.0 中文文档](https://muyangmin.github.io/glide-docs-cn/)
- [placeholder影响ImageSize：issues](https://github.com/bumptech/glide/issues/363)
- [Glide系列综述](http://mrfu.me/2016/02/28/Glide_Series_Roundup/)
- [Glide显示图片进度的方案](https://github.com/bumptech/glide/issues/232)
- [Glide源码分析](http://www.lightskystreet.com/2015/10/12/glide_source_analysis/)

### 基于Glide的开源库

- [一个基于Glide的transformation库，拥有裁剪，着色，模糊，滤镜等多种转换效果 ]( https://github.com/wasabeef/glide-transformations)
- [一个可以在Glide加载时很方便使用Palette的库 ]( https://github.com/florent37/GlidePalette)


---
## Google Auto

Google的auto框架包括很多模块，利用注解帮开发者生成模板类，

资料：

```
    https://github.com/google/auto
    //自动生成代码 包括Gson和Parcelable都支持
    http://ryanharter.com/blog/2016/03/22/autovalue/
    //Auto生成parcel依赖声明必须放在Dagger2前面
    https://github.com/rharter/auto-value-parcel/issues/64
```

集成(gradle):

```groovy
squaeJavapoetApt          : 'com.squareup:javapoet:1.8.0',//注解处理器工具
autoValue                 : 'com.google.auto.value:auto-value:1.2',//https://github.com/google/auto/blob/master/value/userguide/index.md
autoValueApt              : 'com.google.auto.value:auto-value:1.2',
autoValueParcelApt        : 'com.ryanharter.auto.value:auto-value-parcel:0.2.5',//https://github.com/rharter/auto-value-parcel
autoValueParcelAdapterApt : 'com.ryanharter.auto.value:auto-value-parcel-adapter:0.2.5',
```

---
## agera

- [agera](https://github.com/google/agera)
- [agera-cn](https://github.com/captain-miao/AndroidAgeraTutorial/wiki)
- [retrofit-agera-call-adapter](https://github.com/drakeet/retrofit-agera-call-adapter)
- [agera-event-bus](https://github.com/drakeet/agera-event-bus)