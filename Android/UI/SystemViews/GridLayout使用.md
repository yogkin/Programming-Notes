# GridLayout使用说明

当需要实现一个表格布局时，使用GridLayout可能会事半功倍。

```xml
    <android.support.v7.widget.GridLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:columnCount="3"
        app:rowCount="7">
    </android.support.v7.widget.GridLayout>
```

`columnCount`用于指定GridLayout的列数
`rowCount用于`指定GridLayout的行数

GridLayout中子View可使用的属性

```xml
    app:layout_column="1"//表示子view在第几列
    app:layout_columnSpan="2"//表示子view占用几列
    app:layout_gravity="fill_horizontal"//子view的对齐方式
    app:layout_row="0"//表示子view在第几行
    app:layout_columnWeight="1"//子view的权重
```

除了在xml中添加GridLayout的子view，还可以使用代码添加。从而可以动态构建Grid。

