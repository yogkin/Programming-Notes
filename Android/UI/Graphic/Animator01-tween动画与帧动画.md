# Android 动画

---
## 1 Android动画——帧动画

View动画和帧动画是安卓一开始就支持的动画，**帧动画**是连续播放一些列图片来形成的动画，使用起来非常简单，但是非常浪费内存的，现在应该被遗弃，所以不做过多介绍。相关类就是**AnimationDrawable**

---
## 2 View动画

Creates an animation by performing a series of transformations on a single image with an Animation：通过对一个单一的图像进行了一系列变换形成动画

View动画包括：

- 平移(translate)
- 旋转(rotate)
- 缩放(sclae)
- 透明度(alpha)

### 2.1 View动画在xml中的定义

View动画支持xml定义，语法如下：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android"
        android:interpolator="@[package:]anim/interpolator_resource"
        android:shareInterpolator=["true" | "false"] >

        <alpha
            android:fromAlpha="float" 
            android:toAlpha="float" /> 

        <scale
            android:fromXScale="float"
            android:toXScale="float"
            android:fromYScale="float"
            android:toYScale="float"
            android:pivotX="float"
            android:pivotY="float" />

        <translate
            android:fromXDelta="float"
            android:toXDelta="float"
            android:fromYDelta="float"
            android:toYDelta="float" />

        <rotate
            android:fromDegrees="float"
            android:toDegrees="float"
            android:pivotX="float"
            android:pivotY="float" />
        <set>
            ...
        </set>
    </set>
```

**set** 可以包含若干个动画，并且其内部可以包含其他动画集合，另外它还有两个属性：

- interpolator 表示动画所使用的插值器，插值器影响动画的播放速率，默认为`android:interpolator="@android:anim/accelerate_interpolator"`，加速减速插值器
- shareInterpolator 表示集合中的动画是否和集合共享一个插值器，如果不共享单个动画使用自己执行的或者默认的插值器

但是set不支持include其他动画，这也算是一个缺陷吧。


其他四种动画也可以单独在xml中定义，语法也类似，如下面示例：

```xml
    //透明度范围（0-1）0表示完全透明，1表示完全不透明
    <alpha xmlns:android="http://schemas.android.com/apk/res/android"
        android:fromAlpha="0" android:toAlpha="1"
        android:duration="3000"
        android:repeatCount="3"
        android:repeatMode="reverse">
    </alpha>

    <rotate xmlns:android="http://schemas.android.com/apk/res/android"
        android:fromDegrees="0" 
        android:toDegrees="360"
        android:pivotX="50%"  //缩放轴的x坐标，推荐使用%
        android:pivotY="50%"
        android:repeatCount="3" 
        android:repeatMode="reverse"
        android:duration="3000">
    </rotate>
    
    <scale xmlns:android="http://schemas.android.com/apk/res/android"
        android:duration="3000"
        android:fromXScale="0.5"
        android:fromYScale="0.5"
        android:pivotX="0.5"
        android:pivotY="0.5"
        android:repeatCount="3"
        android:repeatMode="reverse"
        android:toXScale="1.0"
        android:toYScale="1.0" >
    </scale>
    
    <translate xmlns:android="http://schemas.android.com/apk/res/android"
        android:duration="3000"
        android:fromXDelta="-30%p" //-30%p的p表示相对于父控件
        android:fromYDelta="-30%p"
        android:repeatCount="3"
        android:repeatMode="restart"
        android:toXDelta="30%p"
        android:zAdjustment="1"
     android:interpolator="@android:anim/accelerate_interpolator"
        android:toYDelta="30%p" >
    </translate>
```

几个公共属性的说明:

- repeatCount 重复次数 -1表示无限循环
- repeatMode 重复模式，restart表示重新开始，reverse反复执行
- duration 表示动画时间
- fillAfter 表示动画结束后，view是否保存动画后的位置
- interpolator 指定插值器
- zAdjustment 用于设置动画的叠放次序

另外一些属性加上p表示相对于父控件，如：android:pivotX="50%p"


### 2.2 使用代码实现View动画

View动画也支持代码书写，如下面示例：

```java
    public void alpha(View v) {
            // 其实透明度0 最后的透明度1
            AlphaAnimation aa = new AlphaAnimation(0, 1.0f);
            aa.setDuration(3000);// 设置动画的持续时间
            aa.setRepeatCount(3);
            aa.setRepeatMode(Animation.REVERSE);// 重复模式 反着来 还有restart 从头开始
            iv.startAnimation(aa);
        }
    
        public void scale(View v) {
            ScaleAnimation sa = new ScaleAnimation(0.1f, 2.0f, 0.1f, 2.0f,
                    Animation.RELATIVE_TO_SELF, 0.5f,
                    Animation.RELATIVE_TO_SELF, 0.5f);
            sa.setDuration(3000);
            sa.setRepeatCount(3);
            sa.setRepeatMode(Animation.REVERSE);
            iv.startAnimation(sa);
        }
    
        public void translate(View v) {
            TranslateAnimation ta = new TranslateAnimation(
                    Animation.RELATIVE_TO_PARENT, -0.5f,
                    Animation.RELATIVE_TO_PARENT, 0.5f,
                    Animation.RELATIVE_TO_PARENT, -0.5f,
                    Animation.RELATIVE_TO_PARENT, 0.5f);
            ta.setDuration(3000);
            ta.setRepeatCount(3);
            // ta.setFillAfter(true);//保持播放后的效果
            ta.setRepeatMode(Animation.REVERSE);
            iv.startAnimation(ta);
        }
    
        public void rotate(View v) {
            // 旋转前的角度 旋转后的角度 相对于谁旋转 x轴心 相对于谁旋转 y轴心
            RotateAnimation ra = new RotateAnimation(0, 360,
                    Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,
                    0.5f);
            ra.setDuration(3000);
            ra.setRepeatCount(3);
            // ta.setFillAfter(true);//保持播放后的效果
            ra.setRepeatMode(Animation.RESTART);
            iv.startAnimation(ra);
        }
    
        public void set(View v) {
            // AnimationSet表示所有子view是否公用一个插值器
            AnimationSet as = new AnimationSet(false);
            ScaleAnimation sa = new ScaleAnimation(0.5f, 1.0f, 0.5f, 1.0f,
                    Animation.RELATIVE_TO_SELF, 0f, Animation.RELATIVE_TO_SELF,0f);
            sa.setDuration(3000);
            sa.setRepeatCount(3);
            sa.setRepeatMode(Animation.REVERSE);
            TranslateAnimation ta = new TranslateAnimation(
                    Animation.RELATIVE_TO_PARENT, -0.3f,
                    Animation.RELATIVE_TO_PARENT, 0.3f,
                    Animation.RELATIVE_TO_PARENT, -0.3f,
                    Animation.RELATIVE_TO_PARENT, 0.3f);
            ta.setDuration(3000);
            ta.setRepeatCount(3);
            ta.setRepeatMode(Animation.REVERSE);
    
            RotateAnimation ra = new RotateAnimation(0, 360,
                    Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF,
                    0.5f);
            ra.setDuration(3000);
            ra.setRepeatCount(3);
            ra.setRepeatMode(Animation.RESTART);
            as.addAnimation(ra);
            as.addAnimation(ta);
            as.addAnimation(sa);
            iv.startAnimation(as);
        }
```

需要注意的是，如果一个set动画表现出来的行为与期望的行为不一样，试着调整子动画的添加顺序。


另外动画还支持动画的播放监听，如下：

```
    new AnimationListener() {
                @Override
                public void onAnimationStart(Animation animation) {
                    // 动画开始
                    
                }
                @Override
                public void onAnimationRepeat(Animation animation) {
                    // 动画重复
                    
                }
                @Override
                public void onAnimationEnd(Animation animation) {
                    //动画结束
                }
            }
```

其他方法说明：

```
    RotateAnimation ra = ....;
    ra.setStartOffset(startOffset) //表示动画在偏移多少秒后执行
    ra.setZAdjustment(1);//设置叠放次序
    ra.cancel();// 取消动画
```

setZAdjustment如何使用，看这个链接：[Android: how to bring Animation to front: bringToFront and zAdjustment/Z-ordering](http://stackoverflow.com/questions/10427298/android-how-to-bring-animation-to-front-bringtofront-and-zadjustment-z-orderin)

    
    This should be possible if you extend LinearLayout and override the following method:
            getChildDrawingOrder (int childCount, int i)
    To make sure layout uses your function you need to call      
            setChildrenDrawingOrderEnabled(true)
    See ViewGroup javadoc

关于getChildDrawingOrder可以看这篇文章:[改变GridView或ListView刷新ITEM的次序-Android](http://watt201211.blog.163.com/blog/static/2234870342013779484841/)


或者下面方法:

```java
    //For your z reordering will apply you gonna need to request a new layout on the view, but try doing it before starting your animation:

        AnimationSet set = new AnimationSet(true);
        // init animation ... 
        scaledView.bringToFront(); 
        scaledView.requestLayout(); 
        myFrameLayout.startAnimation(set);
```

### 2.3 插值器

插值器：在一个连续变换的动作中，把每个时刻对于的值都转变为另一个值，对于动画就是**动画的时间完成度和动画完成度**。

官方提供了以下几种插值器

| Interpolator class | Resource ID |
| ------------ | ------------ |
| 先加后减(默认)`[AccelerateDecelerateInterpolator]` | `@android:anim/accelerate_decelerate_interpolator` |
| 加速`[AccelerateInterpolator]` | `@android:anim/accelerate_interpolator` |
| 先反后加`[AnticipateInterpolator]` | `@android:anim/anticipate_interpolator` |
| 先反后加再反`[AnticipateOvershootInterpolator]` | `@android:anim/anticipate_overshoot_interpolator` |
| 自由落体反弹`[BounceInterpolator]` | `@android:anim/bounce_interpolator` |
| 周期`[CycleInterpolator]` | `@android:anim/cycle_interpolator` |
| 减速`[DecelerateInterpolator]` | `@android:anim/decelerate_interpolator` |
| 线性`[LinearInterpolator]` | `@android:anim/linear_interpolator` |
| 先加后反`[OvershootInterpolator]` | `@android:anim/overshoot_interpolator` |

**自定义插值器[官方文档](http://developer.android.com/intl/zh-cn/guide/topics/resources/animation-resource.html#obj-animator-element)**:

如果你不满意这个平台提供的内插器，您可以创建与修改的属性自定义的内插的资源。例如，你可以调整的加速，为AnticipateInterpolator率,或调整周期为CycleInterpolator的数目。为了做到这一点，你需要在一个XML文件中创建自己的内插器的资源。


在res/anim中定义一个资源文件：
如：`res/anim/_filename_.xml`

语法为：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <InterpolatorName xmlns:android="http://schemas.android.com/apk/res/android"
        android:attribute_name="value"/>
```

各个插值器的属性为：

```
    <accelerateDecelerateInterpolator>：变化率开始和结束，但慢慢地加速中间穿过。 没有属性。

    <accelerateInterpolator>变化率开始慢慢，然后加速。属性：android:factor，Float 加速度（默认为1）

    <anticipateInterpolator>这种变化开始向后甩，然后向前。属性：android:tension，Float张力的量应用（默认为2）

    <anticipateOvershootInterpolator>变化开始向后，甩向前和过冲的目标值，然后稳定在最终值。属性:android:tension,张力的量应用（默认为2）;android:extraTension，Float 由量乘以张力（默认值为1.5）。

    <bounceInterpolator>变化反弹在末端。无属性

    <cycleInterpolator>重复的动画周期的指定数目。变化率遵循正弦模式。属性：android:cycles：Int。的周期数（默认值为1）

    <decelerateInterpolator>变化率迅速开出，然后减速。属性：android:factor，Float减速度（默认为1）

    <linearInterpolator>线性，无属性

    <overshootInterpolator>变化甩向前和过冲的最后一个值，然后回来。属性：android:tension，Float，张力的量应用（默认为2）
```

使用示例：

```xml
    <!--XML file saved at res/anim/my_overshoot_interpolator.xml:-->
    <?xml version="1.0" encoding="utf-8"?>
    <overshootInterpolator xmlns:android="http://schemas.android.com/apk/res/android"
        android:tension="7.0"
        />

    This animation XML will apply the interpolator:
    <scale xmlns:android="http://schemas.android.com/apk/res/android"
        android:interpolator="@anim/my_overshoot_interpolator"
        android:fromXScale="1.0"
        android:toXScale="3.0"
        android:fromYScale="1.0"
        android:toYScale="3.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:duration="700" />
```

---
## 参考
- [官方文档](http://developer.android.com/intl/zh-cn/guide/topics/resources/animation-resource.html#obj-animator-element)















