#  Drawable学习

## 1 Drawable简介

Drawable表示的是一种可以在canvas上进行绘制的抽象概念，在安卓中Drawable有很多种：

 - BitmapDrawable 位图
 - NimePatchDrawable .9图
 - ShapeDrawable 形状drawable
 - LayerDrawable 层级drawable容器
 - StateListrawable 状态drawable容器
 - AnimatedStateListDrawable 带动画的StateListrawable(Api21)
 - LevelListDrawable 等级drawable容器
 - TransitionDrawable 过度
 - InsetDrawable 内嵌
 - ScaleDrawable 缩放
 - ClipDrawable 裁剪
 - RotateDrawable 旋转
 - AnimationDrawable 动画
 - AnimatedRotateDrawable 旋转动画
 - RippleDrawable(Api21)

Drawable表示一种图像的概念，但他们又不全是图片，通过颜色也可以构造出各式各样的图形效果，Drawable一般通过xml来定义，但是自定义的Drawable除外，在xml中定义的Drawable都会有一个Java类对应，在解析这个xml时，Android会创建对应的Drawable对象。

#### Drawable的内部宽高

Drawable的内部宽高这两个参数比较重要，通过getIntrinsicWidth和getIntrinsicHeight来获取(intrinsic译作固有的)，但不是所有的Drawable都有宽高，这个得分类讨论

- 比如一张图片形成的Drawable，他的内部宽高就是图片的宽高
- 但是一个图片形成的Drawable他就没有内部宽高的概念

一般来说，Drawable没有大小的概念，当用作view的背景时，Drawable会被拉伸到view 的同等大小

---
## 2 Drawable的分类详解

### RippleDrawable

ripple用于表示一个涟漪，语法如下：

    <?xml version="1.0" encoding="utf-8"?>
    <ripple xmlns:android="http://schemas.android.com/apk/res/android"  
            android:color="@color/colorAccent" android:radius="3dp">
        <item android:drawable="@color/colorPrimary"/>
    </ripple>

其中color是必须的，其次当没有定义item时，涟漪是圆形并且超出控件范围的涟漪，如果定义了item，那么涟漪范围在控件的范围内


### vector/animated-vector

用于表示一个矢量图

### BitmapDrawable

表示一张图片，在实际开发中直接引用原图片即可，但是也可以通过xml方式来描述它，且可以设置更多的效果，如下代码：

```
    <bitmap xmlns:android="http://schemas.android.com/apk/res/android"
            android:src="@com.ztiany:mipmap/ic_launcher"
            android:antialias="true"
            android:dither="true"
            android:filter="true"
            android:gravity="center"
            android:mipMap="true"
            android:tileMode="clamp"
        >
    </bitmap>
```
- src 指定引用的图片，格式为@[packageName:]drawable/drawable_src,**必须属性**
- antialias 是否开启抗锯齿功能，开启后图片变得平滑，在一定程度上降低图片的清晰度(忽略不计)，一般应该开启
- dither 是否开启抖动效果，防止图片失真，一般应该开启
- filter 是否开启过滤效果，当图片被拉伸或者压缩时，开启过滤效果可以保证较好的显示效果。
- mipMap 一种图形相关的处理技术，纹理技术，默认为false
- tileMode 平铺模式，默认关闭，当开启屏幕模式之后gravity失效，这个选项有如下几个值：
 - disabled 关闭屏幕模式，默认值
 - clamp 图片四周的像素会被拉伸到周围区域
 - repeat 简单的水平方向和垂直方向上的平铺效果
 - mirror 水平方向和垂直方向上的镜像投影效果
- gravity 当图片的尺寸小于容器的大小时，可以使用此属性来对图片进行定位，多个属性可以用"|"来组合，下面用一个表格来说明各个属性的效果

|  可选值 | 含义  |
| ------------ | ------------ |
|  top |  图片放在容器顶部，不改变图片大小 |
| bottom  |  图片放容器在底部，不改变图片大小 |
| left  |  图片放在容器左部，不改变图片大小 |
| right  | 图片放在容器右部，不改变图片大小  |
| center_vertical  |  图片垂直剧中，不改变图片大小 |
| fill_vertical  | 图片垂直方向填充  |
| center_horizontal  |  图片水平居中中，不改变图片大小 |
| fill_horizontal  | 图片水平方向填充  |
| center  |  图片水平和垂直方向都居中，不改变图片大小 |
| fill  |  图片水平方向和垂直方向都填充，**这是默认效果** |
| clip_vertical  | 附加项，表示垂直方向的裁剪，较少使用  |
| clip_horizontal  |  附加项，表示水平方向的裁剪，较少使用  |


### NimePatchDrawable

表示一张.9图片，也可以直接引用，属性与BitmapDrawable差不多

```
    <nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
                android:src="@drawable/zoom_plate"
                android:tint="@color/colorAccent"//api21才有效果
                android:tintMode="src_in"//api21才有效果
        >
    </nine-patch>
```


### GradientDrawable

GradientDrawable对应的xml标签是shape

可以理解为通过颜色来构造图形，它既可以表示纯色图形，也可以是具有渐变效果的图形，语法如下：
```
    <?xml version="1.0" encoding="utf-8"?>
    <shape
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:shape=["rectangle" | "oval" | "line" | "ring"] >
        //圆角属性，只对rectangle有效
        <corners
            android:radius="integer"
            android:topLeftRadius="integer"
            android:topRightRadius="integer"
            android:bottomLeftRadius="integer"
            android:bottomRightRadius="integer" />

            //渐变属性，只对rectangle，oval有效
        <gradient
            android:angle="integer" //只有linear类型的渐变才有效,表示渐变角度,必须为45的倍数
            android:centerX="integer"  //渐变中间颜色的X坐标,取值范围为:0~1
            android:centerY="integer"//渐变中间颜色的Y坐标,取值范围为:0~1
            android:centerColor="integer"
            android:endColor="color"
            android:gradientRadius="integer"  //只有radial和sweep类型的渐变才有效,radial必须设置,表示渐变效果的半径
            android:startColor="color"
            android:type=["linear" | "radial" | "sweep"]//线性渐变(可设置渐变角度),发散渐变(中间向四周发散),平铺渐变
            android:useLevel=["true" | "false"] />//当drawable作为StateListDrawable时为true，是否根据level绘制渐变效果
            //表示的是view的空白，而不是自身的空白
        <padding
            android:left="integer"
            android:top="integer"
            android:right="integer"
            android:bottom="integer" />
            //表示shape的宽高，表示shape的固定大小intrinsicWidth和intrinsHeight，但并不是shape的最终显示高度，当shape作为view的背景时，会被拉伸或缩小为view的大小，
        <size
            android:width="integer"
            android:height="integer" />
            //填充颜色
        <solid
            android:color="color" />
            //描边与填充互斥
        <stroke
            android:width="integer" //线的厚度
            android:color="color"// 颜色
            android:dashWidth="integer" // 虚线的厚度
            android:dashGap="integer" /> 线段间隔
    </shape>
```

shape表示形状，上面分别是矩形，椭圆，线，圆环，另外ring还有一些特殊
```
    android:innerRadius 圆环内半径，优先级高于innerRadiusRatio
    android:innerRadiusRatio 内半径占整个高度的比例，默认是9，如果是n，那么内半径=宽度/n
    android:thickness 圆环厚度，外半径-内半径，优先级高于thicknessRatio
    android:thicknessRatio 厚度占整个宽度的比例，默认为3，如果是n，那么 厚度=宽度/n
    android:useLevel 一般都为false，除非被当作LevelDrawable
```


### LayerDrawable

表示一种层次化的drawable，是一种叠加后的效果，他的语法如下

```
    <?xml version="1.0" encoding="utf-8"?>
    <layer-list
        xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:id="@[+][package:]id/resource_name"
            android:top="dimension"
            android:right="dimension"
            android:bottom="dimension"
            android:left="dimension" />
    </layer-list>
```

layer-list语法很简单，可以包含多个item，每个item表示一个drawable，top，right，bottom，left分别表示各个方向上的偏移量

### StateListrawable

对应于selector标签，表示的是一drawable的集合，每个drawable都对应一种状态，语法如下：

```
    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android"
        android:constantSize=["true" | "false"]
        android:dither=["true" | "false"]
        android:variablePadding=["true" | "false"] >
        <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:state_pressed=["true" | "false"]
            android:state_focused=["true" | "false"]
            android:state_hovered=["true" | "false"]
            android:state_selected=["true" | "false"]
            android:state_checkable=["true" | "false"]
            android:state_checked=["true" | "false"]
            android:state_enabled=["true" | "false"]
            android:state_activated=["true" | "false"]
            android:state_window_focused=["true" | "false"] />
    </selector>
```

- constantSize StateListrawable的固有大小是否随着其状态的改变而改变，因为状态改变会切换到不同的drawable，不同的drawable可能有不同的大小，如果为true，那么StateListrawable的固有大小与内部所有drawable固有大小最大的那个相等，默认是false
- dither 是否开启抖动效果，默认true
- variablePadding StateListrawable的padding是否随着状态d变化而变化，true表示是，false表示不改变，其值为所有内部drawable中padding最多的那个默认false

### LevelListDrawable

对应level-list标签，同样表示一个drawable集合，内部每个drawable都对应一个level，根据不同的等级，不同的等级显示不同drawable

    <?xml version="1.0" encoding="utf-8"?>
    <level-list
        xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@drawable/drawable_resource"
            android:maxLevel="integer"
            android:minLevel="integer" />
    </level-list>

LevelListDrawable的等级范围是0-10000，默认是0


### TransitionDrawable

对应于transition标签，用于实现两个drawable之间的淡入淡出效果，语法如下：

    <?xml version="1.0" encoding="utf-8"?>
    <transition
    xmlns:android="http://schemas.android.com/apk/res/android" >
        <item
            android:drawable="@[package:]drawable/drawable_resource"
            android:id="@[+][package:]id/resource_name"
            android:top="dimension"
            android:right="dimension"
            android:bottom="dimension"
            android:left="dimension" />
    </transition>

通过代码调用：

     TransitionDrawable background = (TransitionDrawable) transitoinIv.getBackground();
     background.startTransition(1000);


### InsetDrawable
InsetDrawable 对应于inset标签，可以将其他drawable内嵌到自己当中，并可以在四周流出一定的间距，当一个view希望自己的背景比自己的实际区域小的时候，可以采用insetDrawable实现，同时也可以用LayerDrawable实现，语法

    <?xml version="1.0" encoding="utf-8"?>
    <inset
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:insetTop="dimension"
        android:insetRight="dimension"
        android:insetBottom="dimension"
        android:insetLeft="dimension" />

insetTop等表示个方向上的内凹大小

### ScaleDrawable

ScaleDrawable对应于scale标签，它可以根据自己等级，将指定的drawable缩放到一定的大小比例，语法如下：

    <?xml version="1.0" encoding="utf-8"?>
    <scale
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:drawable="@drawable/drawable_resource"
        android:scaleGravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                              "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                              "center" | "fill" | "clip_vertical" | "clip_horizontal"]
        android:scaleHeight="percentage"
        android:scaleWidth="percentage" />


- scaleGravity 表示对其方向
- scaleHeight和scaleWidth 分别表示对drawable的缩放比例

ScaleDrawable内部通过等级0表示缩放比例， 表示ScaleDrawable不可见，10000表示不缩放，在xml中缩放比例越大，那么缩放的drawable看起来就越小，ScaleDrawable更倾向于将drawable缩小到一个特定的比例

### ClipDrawable

对应于clip标签，可以根据自己的等级level来裁剪一个drawable，裁剪方向通过clipOrientation和gravity这两个属性共同决定，语法如下：

    <?xml version="1.0" encoding="utf-8"?>
    <bitmap
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:src="@[package:]drawable/drawable_resource"
        android:antialias=["true" | "false"]
        android:dither=["true" | "false"]
        android:filter=["true" | "false"]
        android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                          "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                          "center" | "fill" | "clip_vertical" | "clip_horizontal"]
        android:mipMap=["true" | "false"]
        android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] />


| Value | Description |
|-------|-----------|
| `top` | 内部drawable放在容器顶部，不改变大小，如果竖直裁剪，从底部开始 |
| `bottom` | 内部drawable放在容器底部，不改变大小，如果竖直裁剪，从顶部开始  |
| `left` | 内部drawable放在容器左边，不改变大小，如果竖直裁剪，从右边开始  |
| `right` | 内部drawable放在容器右边，不改变大小，如果竖直裁剪，从左边开始 |
| `center_vertical` |  内部drawable垂直方向填充，，如果为垂直裁剪，那么当clipDrawable的level为0时(0表示完全裁剪，即不可见)，才能有裁剪行为  |
| `fill_vertical` | Grow the vertical size of the object if needed so it completely fills its container. |
| `center_horizontal` | 内部drawable水平居中，不改变大小，如果水平裁剪，左右同时开始  |
| `fill_horizontal` | 内部drawable水平方向填充，，如果为水平裁剪，那么当clipDrawable的level为0时(0表示完全裁剪，即不可见)，才能有裁剪行为  |
| `center` | 内部drawable居中，不改变大小，如果竖直裁剪，上下同时开始，如果水平裁剪，左右同时开始 |
| `fill` | 内部drawable填充，当clipDrawable的level为0时(0表示完全裁剪，即不可见)，才能有裁剪行为  |
| `clip_vertical` |附加选项，表示垂直方向的裁剪，较少使用 |
| `clip_horizontal` | 附加选项，表示水平方向的裁剪，较少使用 |


### RotateDrawable

表示旋转一个bitmap

    <rotate xmlns:android="http://schemas.android.com/apk/res/android"
            android:drawable="@drawable/avatar_1"
            android:fromDegrees="90"
            android:pivotX="50%"
            android:pivotY="50%"
            android:visible="true">
    </rotate>


### AnimationDrawable

AnimationDrawable用于实现帧动画，不推荐使用

### AnimatedRotateDrawable

对应`animated_rotate`标签，用于实现一个旋转动画


---
## 3 WebP

WebP是Google提供的有损压缩（如JPEG）以及透明度（如PNG）的图像文件格式，但可以提供比JPEG或PNG更好的压缩。 Android 4.0（API级别14）及更高版本支持有损WebP图像，Android 4.3（API级别18）及更高版本支持无损和透明的WebP图像。

---
## 引用

- [官方文档](http://developer.android.com/intl/zh-cn/guide/topics/resources/drawable-resource.html)
- 《安卓开发艺术探索》
