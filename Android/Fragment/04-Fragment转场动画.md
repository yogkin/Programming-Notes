# Fragment的转场动画

---
##  1 Fragment的转场动画使用

在Fragment的切换过程中添加动人的动画，可以让界面跳转更加自然，当然Fragment是支持动画的，而且方式还有很多，但是这些方式中很多是事有问题的。

---
### 1.1     setCustomAnimations(int enter, int exit)

两个参数 setCustomAnimations(int enter, int exit)方法，参数为动画的xml，支持View动画和属性动画。

此方法给Fragment的添加和切换设置动画，enter表示进入的动画，exit表示退出界面的动画。比如当一个Fragment被添加到容器中，它的View会执行enter动画，当它被replace时它的view执行exit动画。但是直接remove一个Fragment是不会执行exit动画的。

---
### 1.2 setCustomAnimations(int enter, int exit , int popEnter , int popExit)

这个方法多次两个参数，从参数名就可以看出，多了的两个参数用于给backStack操作设置动画。

popEnter表上栈顶的Fragment出栈后，重新回到栈顶的Fragment所执行的动画
popExit表示栈顶的Fragment的出栈动画。

---
### 1.3 setCustomAnimations的Bug

setCustomAnimations方法有一个很大的bug，就是在**内存重启**后所有设置的动画都将失效。

---
### 1.4 setTranseion和onCreateAnimation

setTranseion是FragmentTransaction的方法，而onCreateAnimation是Fragment的方法，一般两个方法需要配合使用。而且它们不会像`setCustomAnimations`一样，即使是**内存重启**也不会失效，因为他们是动态调用的。

先来看一下`setTranseion`方法：

```
        setTransition(@Transit int transit);
    
        @IntDef({TRANSIT_NONE, TRANSIT_FRAGMENT_OPEN, TRANSIT_FRAGMENT_CLOSE})
        @Retention(RetentionPolicy.SOURCE)
        private @interface Transit {}
```

很明显，要实现动画，我们只能传`TRANSIT_FRAGMENT_OPEN`和`TRANSIT_FRAGMENT_CLOSE`，他们分别表示进场和退场，首先使用FragmentTransaction设置Transeion：

```
     mFragmentManager.beginTransaction()
    .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
    .add(mLayoutId, mCurrentFragment, mCurrentFragment.getClass().getName());
    .addToBackStack(mCurrentFragment.getClass().getName());
    .commit();
```

单独使用`setTranseion`的话，FragmentManager会生成默认的动画，源码如下：

```
        //根据transit或动作拿animAttr
        public static int transitToStyleIndex(int transit, boolean enter) {
            int animAttr = -1;
            switch (transit) {
                case FragmentTransaction.TRANSIT_FRAGMENT_OPEN:
                    animAttr = enter ? ANIM_STYLE_OPEN_ENTER : ANIM_STYLE_OPEN_EXIT;
                    break;
                case FragmentTransaction.TRANSIT_FRAGMENT_CLOSE:
                    animAttr = enter ? ANIM_STYLE_CLOSE_ENTER : ANIM_STYLE_CLOSE_EXIT;
                    break;
                case FragmentTransaction.TRANSIT_FRAGMENT_FADE:
                    animAttr = enter ? ANIM_STYLE_FADE_ENTER : ANIM_STYLE_FADE_EXIT;
                    break;
            }
            return animAttr;
        }
        //根据animAttr生成动画
        switch (styleIndex) {
                case ANIM_STYLE_OPEN_ENTER:
                    return makeOpenCloseAnimation(mHost.getContext(), 1.125f, 1.0f, 0, 1);
                case ANIM_STYLE_OPEN_EXIT:
                    return makeOpenCloseAnimation(mHost.getContext(), 1.0f, .975f, 1, 0);
                case ANIM_STYLE_CLOSE_ENTER:
                    return makeOpenCloseAnimation(mHost.getContext(), .975f, 1.0f, 0, 1);
                case ANIM_STYLE_CLOSE_EXIT:
                    return makeOpenCloseAnimation(mHost.getContext(), 1.0f, 1.075f, 1, 0);
                case ANIM_STYLE_FADE_ENTER:
                    return makeFadeAnimation(mHost.getContext(), 0, 1);
                case ANIM_STYLE_FADE_EXIT:
                    return makeFadeAnimation(mHost.getContext(), 1, 0);
            }


setTranseion和和Fragment的onCreateAnimation配合使用：

        public Animation onCreateAnimation(int transit, boolean enter, int nextAnim) {
            if (transit == FragmentTransaction.TRANSIT_FRAGMENT_OPEN) {//表示是一个进入动作，比如add.show等
                if (enter) {//普通的进入的动作
                    return AnimationUtils.loadAnimation(getContext(), R.anim.anim_bottom_in);
                } else {//比如一个已经Fragmen被另一个replace，是一个进入动作，被replace的那个就是false
                    return AnimationUtils.loadAnimation(getContext(), R.anim.anim_out);
                }
            } else if (transit == FragmentTransaction.TRANSIT_FRAGMENT_CLOSE) {//表示一个退出动作，比如出栈，hide，detach等
                if (enter) {//之前被replace的重新进入到界面或者Fragment回到栈顶
                    return AnimationUtils.loadAnimation(getContext(), R.anim.anim_in);
                } else {//Fragment退出，出栈
                    return AnimationUtils.loadAnimation(getContext(), R.anim.anim_bottom_out);
                }
            }
            return null;
        }
```

transit对应FragmentTransaction设置的动作，`onCreateAnimation`在Fragment的每个操作动作中都会被回调，最好是配合FragmentTransaction的`setTranseion`方法使用，才能更加灵活的实现各种动画，不然`onCreateAnimation`方法的transit参数永远是0，而`nextAnim`与`setCustomAnimations`有关，而我一般不使用`setCustomAnimations`。

---
### 1.5 setTransitionStyle

setTransitionStyle也是可以实现动画的，但是Support包中的FragmentManager不支持，对比Support 包中的FragmentManager和系统的FragmentManager。

Support FragmentManager：

这个Support FragmentManager的`loadAnimaion`的源码的最后一段，当没有加载到任何动画时才根据`transitionStyle`加载动画，但是源码都把这段给注释了，嘿嘿-_-!。

```
        if (transitionStyle == 0 && mHost.onHasWindowAnimations()) {
                transitionStyle = mHost.onGetWindowAnimations();
            }
            if (transitionStyle == 0) {
                return null;
            }
            //TypedArray attrs = mActivity.obtainStyledAttributes(transitionStyle,
            //        com.android.internal.R.styleable.FragmentAnimation);
            //int anim = attrs.getResourceId(styleIndex, 0);
            //attrs.recycle();

            //if (anim == 0) {
            //    return null;
            //}

            //return AnimatorInflater.loadAnimator(mActivity, anim);
            return null;
```

---
### 1.6 Android 5.0后的ShareElement

利用ShareElement可以实现很绚丽的效果哦，不过这只在5.0以上才支持，具体的做法如下：


首先我们拟定一个场景，有一个显示图片列表的GridFragment，当点击任何一张图片后执行动画过度到一个图片详情的DetaiFragment。

GridFragment中的关键代码：

在列表中给ItemView设置transitionName：

```
                public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
                    ImageView iv = (ImageView) holder.itemView.findViewById(R.id.item_grid_iv);
                    iv.setImageResource(mImages[position]);
                    iv.setTag(position);
                    iv.setOnClickListener(mOnClickListener);
                    ViewCompat.setTransitionName(iv, String.valueOf(position) + "_image");
                }
```

当需要打开新的Fragment时，回调Activity打开新的界面：

```
       //注意做好版本判断
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
      
                detailFragment.setSharedElementEnterTransition(new DetailsTransition());
                
                detailFragment.setEnterTransition(new Slide());
               getSupportFragmentManager()
                        .findFragmentByTag(GridLayoutFragment.class.getName())
                        .setExitTransition(new Slide());
    
                detailFragment.setSharedElementReturnTransition(new DetailsTransition());
            }
            getSupportFragmentManager()
                    .beginTransaction()
                    .addSharedElement(view, DetailFragment.TRANS_NAME)
                    .replace(R.id.act_share_container_rl, detailFragment, DetailFragment.class.getName())
                    .addToBackStack(DetailFragment.class.getName())
                    .commit();


    
        @TargetApi(Build.VERSION_CODES.LOLLIPOP)
        private class DetailsTransition extends TransitionSet {
            DetailsTransition() {
                init();
            }
    
            @TargetApi(Build.VERSION_CODES.LOLLIPOP)
            private void init() {
                setOrdering(ORDERING_TOGETHER);
    
                         addTransition( new ChangeBounds())
                         .addTransition(new ChangeTransform())
                         .addTransition(new ChangeImageTransform());
            }
        }
```


还有一点,DetaiFragment的布局中给view定义一个TransitionName:

```xml
      <ImageView
            android:id="@+id/frag_detai_iv"
            android:transitionName="placekitten"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="100dp"
            tools:src="@drawable/placekitten_1"/>
```

需要注意的地方是两个界面都需要给View定义不同的TransitionName。


- setSharedElementEnterTransition() 指定 View 如何从GridFragment 转换为跳转 DtailFragment 中的 View。
- 调用 setSharedElementReturnTransition() 指定 View 在用户点击返回按钮后如何从跳转 GridFragment 中回到第一个 GridFragment 中。
- ChangeBounds 显示 View 大小和位置变化的动画
- ChangeTransform 显示 View 及其父布局的缩放动画
- ChangeImageTransform 能修改图片的大小或者缩放
- Slide 为滑动效果
- Fade  为渐变效果
- Explode 为爆炸效果
- setEnterTransition 设置View的进场转换
- setExitTransition 设置View的退场转换，不是因为栈顶的fragment出栈
- setReenterTransition 设置因为栈顶的Fragment出栈，而原来的Fragment重新回到栈顶时View的转换效果，如果没有设置，就使用setExitTransition设置的Transition
- setReturnTransition 置View的退场转换，用于fragment出栈，如果没有设置使用setEnterTransition设置的Transition

具体介绍和动画效果，查看[用 Transition 完成 Fragment 共享元素的切换](https://github.com/hehonghuidev/android-tech-frontier/blob/master/issue-35/%E7%94%A8Transition%E5%AE%8C%E6%88%90Fragment%E5%85%B1%E4%BA%AB%E5%85%83%E7%B4%A0%E7%9A%84%E5%88%87%E6%8D%A2.md)

---
## 2 使用Fragmen转场动画的一些注意事项

1. setCustomAnimations会在**内存重启后失效**，如果不在意的话还是可以用的
2. 多个Fragment的同时操作，比如pop一个Fragment时add一个Fragment可能某些动画看起来有些混乱或者不协调。
 - 如果使用`setCustomAnimations`的话，推荐使用属性动画，根据实际情况设置动画延时，可以让动画变得协调一点，具体可以看出这个库[FragmentTransactionExtended](https://github.com/DesarrolloAntonio/FragmentTransactionExtended)
 - 如果使用onCreateAnimation可是动态的设置动画的。
3.  动画执行是需要时间的，最好控制Fragment转场动画时View的onCick事件。