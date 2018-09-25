# Percent 库解析

---
## 1 Percent使用的正确姿势

不要指定android的`layout_width`和`layout_height.`，而是直接使用`app:layout_widthPercent`等属性，使用`layout_aspectRatio`可以指定子控间的宽高比，同样是使用百分比。

```xml
    <android.support.percent.PercentFrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">


        <ImageView
            android:layout_centerInParent="true"
            android:background="@color/blue"
            android:contentDescription="ddd"
            android:layout_gravity="center"
            app:layout_aspectRatio="80%"
            app:layout_widthPercent="50%"/>

    </android.support.percent.PercentFrameLayout>
```

---
## 引用

- [Android 百分比布局库(percent-support-lib) 解析与扩展](http://blog.csdn.net/lmj623565791/article/details/46695347)
- [Android 增强版百分比布局库 为了适配而扩展](http://blog.csdn.net/lmj623565791/article/details/46767825)