# App重启

在某些功能需求下，可能需要重启我们的App应用，比如更换了App的语言设置，代码如下：

```
        private fun reLaunch() {
            val intent = Intent(activity, BlackActivity::class.java)
            intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            activity.startActivity(intent)
            activity.overridePendingTransition(0, 0)
            android.os.Process.killProcess(android.os.Process.myPid())
        }
```

但是这样重启会有一个不好的体验，App会有一个短暂的白屏，解决方法是在这短暂的时间被显示一个默认的图片或者Logo，比如定义一个专门用于过度的Activity，比如下面的BlackActivity：

```
    class BlackActivity : Activity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            startActivity(intentFor<MainActivity>())
            finish()
        }
    }
```

非常简单，直接打开MainActivity，然后销毁自己，但是关机在于给其定义的theme：

```
    <activity android:name=".main.BlackActivity"
          android:theme="@style/AppTheme.Black"/>
    
    <style name="AppTheme.Black" parent="AppTheme.FullScreen">
        <item name="android:windowFullscreen">true</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="android:windowBackground">@drawable/bg_black_activity</item>
    </style>
    
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:drawable="@color/black"/>
        <item android:drawable="@drawable/ic_logo" android:gravity="center"/>
    </layer-list>
```

使用 `layer-list drawable`创建一个比较美观的窗口背景，设置给用于过度的 BlackActivity，就解决了在App重启时那短暂的白屏问题。

