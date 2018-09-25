### 学习目标

- 熟悉tween动画和帧动画
- 自定义动画
- 熟悉属性动画
- 扩展插值器
- 学习**API18**`View的Overlay`
- 学习**API19**Scenes和Transitions
- 掌握Android动画的原理
- Activity和Fragmet转场动画
- CircularReveal
- 矢量动画

## Activity动画库

- [方便的添加Activity的侧滑退出](https://github.com/r0adkll/Slidr )
- [高仿微信手势滑动返回](https://github.com/hanhailong/SwipeBackSample)
- [Activit移的MD风格-Animations](https://github.com/lgvalle/Material-Animations)
- [Plaid的动画研究Fab and Dialog Morphing Animation](http://hujiaweibujidao.github.io/blog/2015/12/13/Fab-and-Dialog-Morphing-Animation/)
- [MaterialLogin--揭露动画](https://github.com/shem8/MaterialLogin)
- [结合motion和Transition实现共享元素的酷炫动画](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-43/%E7%BB%93%E5%90%88motion%E5%92%8CTransition%E5%AE%9E%E7%8E%B0%E5%85%B1%E4%BA%AB%E5%85%83%E7%B4%A0%E7%9A%84%E9%85%B7%E7%82%AB%E5%8A%A8%E7%94%BB.md)
- [ImageTransition-Android L之前如何实现Image的过度动画](https://github.com/danylovolokh/ImageTransition)
- [MaterialDesignSample-使用Transaction导致状态栏无法着色参考](https://github.com/rejasupotaro/MaterialDesignSample)
- [DraggablePanel-Activity播放通拖动，Youtube](https://github.com/pedrovgs/DraggablePanel)
- [PreLollipopTransition](https://github.com/takahirom/PreLollipopTransition)
- [一个有趣的网站，实时预览interpolator效果](http://inloop.github.io/interpolator/)，[使用说明](http://hanks.xyz/2016/03/08/material-interpolator/)
- [EasingInterpolator-各种插值器](https://github.com/MasayukiSuda/EasingInterpolator)
- [一个方便的开源动画库-ViewAnimator](https://github.com/florent37/ViewAnimator)
- [FPSAnimator是Android TextureView和SurfaceView非常简单的动画库。](https://github.com/MasayukiSuda/FPSAnimator)
- [android-player](https://github.com/geftimov/android-player)
- [小红书欢迎引导界面第一版(视差动画版)](https://github.com/w446108264/XhsParallaxWelcome)
- [MD风格搜索动画](https://github.com/NiaNingXue/SearchAnimation)
- [Material-Animations](https://github.com/lgvalle/Material-Animations)

## 属性动画

- [浅析Android动画（一），View动画高级实例探究](http://www.cnblogs.com/wondertwo/p/5295976.html)
- [浅析Android动画（二），属性动画高级实例探究](http://www.cnblogs.com/wondertwo/p/5312482.html)
- [浅析Android动画（三），自定义Interpolator与TypeEvaluator](http://www.cnblogs.com/wondertwo/p/5327586.html)
- [属性动画高级用法](http://www.sunnyang.com/401.html)


## 系列教程

- [当数学遇上动画：讲述ValueAnimator、TypeEvaluator和TimeInterpolator之间的恩恩怨怨](https://gold.xitu.io/entry/575063b4df0eea004f57be79)
- [yava](https://github.com/hujiaweibujidao/yava)
- [探索安卓中有意义的动画](https://segmentfault.com/a/1190000004182537)

## ShareElement

- [Activity和Fragment Transition介绍](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0113/2310.html)
- [Content Transitions In-Depth (part 2)](http://www.androiddesignpatterns.com/2014/12/activity-fragment-content-transitions-in-depth-part2.html)
- [深入理解共享元素变换（Shared Element Transition）-上](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0201/2394.html)

## Scenes和Transitions

- [Android Transition Framework 详解---超炫的动画框架](https://mp.weixin.qq.com/s?__biz=MzIxNzU1Nzk3OQ==&mid=2247485549&idx=1&sn=b4dff40f9ed24d94fb95d9e27336ffda&chksm=97f6b6d9a0813fcf7e342ddfe455a67088b0766de9ffd760ad81fe9c5b7431e6eb3670332f46&mpshare=1&scene=1&srcid=0912nkJiOX8hE1AEqXM3zhON#rd)
- [Android 4.4.2引入的超炫动画库](https://mp.weixin.qq.com/s?__biz=MzIwMzYwMTk1NA==&mid=2247487125&idx=1&sn=b3cdac76e37bc4bb234a53cd5f03b527&chksm=96cdafd8a1ba26ceae06a75ace639f9dcd6bd1bec329536ceea6582c6e356a0f9a464f4ddb44&mpshare=1&scene=1&srcid=0925ALdkNrlW2K2Wsz5pdf0d#rd)
- [官方文档](http://developer.android.com/intl/zh-cn/training/transitions/index.html)
- [Transitions-Everywhere 兼容库](https://github.com/andkulikov/Transitions-Everywhere)
- [Scenes和Transitions学习](http://www.jianshu.com/p/5e587a169e70)
- [官方示例](https://github.com/googlesamples/android-BasicTransition/)

## 特殊动效

 - [depth](https://github.com/yongjhih/depth)
 - [depth解析](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0429/4200.html)

## ViewOverlay

[ViewOverlay](http://developer.android.com/reference/android/view/ViewOverlay.html)是4.3以后（api 18+）新增的一个类，它是view的最上面的一个透明的层，我们可以在这个层之上添加内容而不会影响到整个布局结构。这个层和我们的界面大小相同，可以理解成一个浮动在界面表面的二维空间。