# Activity转场动画

Activity都有自己的默认切换效果，这个切换效果我们也可以自定指定，在android5.0之后添加了更多的Activity切换效果，接下来介绍。

## 1 传统Activity的切换动画

传统Activity的切换动画通过`overridePendingTransition(enterAnim,exitAnim)`来指定，**这个方法必须在startActivity和finish之后调用才能生效**：

*   enterAnim 指定Activity被打开时所需要的动画资源id
*   exitAnim 指定Activity被暂停时需要的动画资源id

下面是简单的使用方法：

```java
     //启动一个Activity时：
        Intent intent = null;
        startActivity(intent);
        overridePendingTransition(android.R.anim.fade_in,android.R.anim.fade_out);

     //退出一个Activity时：
       @Override
       public void finish() {
           super.finish();
           overridePendingTransition(android.R.anim.fade_in, android.R.anim.fade_out);
       }
```

###  在style中指定

其实Activity动画特效还可以通过在style中指定`android:theme`实现 ，使用这种方法时我们还需要定义一个 style 配置文件来配置动画，并且通过 `android:theme` 属性为 application或activity 指定动画配置即可。style.xml 内容如下：

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <style name="AnimationWindow">
            <item name="android:windowAnimationStyle">@style/test</item>
        </style>
        <style name="test" parent="@android:style/Animation.Activity">
            <item name="android:activityOpenEnterAnimation">@anim/activity_in_right</item>
            <item name="android:activityOpenExitAnimation">@anim/activity_out_left</item>
            <item name="android:activityCloseEnterAnimation">@anim/activity_in_right</item>
            <item name="android:activityCloseExitAnimation">@anim/activity_out_left</item>
        </style>
    </resources>
```

为整个app 的activity 配置动画
```xml
      <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name"
             android:theme="@style/AnimationWindow">
```
为单独的 activity 配置动画
```xml
    <activity android:name="org.xiaoyunduo.MyDemos"
                android:theme="@style/AnimationWindow">
```

---
## 2 5.0+转场动画

Androd5.0开始，android为Activity的转场添加更加丰富的效果，提供了三种transition类型。

*   **进入**转换将决定操作行为中视图如何进入场景。
*   **退出**转换将决定操作行为中应用行为的显示视图如何退出场景。
*   **共享元素**一个共享元素过渡动画决定两个activity之间的过渡。

其中进入和退出包括：

*   分解 - 从场景中心移入或移出视图。
*   滑动 - 从场景边缘移入或移出视图。
*   淡入淡出 - 通过调整透明度在场景中增添或移除视图。

Android 5.0（API 级别 21）也支持这些共享元素转换：

*   changeBounds - 为目标视图的布局边界的变化添加动画。
*   changeClipBounds - 为目标视图的裁剪边界的变化添加动画。
*   changeTransform - 为目标视图的缩放与旋转变化添加动画。
*   changeImageTransform - 为目标图像的大小与缩放变化添加动画。


###  进入和退出动画

在Activity的theme中添加以下属性(只支持api21+，可以在values-21中设置)`<item name="android:windowContentTransitions">true</item>`，或者直接用代码设置(被打开的那个Activity中设置)`getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);`，然后是打开一个Activity：


```java
            Intent intent = new Intent(this, Start1Activity.class);
            intent.putExtra("FLAG", flag);//加标记
            startActivity(this,intent);
```

然后在被打开的Activity中设置

```java
 if (Build.VERSION.SDK_INT >= 21) {
        getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        int flag = getIntent().getIntExtra("FLAG", -1);
        if (flag == 1) {
            getWindow().setEnterTransition(new Explode());
            getWindow().setExitTransition(new Explode());

        } else if (flag == 2) {
            getWindow().setEnterTransition(new Slide());
            getWindow().setExitTransition(new Slide());

        } else if (flag == 3) {
            getWindow().setEnterTransition(new Fade());
            getWindow().setExitTransition(new Fade());
        }
}
```


### 共享元素

使用共享元素同样需要设置`Window.FEATURE_CONTENT_TRANSITIONS`.

由于需要适配低版本的Android系统，需要使用**ActivityOptionsCompat**类。我们可以通过这个类来启动activity和添加动画。但是所有的动画在给2.x是没有用的，大部分动画也对4.x也不兼容。

ActivityOptionsCompat提供的方法有：

#### 1，ActivityOptionsCompat.makeCustomAnimation(context, enterResId, exitResId)

简单做一个定制的动画，这个参数很简单，传入一个进入的动画的id，和移除动画的id即可，与overridePendingTransition一样


#### 2，ActivityOptionsCompat.makeScaleUpAnimation(source, startX, startY, startWidth, startHeight)

这个在api16及以上有用，可以实现新的Activity从某个固定的坐标以某个大小扩大至全屏

#### 3，ActivityOptionsCompat.makeSceneTransitionAnimation(activity, sharedElement, sharedElementName)

在Api21以上有效，主要用于使用分享元素效果来启动Activity，它还有另一个重载方法`ActivityOptionsCompat.makeSceneTransitionAnimation((Activity arg0, Pair<view string="">… arg1)`，用于分享多个元素


在低于5.0版本上实现此效果可以参考：[实现类似于QQ空间相册的点击图片放大，再点后缩小回原来位置](http://www.cnblogs.com/tianzhijiexian/p/4095756.html)，其原理是在同一个Activity里利用动画来仿造一个Activity的启动。

#### 4，ActivityOptionsCompat.makeThumbnailScaleUpAnimation(source, thumbnail, startX, startY)

这个方法可以用于4.x上，是将一个小块的Bitmpat进行拉伸的动画

**示例**使用下面代码设置共享元素打开Activity：

```java
        //单个共享元素
        Intent intent = new Intent(this, ShareContentActivity.class);
        Bundle share = ActivityOptionsCompat.makeSceneTransitionAnimation(this,
                view, "share"
        ).toBundle();
        ActivityCompat.startActivity(this, intent, share
        );

       //多个共享元素
        Intent intent = new Intent(this, ShareContentActivity.class);
        Bundle share = ActivityOptionsCompat.makeSceneTransitionAnimation(this,
                Pair.create(view, "share1"), Pair.create((View) mBtn2, "share2"), Pair.create((View) mBtn3, "share3")
        ).toBundle();
        ActivityCompat.startActivity(this, intent, share );

```

在被打开的Activity中同样需要指定共享元素的名称，而且必须与打开它的Activity传递的名称保持一致：

指定shareName的方式有两种：

xml中指定：

```xml
 <ImageView
        android:id="@+id/act_s_iv"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_margin="20dp"
        android:background="@color/c_acidgreen"
        android:transitionName="share1"/>

```

代码中指定：

```
    if (Build.VERSION.SDK_INT >= 21) {
           findViewById(R.id.act_s_iv).setTransitionName("share");
    }
```

---
## 参考

- [定义定制动画](https://developer.android.com/training/material/animations.html?hl=zh-cn)
- [实现类似于QQ空间相册的点击图片放大，再点后缩小回原来位置](http://www.cnblogs.com/tianzhijiexian/p/4095756.html)
- [ImageTransition-兼容](https://github.com/danylovolokh/ImageTransition)