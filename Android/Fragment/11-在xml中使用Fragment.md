# Fragment 在布局中定义与代码中使用时的一点区别

---
## 1 在xml定义Fragment

```
    <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <fragment
                android:id="@+id/two_fragment_1"
                android:name="com.ztiany.fragmenttest.TwoFragment"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:tag="com.ztiany.fragmenttest.TwoFragment_1"/>
    
            <fragment
                android:id="@+id/two_fragment_2"
                android:name="com.ztiany.fragmenttest.TwoFragment"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_toEndOf="@id/two_fragment_1"
                android:layout_toRightOf="@id/two_fragment_1"
                android:tag="com.ztiany.fragmenttest.TwoFragment_2"
                tools:ignore="RtlHardcoded"/>
```

1. 在布局中的Fragment同样支持设置xml属性。
2. 在xml中定义的Fragment不受代码中事物操作的影响。

---
## 2 生命周期

```
    XmlFragment: onAttach()
    XmlFragment: -->onCreate  savedInstanceState   =   null
    XmlFragment: -->onViewCreated  savedInstanceState   =   null
    XmlMenuFragment: -->onActivityCreated savedInstanceState   =   null

    CodeFragment: onAttach()
    CodeFragment: -->onCreate  savedInstanceState   =   null
    CodeFragment: -->onViewCreated  savedInstanceState   =   null
    CodeFragment: -->onActivityCreated savedInstanceState   =   null
```

可以看到在xml中定义的Fragment的生命周期方法早于在代码中使用的Fragment，这也是可一理解的，xml中的Fragment是从setContentView开始的解析的。