# FragmentDialog 解析

---
## 1 生命周期

生命周期如下：

```
    onCreate
    getLayoutInflater
    onCreateDialog
    onCreateView
    onActivityCreated
```

---
## 2 源码分析

getLayoutInflater在onCreate后执行：

```java
        public LayoutInflater getLayoutInflater(Bundle savedInstanceState) {
            if (!mShowsDialog) {
                return super.getLayoutInflater(savedInstanceState);
            }
            mDialog = onCreateDialog(savedInstanceState);
            if (mDialog != null) {
                setupDialog(mDialog, mStyle);
                return (LayoutInflater) mDialog.getContext().getSystemService(
                        Context.LAYOUT_INFLATER_SERVICE);
            }
            return (LayoutInflater) mHost.getContext().getSystemService(
                    Context.LAYOUT_INFLATER_SERVICE);
        }
```

这里调用了onCreateDialog，然后是设置Dialog的属性 setupDialog(mDialog, mStyle);

```java
        public void setupDialog(Dialog dialog, int style) {
            switch (style) {
                case STYLE_NO_INPUT:
                    dialog.getWindow().addFlags(
                            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE |
                                    WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE);
                    // fall through...
                case STYLE_NO_FRAME:
                case STYLE_NO_TITLE:
                    dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
            }
        }
```

其实就是为Window添加一些标记，这里我可以看到`STYLE_NO_FRAME`和`STYLE_NO_TITLE`都设置了`Window.FEATURE_NO_TITLE`，然后如果设置了`STYLE_NO_FRAME`还会使用下面style

```
        public void setStyle(@DialogStyle int style, @StyleRes int theme) {
            mStyle = style;
            if (mStyle == STYLE_NO_FRAME || mStyle == STYLE_NO_INPUT) {
                mTheme = android.R.style.Theme_Panel;
            }
            if (theme != 0) {
                mTheme = theme;
            }
        }
    
     <style name="Theme.Panel">
            <item name="windowBackground">@color/transparent</item>
            <item name="colorBackgroundCacheHint">@null</item>
            <item name="windowFrame">@null</item>
            <item name="windowContentOverlay">@null</item>
            <item name="windowAnimationStyle">@null</item>
            <item name="windowIsFloating">true</item>
            <item name="backgroundDimEnabled">false</item>
            <item name="windowIsTranslucent">true</item>
            <item name="windowNoTitle">true</item>
        </style>
```

对于Panel的描述是：删除所有多余的窗口装饰，所以你基本上有一个空的长方形替换你的内容。它使窗口浮动，具有透明背景，并关闭了窗口背后的窗口。

然后我们看一下onActiviyCreate方法

```java
     @Override
        public void onActivityCreated(Bundle savedInstanceState) {
            super.onActivityCreated(savedInstanceState);
            if (!mShowsDialog) {
                return;
            }
            View view = getView();
            if (view != null) {
                if (view.getParent() != null) {
                    throw new IllegalStateException("DialogFragment can not be attached to a container view");
                }
                mDialog.setContentView(view);
            }
            mDialog.setOwnerActivity(getActivity());
            mDialog.setCancelable(mCancelable);
            mDialog.setOnCancelListener(this);
            mDialog.setOnDismissListener(this);
            if (savedInstanceState != null) {
                Bundle dialogState = savedInstanceState.getBundle(SAVED_DIALOG_STATE_TAG);
                if (dialogState != null) {
                    mDialog.onRestoreInstanceState(dialogState);
                }
            }
        }
```

其实就是把onCreateView中返回的View，设置为mDialog的ContentView，到此FragmentDialog的大概流程可以知道了：根据风格创建一个Dialog，把onCreateView返回的View设置给Dialog，然后显示出来，由于Fragment可以很好的处理状态恢复，所以推荐使用DialogFragment。

---
## 3 实现一个 BottomSheet 的 Fragment

```java
    public class TestFragmentDialog extends AppCompatDialogFragment {
    
        private static final String TAG = TestFragmentDialog.class.getSimpleName();
    
        @Override
        public LayoutInflater getLayoutInflater(Bundle savedInstanceState) {
            Log.d(TAG, "getLayoutInflater");
            return super.getLayoutInflater(savedInstanceState);
        }
    
        @Override
        public void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            Log.d(TAG, "onCreate");
    //设置风格样式
            setStyle(DialogFragment.STYLE_NO_TITLE, R.style.full);
        }
    
    
        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            Log.d(TAG, "onCreateView");
    //返回内容布局
            return inflater.inflate(R.layout.test, container, false);
        }
    
        @Override
        public void onActivityCreated(Bundle savedInstanceState) {
            super.onActivityCreated(savedInstanceState);
            Log.d(TAG, "onActivityCreated");
            getView().findViewById(R.id.tv).setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(getContext(), "哈哈", Toast.LENGTH_SHORT).show();
                }
            });
        }
    
        @NonNull
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            Log.d(TAG, "onCreateDialog");
            WindowManager wm = (WindowManager) getContext().getSystemService(Context.WINDOW_SERVICE);
            DisplayMetrics outMetrics = new DisplayMetrics();
            wm.getDefaultDisplay().getMetrics(outMetrics);
            AppCompatDialog dialog = new AppCompatDialog(getActivity(), getTheme());
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setWindowAnimations(R.style.bottom_in_style);
    //设置窗口的布局方向
            dialogWindow.setGravity(Gravity.BOTTOM);
    //如果需要全屏，则把宽度设置为屏幕宽度
            lp.width = outMetrics.widthPixels;
    //高度
            lp.height = (int) (outMetrics.heightPixels * 0.85 + 0.5);
            dialogWindow.setAttributes(lp);
            return dialog;
        }
    }
```

解析： 在onCreate中设置风格样式，因为在 onCreate后 Dialog就创建出来了，必须在Window 创建前设置好窗口属性：

```
       setStyle(DialogFragment.STYLE_NO_TITLE, R.style.full);
```

第一个参数使用DialogFragment的内置参数：

- STYLE_NO_TITLE 不需要title
- STYLE_NORMAL 正常显示
- STYLE_NO_FRAME 不画所有的边框；通过oncreateview返回视图层次完全负责绘制对话框。
- STYLE_NO_INPUT 同STYLE_NO_FRAME，并且获取不到输入焦点


第二个参数使用自定义的style

如果不设置那么dialog不会填充屏幕，即使设置了窗口宽度为屏幕宽度，所以设置一个style：

```
      <style name="full" parent="AlertDialog.AppCompat.Light">
            <item name="android:windowContentOverlay">@null</item>
                //边缘不需要阴影
            <item name="android:windowCloseOnTouchOutside">true</item>
        </style>
```

不使用自定义的style而使用`STYLE_NO_FRAME`也可以实现全屏，但是dialog显示出后没有阴影，体验不好，推荐的theme在下面的示例中。

---
## 4 示例

代码：

```java
    public abstract class BottomSheetFragmentDialog extends AppCompatDialogFragment {
    
        @Override
        public void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            //设置不需要标题和自定义style
            setStyle(DialogFragment.STYLE_NO_TITLE, R.style.CommonFragmentDialog);
        }
    
        @NonNull
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            WindowManager wm = (WindowManager) getContext().getSystemService(Context.WINDOW_SERVICE);
            DisplayMetrics outMetrics = new DisplayMetrics();
            wm.getDefaultDisplay().getMetrics(outMetrics);
            AppCompatDialog dialog = new AppCompatDialog(getActivity(), getTheme());
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setGravity(Gravity.BOTTOM);
            lp.width = outMetrics.widthPixels;
            configDialogAttributes(lp);
            dialogWindow.setAttributes(lp);
            return dialog;
        }
    
        protected abstract void configDialogAttributes(WindowManager.LayoutParams lp);
    }
```

style：

```xml
      <!--Fragment Dialog Style-->
      <style name="FragmentDialog">
            <item name="android:windowAnimationStyle">@style/bottom_in_style</item>
            <item name="android:windowContentOverlay">@null</item>
            <item name="android:backgroundDimEnabled">true</item>
            <item name="android:backgroundDimAmount">0.4</item>
            <item name="android:windowNoTitle">true</item>
            <item name="android:windowCloseOnTouchOutside">true</item>
        </style>
        
        <style name="bottom_in_style">
            <item name="android:windowEnterAnimation">@anim/bottom_in</item>
            <item name="android:windowExitAnimation">@anim/bottom_out</item>
        </style>
```