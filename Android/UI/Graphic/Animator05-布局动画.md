
---
# 1 LayoutAnimation

LayoutAnimation用于ViewGroup，为GroupView指定一个动画，这样当它的子元素出场时都会具有这样的动画效果。经常用于ListView上。

参考文章：[Android LayoutAnimation 使用及扩展](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0619/3090.html)

### LayoutAnimation使用

#### 1 使用animateLayoutChanges属性

```
        <ListView
            android:animateLayoutChanges="true"
            android:id="@+id/fragment_lv"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
```

animateLayoutChanges通过上面熟悉设置，当ViewGroup添加View时，子view会呈现逐渐实现的效果，不过这个效果是安卓默认的，不同被修改。

#### 2 xml设置animation属性

```xml
    <layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
                     android:animation="@anim/la_scale"
                     android:animationOrder="normal"
                     android:delay="0.5">
    
    </layoutAnimation>
    
    <scale xmlns:android="http://schemas.android.com/apk/res/android"
           android:duration="1000"
           android:fromXScale="10%"
           android:fromYScale="10%"
           android:pivotX="50%"
           android:pivotY="50%"
           android:toXScale="100%"
           android:toYScale="100%"
        >
    </scale>

    //使用
     <ListView
            android:layoutAnimation="@anim/la_demo"
            android:id="@+id/fragment_lv"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
```

在res/anim中新建layoutAnimation标签，其属性有：

- animation 指定动画
- animationOrder 表示子view播放动画的顺序，有顺序，随机，反序可选。


#### 3 使用代码设置

```java
            mListView = (ListView) view.findViewById(R.id.fragment_lv);
            LayoutAnimationController lac = new LayoutAnimationController(AnimationUtils.loadAnimation(getActivity() ,R.anim.la_scale));
            lac.setDelay(0.5f);
            mListView.setLayoutAnimation(lac);
```

---
# 2 LayoutTransition

- 参考：[容器布局动画 LayoutTransition](http://www.cnblogs.com/mengdd/p/3305973.html)
- 参考: [Android属性动画Property Animation系列三之LayoutTransition（布局容器动画）](http://blog.csdn.net/feiduclear_up/article/details/45919613)

通过LayoutTransition类为布局的变化添加动画。当一个ViewGroup中添加或者移除某一个item，或者调用了View的setVisibility方法，使得View 变得VISIBLE或者GONE的时候，在ViewGroup内部的View可以完成出现或者消失的动画。当你添加或者移除View的时候，那些剩余的View也可以通过动画的方式移动到自己的新位置。你可以通过setAnimator()方法并传递一个Animator对象,在LayoutTransition内部定义以下动画。以下是几种事件类型的常量：

- APPEARING　　　　　　　　为那些添加到父元素中的元素应用动画
- CHANGE_APPEARING　　　 为那些由于父元素添加了新的item而受影响的item应用动画
- DISAPPEARING　　　　　　 为那些从父布局中消失的item应用动画
- CHANGE_DISAPPEARING　  为那些由于某个item从父元素中消失而受影响的item应用动画

可以为这四种事件定义自己的交互动画。

LayoutTransition使用：

```java
    transition.setAnimator(LayoutTransition.DISAPPEARING, customDisappearingAnim);
    /此函数设置动画延迟时间，参数分别为类型与时间。
    mTransitioner.setStagger(LayoutTransition.CHANGE_APPEARING, 30);
```

代码示例：

1，创建一个LayoutTransaction
```java
        mLayoutTransition = new LayoutTransition();
        // 生成自定义动画
        private void setupCustomAnimations() {
            // 动画：CHANGE_APPEARING
            // Changing while Adding
            PropertyValuesHolder pvhLeft = PropertyValuesHolder.ofInt("left", 0, 1);
            PropertyValuesHolder pvhTop = PropertyValuesHolder.ofInt("top", 0, 1);
            PropertyValuesHolder pvhRight = PropertyValuesHolder.ofInt("right", 0,
                    1);
            PropertyValuesHolder pvhBottom = PropertyValuesHolder.ofInt("bottom",
                    0, 1);
            PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofFloat("scaleX",
                    1f, 0f, 1f);
            PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofFloat("scaleY",
                    1f, 0f, 1f);
    
            final ObjectAnimator changeIn = ObjectAnimator.ofPropertyValuesHolder(
                    this, pvhLeft, pvhTop, pvhRight, pvhBottom, pvhScaleX,
                    pvhScaleY).setDuration(
                    mLayoutTransition.getDuration(LayoutTransition.CHANGE_APPEARING));
            mLayoutTransition.setAnimator(LayoutTransition.CHANGE_APPEARING, changeIn);
            changeIn.addListener(new AnimatorListenerAdapter() {
                public void onAnimationEnd(Animator anim) {
                    View view = (View) ((ObjectAnimator) anim).getTarget();
                    view.setScaleX(1f);
                    view.setScaleY(1f);
                }
            });
    
            // 动画：CHANGE_DISAPPEARING
            // Changing while Removing
            Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
            Keyframe kf1 = Keyframe.ofFloat(.9999f, 360f);
            Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
            PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe(
                    "rotation", kf0, kf1, kf2);
            final ObjectAnimator changeOut = ObjectAnimator
                    .ofPropertyValuesHolder(this, pvhLeft, pvhTop, pvhRight,
                            pvhBottom, pvhRotation)
                    .setDuration(
                            mLayoutTransition
                                    .getDuration(LayoutTransition.CHANGE_DISAPPEARING));
            mLayoutTransition.setAnimator(LayoutTransition.CHANGE_DISAPPEARING,
                    changeOut);
            changeOut.addListener(new AnimatorListenerAdapter() {
                public void onAnimationEnd(Animator anim) {
                    View view = (View) ((ObjectAnimator) anim).getTarget();
                    view.setRotation(0f);
                }
            });
    
            // 动画：APPEARING
            // Adding
            ObjectAnimator animIn = ObjectAnimator.ofFloat(null, "rotationY", 90f,
                    0f).setDuration(
                    mLayoutTransition.getDuration(LayoutTransition.APPEARING));
            mLayoutTransition.setAnimator(LayoutTransition.APPEARING, animIn);
            animIn.addListener(new AnimatorListenerAdapter() {
                public void onAnimationEnd(Animator anim) {
                    View view = (View) ((ObjectAnimator) anim).getTarget();
                    view.setRotationY(0f);
                }
            });
    
            // 动画：DISAPPEARING
            // Removing
            ObjectAnimator animOut = ObjectAnimator.ofFloat(null, "rotationX", 0f,
                    90f).setDuration(
                    mLayoutTransition.getDuration(LayoutTransition.DISAPPEARING));
            mLayoutTransition.setAnimator(LayoutTransition.DISAPPEARING, animOut);
            animOut.addListener(new AnimatorListenerAdapter() {
                public void onAnimationEnd(Animator anim) {
                    View view = (View) ((ObjectAnimator) anim).getTarget();
                    view.setRotationX(0f);
                }
            });
        }
```
2，与ViewGroup关联：
```
    mLinearLayout.setLayoutTransition(mLayoutTransition);
```

3，设置添加一个View
```
    mLinearLayout.addView(textView);
```
