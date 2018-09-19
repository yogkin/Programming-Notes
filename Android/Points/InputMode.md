## android的软件盘属性讲解

在Android开发中，经常使用到EditText,让用户输入文本，但是如果对软键盘不是很熟悉，可能对很多现象的发送不是很了解，比如进入到某一个有EditText的界面，软键盘就自动弹出，但是有时候又不会，这是为什么呢？这就需要了解软键盘的一些属性了。

首先如何设置软件盘的相关属性呢？

**方式1：在manifest文件的Activity节点中设置**

```
      <activity
            android:name=".activity.OrderActivity"
            android:exported="false"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="adjustPan"/>
```

使用的属性是：windowSoftInputMode

**方式2：在代码中设置**

```
	getDialog().getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);

    View view = activity.getCurrentFocus();
    if (view != null) {
        InputMethodManager imm = (InputMethodManager) view.getContext().getSystemService(
                Context.INPUT_METHOD_SERVICE);
        return imm.showSoftInput(view, InputMethodManager.SHOW_FORCED);
    }
```

通过上面列出的方式就可以对软键盘的模式进行设置了，那么软键盘都有哪些模式呢，都有什么作用呢？

## 软键盘的模式与作用

首先，windowSoftInputMode属性一共有10个取值，分别是：

```
    stateUnspecified
    stateUnchanged
    stateHidden
    stateAlwaysHidden
    stateVisible
    stateAlwaysVisible
    adjustUnspecified
    adjustResize
    adjustPan
    adjustNothing
```

他们可以分为两类，一类是state类，控制软键盘显示，一类是adjust类，控制软键盘与界面上控件布局的关系

#### stateUnspecified

软件默认采用的就是这种交互方式，然后**系统会根据界面采取相应的软键盘的显示模式**
- 当界面上只有按钮和文本时，软键盘肯定不会弹出
- 当界面上有一个**获取到焦点的输入框**并且**有滚动需求时**软键盘会自动弹出来。

#### stateUnchanged

当前界面的软键盘状态，取决于上一个界面的软键盘状态

#### stateHidden

键盘状态一定是隐藏的，不管上个界面什么状态，也不管当前界面有没有输入的需求，反正就是不显示。因此，我们可以设置这个属性，来控制软键盘不自动的弹出。

#### stateAlwaysHidden

当该Activity主窗口获取焦点时，软键盘也总是隐藏的

#### stateVisible

设置为这个属性，可以将软键盘召唤出来，即使在界面上没有输入框的情况下也可以强制召唤出来。

#### stateAlwaysVisible

设置为这个属性，也可以将软键盘召唤出来，与stateVisible不同之处在于：
如果当前界面的软键盘是弹出的，当点击某一按钮进入到写一个界面，但是下一个界面由于没有获取到焦点的输入框(等等原因)，导致软键盘隐藏，当回到上一个界面时：如果属性是stateAlwaysVisible，那么软键盘会再次弹出来，如果属性是stateVisible则不会

#### adjustUnspecified

这个选项是默认的设置模式。在这情况下，系统会根据界面选择不同的模式。

- 如果界面里面有可以滚动的控件，比如ScrowView，系统会减小可以滚动的界面的大小，从而保证即使软键盘显示出来了，也能够看到所有的内容。标题栏不会受影响
- 如果布局里面没有滚动的控件，那么软键盘可能就会盖住一些内容。当点击界面较下的输入框时，标题栏可能被顶上去。

上面所说的有滚动控件，要求滚动控件式根布局哦！！！

#### adjustResize

这个属性表示Activity的主窗口总是会被调整大小，从而保证软键盘显示空间。标题栏不会受影响：
- 对于没有滚动控件的布局来说，当点击偏下面的输入框时，当弹出软键盘后，可能输入框被盖住。
- 对于有滚动控件的布局来说，则不存在这样的问题，可以随意滚动。

#### adjustPan

Activity的屏幕大小并不会调整来保证软键盘的空间，而是采取了另外一种策略，系统会**通过布局的移动**，**来保证用户要进行输入的输入框肯定在用户的视野范围里面**.从而让用户可以看到自己输入的内容。

- 对于没有滚动控件的布局来说，这个其实就是默认的设置，如果我们选择的位置偏下，上面的标题栏和部分控件会被顶上去。

#### adjustNothing

界面不做任何调整








