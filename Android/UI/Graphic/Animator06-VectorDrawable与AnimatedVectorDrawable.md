# SVG

---
## 1 概念

SVG，即 Scalable Vector Graphics 可伸缩矢量图形，这种图像格式在前端中已经使用的非常广泛了。

首先要解释下什么是矢量图像，什么是位图图像？ 

1. 矢量图像：SVG是W3C推出的一种开放标准的文本式矢量图形描述语言，它是基于XML的、专门为网络而设计的图像格式，SVG是一种采用XML来描述二维图形的语言，所以它可以直接打开xml文件来修改和编辑。 
2. 位图图像：位图图像的存储单位是图像上每一点的像素值，因而文件会比较大，像GIF、JPEG、PNG等都是位图图像格式。

---
## 2 Vector Android

Android中的是Vector Drawable，也就是Android中的矢量图，可以说Vector就是Android中的SVG实现，Vector图像刚发布的时候，是只支持Android 5.0+的，自从AppCompat 23.2之后，Vector可以使用于Android 2.1以上的所有系统，只需要引用com.android.support:appcompat-v7:23.2.0以上的版本就可以了。（所谓的兼容也是个坑爹的兼容，即低版本非真实使用SVG,而是生成PNG图片）
    
### Vector Drawable

Android 5.0发布的时候，Google提供了Vector的支持，即：Vector Drawable类。Vector Drawable相对于普通的Drawable来说，有以下几个好处：

- Vector图像可以自动进行适配，不需要通过分辨率来设置不同的图片。
- Vector图像可以大幅减少图像的体积，同样一张图，用Vector来实现，可能只有PNG的几十分之一。
- 使用简单，很多设计工具，都可以直接导出SVG图像，从而转换成Vector图像，功能强大。
- 不用写很多代码就可以实现非常复杂的动画 成熟、稳定，前端已经非常广泛的进行使用了。

### Vector 语法简介

通过使用SVG的Path标签，几乎可以实现SVG中的其它所有标签，虽然可能会复杂一点，但这些东西都是可以通过工具来完成的。

```
        Path指令解析如下所示：
                M = moveto(M X,Y) ：将画笔移动到指定的坐标位置，相当于 android Path 里的moveTo()
                L = lineto(L X,Y) ：画直线到指定的坐标位置，相当于 android Path 里的lineTo()
                H = horizontal lineto(H X)：画水平线到指定的X坐标位置 
                V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置 
                C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：三次贝赛曲线 
                S = smooth curveto(S X2,Y2,ENDX,ENDY) 同样三次贝塞尔曲线，更平滑 
                Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：二次贝赛曲线 
                T = smooth quadratic Belzier curveto(T ENDX,ENDY)：映射 同样二次贝塞尔曲线，更平滑 
                A = elliptical Arc(A RX,RY,XROTATION,FLAG1,FLAG2,X,Y)：弧线 ，相当于arcTo()
                Z = closepath()：关闭路径（会自动绘制链接起点和终点）

                注意，’M’处理时，只是移动了画笔， 没有画任何东西。
```


注意：关于这些语法，开发者不需要全部精通，而是能够看懂即可，这些path标签及数据生成都可以交给工具来实现。
使用AndroidStudio导入SVG图片可以直接生成xml文件，AS会自动生成兼容性图片(高版本会生成xxx.xml的SVG图片；低版本会自动生成xxx.png图片)


###  静态Vector图像


```xml
            生成的一个图片

            <vector android:alpha="0.78" android:height="24dp"
                android:viewportHeight="24.0" android:viewportWidth="24.0"
                android:width="24dp" xmlns:android="http://schemas.android.com/apk/res/android">
                <path android:fillColor="#FF000000" android:pathData="M19.35,10.04C18.67,6.59 15.64,4 12,4 9.11,4 6.6,5.64 5.35,8.04 2.34,8.36 0,10.91 0,14c0,3.31 2.69,6 6,6h13c2.76,0 5,-2.24 5,-5 0,-2.64 -2.05,-4.78 -4.65,-4.96zM14,13v4h-4v-4H7l5,-5 5,5h-3z"/>
            </vector>
            
            一个矩形

            <vector xmlns:android="http://schemas.android.com/apk/res/android"
                android:width="200dp"
                android:height="200dp"
                android:viewportHeight="500"
                android:viewportWidth="500">

                <path
                    android:name="square"
                    android:fillColor="#000000"
                    android:pathData="M100,100 L400,100 L400,400 L100,400 z"/>

            </vector>
```

解释头部的几个标签：

- `android:width \ android:height`:定义图片的宽高
- `android:viewportHeight \ android:viewportWidth`:定义图像被划分的比例大小，例如例子中的500，即把200dp大小的图像划分成500份，后面Path标签中的坐标，就全部使用的是这里划分后的坐标系统。这样做有一个非常好的作用，就是将图像大小与图像分离，后面可以随意修改图像大小，而不需要修改PathData中的坐标。

### AndroidSupportLibrary 23.2对Vector的兼容

Vector drawables 让你可以用一个定义在XML里的矢量图象替换多个png资源。而之前这一用法只局限于Lollipop以及更高的设备，VectorDrawable和AnimatedVectorDrawable现在可以分别通过两个新的支持库support-vector-drawable 和 support-animated-vector-drawable得到。

- VectorDrawableCompat 兼容到 API7
- AnimatedVectorDrawableCompat 兼容到 API11

使用这两个库需要在 build.gradle 文件里添加 `vectorDrawables.useSupportLibrary = true`

```groovy
 android {  
   defaultConfig {  
     vectorDrawables.useSupportLibrary = true  
    }  
 }
```

鉴于安卓加载drawable的方式，并不是每个接受 drawable id 的地方（比如在一个XML文件中）都支持加载VectorDrawable，AppCompat 添加了几个功能让你更容易使用新的VectorDrawable。

当ImageView和（或者例如 ImageButton 和 FloatingActionButton这样的子类）使用AppCompat一起使用的时候，可以使用新的`app:srcCompat`属性来引用VectorDrawable。

```xml
<ImageView  
  android:layout_width="wrap_content"  
  android:layout_height="wrap_content"  
  app:srcCompat="@drawable/ic_vector" />
```

在 Lollipop之前直接在 `app:srcCompat` 之外引用 VectorDrawable 会失败。但是 AppCompat 却支持使用其它 drawable 容器来间接的引用 VectorDrawable，比如使用` StateListDrawable, InsetDrawable, LayerDrawable, LevelListDrawable, 或者 RotateDrawable` 来加载VectorDrawable。比如TextView的 `android:drawableLeft` 属性，本来在正常情况下，它是不支持VectorDrawable的。现在只要在 App 启动的时候调用 `AppCompatDelegate.setCompatVectorFromResourcesEnabled()`即可。

### AnimatedVectorDrawable

AnimatedVectorDrawable的作用是给VectorDrawable提供动画效果。
在XML文件中通过`<animated-vector>`标签来声明对AnimatedVectorDrawable的使用，并指定其作用的`<path>或<group>`。

### VectorDrawable 的性能问题

1. Bitmap的绘制效率并不一定会比Vector高，它们有一定的平衡点，当Vector比较简单时，其效率是一定比Bitmap高的，所以，为了保证Vector的高效率，Vector需要更加简单，PathData更加标准、精简，当Vector图像变得非常复杂时，就需要使用Bitmap来代替了
2. Vector适用于ICON、Button、ImageView的图标等小的ICON，或者是需要的动画效果，由于Bitmap在GPU中有缓存功能，而Vector并没有，所以Vector图像不能做频繁的重绘
3. vector图像过于复杂时，不仅仅要注意绘制效率，初始化效率也是需要考虑的重要因素

---
## 引用

### blog

- [W3C SVG](http://www.w3school.com.cn/svg/svg_intro.asp)
- [微信引入的SVG技术](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207863967&idx=1&sn=3d7b07d528f38e9f812e8df7df1e3322&scene=4#wechat_redirect)
- [Vectors时代(1) - SVG 格式缩减应用图像](http://www.wangchenlong.org/2016/03/15/replace-svg-image/)
- [Vectors时代(2) - 图像的路径动画](http://www.wangchenlong.org/2016/03/15/svg-path-animation/)
- [Vectors For All (最终篇)](http://gold.xitu.io/entry/574d7894df0eea005bcfb10b?utm_source=gold-miner&utm_medium=readme&utm_campaign=github)
- [Android Vector曲折的兼容之路](http://www.jianshu.com/p/e3614e7abc03)

### libraries

- [RoadRunner](https://github.com/glomadrian/RoadRunner)
- [android可缩放矢量图](https://github.com/pixplicity/sharp)
- [Android实现炫酷SVG动画效果](http://blog.csdn.net/crazy__chen/article/details/47728241)
- [Android对SVG的支持](http://daycoding.com/2015/08/24/android%E5%AF%B9svg%E7%9F%A2%E9%87%8F%E5%9B%BE%E7%9A%84%E6%94%AF%E6%8C%81/)
- [sharp - svgLibrary](https://github.com/Pixplicity/sharp)
- [绘制允许任何对SVG图形复杂的变形动画工具](https://github.com/bonnyfone/vectalign#vectalign----)
- [AnimatedSvgView](https://github.com/jaredrummler/AnimatedSvgView)