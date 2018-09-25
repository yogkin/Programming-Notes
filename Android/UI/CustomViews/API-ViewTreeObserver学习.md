# ViewTreeObserver

ViewTreeObserver在View体系中用来监听view树的全局状态的变化，**这种全局事件包括但不限于：整个视图树的布局发生改变、在视图开始绘制之前、视图触摸模式改变时**

ViewTreeObserver可以为我们所用的方法：

- `addOnDrawListener`
- `addOnGlobalFocusChangeListener`
- `addOnGlobalLayoutListener`
- `addOnPreDrawListener`
- `addOnScrollChangedListener`监听整个视图的滑动行为
- `addOnTouchModeChangeListener`
- `addOnWindowAttachListener`
- `addOnwindowFocusChangeListener`

>查看ViewTreeObserver的源码发现其并不只有这些方法，但是有些监听是被@hide注解隐藏了，@hide注解表示google并没有准备好要公开这些api给开发者用，即有可能会在今后的开发版本中被去掉，所以我们不应该使用。

然后我们对上面监听器进行侧滑，在一个Activity被打开时打印如下log：

```
    1 onWindowAttached() called
    2 onTouchModeChanged() called with: isInTouchMode = [true]
    3 onGlobalLayout() called
    4 onPreDraw() called
    5 onGlobalLayout() called
    4 onPreDraw() called
    6 onWindowFocusChanged() called with: hasFocus = [true]
    4 onPreDraw() called
    5 onPreDraw() called

打开新的Activity

    onPreDraw() called
    onPreDraw() called
    onWindowFocusChanged() called with: hasFocus = [false]
    onPreDraw() called
    onPreDraw() called
    onPreDraw() called
    onPreDraw() called

回到Activity

    onPreDraw() called
    onPreDraw() called
    onWindowFocusChanged() called with: hasFocus = [true]
    onPreDraw() called
```

## getViewTreeObserver()

通过Viwe的getViewTreeObserver方法可以获取到ViewTreeObserver对象，用来添加观察者，其代码如下：

```java
       public ViewTreeObserver getViewTreeObserver() {
            if (mAttachInfo != null) {
                return mAttachInfo.mTreeObserver;
            }
            if (mFloatingTreeObserver == null) {
                mFloatingTreeObserver = new ViewTreeObserver();
            }
            return mFloatingTreeObserver;
        }
```

可见当View还没有被关联到Window上时，是返回的mFloatingTreeObserver。但是当View被管理到window后，这个mFloatingTreeObserver又会被合并

```java
     void dispatchAttachedToWindow(AttachInfo info, int visibility) {
            //System.out.println("Attached! " + this);
            mAttachInfo = info;
            if (mOverlay != null) {
                mOverlay.getOverlayView().dispatchAttachedToWindow(info, visibility);
            }
            mWindowAttachCount++;
            // We will need to evaluate the drawable state at least once.
            mPrivateFlags |= PFLAG_DRAWABLE_STATE_DIRTY;
            if (mFloatingTreeObserver != null) {
                info.mTreeObserver.merge(mFloatingTreeObserver);
                mFloatingTreeObserver = null;
            }
             ......
```

## ViewTreeObserver的这些方法在哪里被调用？

我们知道View树的遍历是由ViewRootImpl来执行的，从测量，布局，绘制都在performTraversals中执行，所以View树的全局状态变化应该也是在这个过程中被调用的。所以试着在ViewRootImpl中搜索一些ViewTreeObserver中方法的关键字，果然搜到了很多被调用的方法：

```java
    boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() ||
                    viewVisibility != View.VISIBLE;
    if (mAttachInfo.mViewScrollChanged) {
                mAttachInfo.mViewScrollChanged = false;
                mAttachInfo.mTreeObserver.dispatchOnScrollChanged();
            }

    mAttachInfo.mTreeObserver.dispatchOnDraw();
    ...
```

## 移除ViewTreeObserver

```java
    getViewTreeObserver().addOnGlobalLayoutListener(new    ViewTreeObserver.OnGlobalLayoutListener() {
    @Override
    public void onGlobalLayout() {
        // 移除监听
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            mHeader.getViewTreeObserver().removeOnGlobalLayoutListener(this);
        } else {
            mHeader.getViewTreeObserver().removeGlobalOnLayoutListener(this);
        }
```









