# View动画原理解析

---
## 1 View动画原理解析

参考文章:[Android动画原理分析](http://www.cnblogs.com/kross/p/4087780.html)

在View的draw方法我们可以找到关于Animation的踪影：

```java
            final Animation a = getAnimation();
            if (a != null) {
                more = applyLegacyAnimation(parent, drawingTime, a, scalingRequired);
                concatMatrix = a.willChangeTransformationMatrix();
                if (concatMatrix) {
                    mPrivateFlags3 |= PFLAG3_VIEW_IS_ANIMATING_TRANSFORM;
                }
                transformToApply = parent.getChildTransformation();
            }
```

其实View动画就是不断的改变View的绘制区域来实现动画，这也就是为什么当一个View执行动画到了另外一个区域，但是它的点击事件还是在原来的地方的原因，因为它的位置并没有真的改变，改变的只是它的绘制区域。

---
## 2 属性动画源码分析

- [动画系列 - PropertyAnim 详解](http://www.lightskystreet.com/2014/12/04/propertyview-anim-analysis/)
- 《安卓开发艺术探索》

---
## 3 使用动画的注意事项

- OOM：避免使用帧动画
- 内存泄漏：属性动画可以无限循环，这类动画需要在Activity退出时及时的停止掉，否则照成内存泄漏，而View动画不会
- 兼容性问题，由于现在应用大都只支持4.0以上的手机，所以此问题也不算问题了
- View动画的问题：View动画只是对View的影像做了移动，并不是真的改变了View的状态，在调用View的setVisibility时，记得先调用View的clearAnimation方法
- 动画中不要使用px