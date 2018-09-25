# ConstraintLayout 使用

---
## 1 对齐约束

```
            <attr name="layout_constraintLeft_toLeftOf"/>
            <attr name="layout_constraintLeft_toRightOf"/>
            <attr name="layout_constraintRight_toLeftOf"/>
            <attr name="layout_constraintRight_toRightOf"/>
            <attr name="layout_constraintTop_toTopOf"/>
            <attr name="layout_constraintTop_toBottomOf"/>
            <attr name="layout_constraintBottom_toTopOf"/>
            <attr name="layout_constraintBottom_toBottomOf"/>
            <attr name="layout_constraintBaseline_toBaselineOf"/>
            <attr name="layout_constraintStart_toEndOf"/>
            <attr name="layout_constraintStart_toStartOf"/>
            <attr name="layout_constraintEnd_toStartOf"/>
            <attr name="layout_constraintEnd_toEndOf"/>
```

取值范围可以是`parent`或者parent内的`其他View的id`。对齐约束与RelativeLayout不同的是，这里的约束使用的是类似拉力的实现方式。

---
## 2 设置对齐偏移比例

当为目标控件设置好横纵向的约束时，偏移才会生效

```
    app:layout_constraintLeft_toLeftOf=”parent”
    app:layout_constraintRight_toRightOf=”parent”
    或者
    app:layout_constraintTop_toTopOf=”parent”
    app:layout_constraintBottom_toBottomOf=”parent”

    layout_constraintHorizontal_bias 横向比例
    layout_constraintVertical_bias 纵向比例
```

---
## 3 设置 gone 之后的 margin

```
            <attr name="layout_goneMarginLeft"/>
            <attr name="layout_goneMarginTop"/>
            <attr name="layout_goneMarginRight"/>
            <attr name="layout_goneMarginBottom"/>
            <attr name="layout_goneMarginStart"/>
            <attr name="layout_goneMarginEnd"/>
```

---
## 4 MatchConstraint 与 Ratio（横纵比）

ConstraintLayou的layout_width和layout_height不支持设置match_parent，其属性取值只有以下三种情况

*   wrap_content；
*   指定具体dp值；
*   0dp（match_constraint)，这里的0有特殊意义，表示匹配约束。

当设置为0dp时，可以使用layout_constraintDimension属性，默认为宽比高。

```
    app:layout_constraintDimensionRatio="2:1"/*默认为宽比高*/
    app:layout_constraintDimensionRatio="W,16:6"/*高比宽*/
    app:layout_constraintDimensionRatio="H,16:6"/*宽比高*/
```

---
## 5 Chain

![](index_files/dfe39caa-0a08-471b-b3c9-15642a04c7c7.jpg)

```
    <attr name="layout_constraintHorizontal_weight"/>
    <attr name="layout_constraintVertical_weight"/>
    <attr name="layout_constraintHorizontal_chainStyle"/>
    <attr name="layout_constraintVertical_chainStyle"/>
```

---
## 6 GuideLine

设置一个基准线，其他View可以以此作为基准点来设置位置。

```
        <android.support.constraint.Guideline
            android:id="@+id/guideline_right"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.8"/>
```

---
## 7 Circular Positioning

让一个控件以另一个控件的中心为中心点，来设置其相对与该中心点的距离和角度。

```
layout_constraintCircle：引用另一个控件的 id。
layout_constraintCircleRadius：到另一个控件中心的距离。
layout_constraintCircleAngle：控件的角度（顺时针，0 - 360 度）。
```

---
## 8 Dimensions（百分比和最大最小约束）

当控件的尺寸设置为了`MATCH_CONSTRAINT`时(即0dp），可以设置相对于父控件的宽高的百分比。可设置的值在`0 -1 `之间，1就是100%，也可以设置最小或最小值，min和max指示的值可以是Dp中的一个维度，也可以是`wrap`，它将使用与WRAP_CONTENT相同的值。

```
layout_constraintWidth_min
layout_constraintHeight_min

layout_constraintWidth_max
layout_constraintHeight_max

layout_constrainWidth_percent
layout_constrainHeight_percent
```


---
## 9 Enforcing constraints 强制约束

在 1.1 版本之前，如果将控件的尺寸设置为了WRAP_CONTENT，那么对控件设置约束（如：`app:layout_constraintWidth_max 和 layout_constraintHeight_min`等）是不起作用的。强制约束（Enforcing constraints）的作用就是，在控件被设置 WRAP_CONTENT 的情况下，使约束依然生效。

```
app:constrainedWidth="true|false"
app:constrainedHeight="true|false"
```

---
## 10 Margins and chains

在 1.1.0-beta4 版本中（已知），为链中的控件设置 marginRight/End 是无效的。而在 1.1 稳定版中，无论设置右边距还是左边距都是有效果的，会累计计算。并且在计算剩余空间时，会将边距一起考虑。

---
## 11 Optimizer


当我们使用 MATCH_CONSTRAINT 时，ConstraintLayout 将不得不对控件进行 2 次测量，而测量的操作是昂贵的。而优化器（Optimizer）的作用就是对 ConstraintLayout 进行优化，对应设置给 ConstraintLauyout 的属性是：`layout_optimizationLevel`。

```
none：不应用优化。
standard：仅优化直接约束和屏障约束（默认的）。
direct：优化直接约束。
barrier：优化屏障约束。
chain：优化链约束（实验）。
dimensions：优化尺寸测量（实验）。
```

可以设置多个值：

```
app:layout_optimizationLevel="direct|barrier|dimensions"
```

---
## 12 Barrier

```
barrierDirection：设置 Barrier  所创建的位置。可设置的有：bottom、end、left、right、start、top。

constraint_referenced_ids：设置 Barrier 引用的控件。可设置多个，设置的方式是：id, id。（无需加 @id/）

barrierAllowsGoneWidgets：默认为 true，即当 Barrier 引用的控件被 GONE 掉时，则 Barrier 默认的创建行为是在已 GONE 掉控件的已解析位置上进行创建。如果设置为 false，则不会将 GONE 掉的控件考虑在内。
```

---
## 13 Group

Group 的作用就是控制一组控件的可见性。其可使用到的属性为：`constraint_referenced_ids`：指定所引用控件的 id。

```
<android.support.constraint.Group
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone"
    app:constraint_referenced_ids="title, desc" />
```

如果有多个 Group，是可以同时指定相同的控件的，最终是以 XML 中最后声明的 Group 为准。

---
## 14 Placeholder

Placeholder（占位符）是一个虚拟对象，作用和它的名字一样，就是占位。当放置好 Placeholder 后，可以通过 setContentId() 方法将占位符变为有效的视图。如果视图已经存在于屏幕上，那么视图将会从原有位置消失。除此之外，还可以通过 setEmptyVisibility() 方法设置当视图不存在时占位符的可见性。

---
## 15 ConstraintSet

在 LinearLayout、RelativeLayout 等中，如果想通过代码来更改布局，则需要 LayoutParams 来控制控件的大小位置等。但是在 ConstraintLayout 中官方不建议使用 LayoutParams，而是推荐使用 ConstraintSet 来使用。ConstraintSet 不仅可以调整布局，还可以添加动画。


---
## 引用

- [android-ConstraintLayoutExamples](https://github.com/googlesamples/android-ConstraintLayoutExamples)，官方示例
- [ConstraintLayout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html)
- [ConstraintLayout 完全解析 快来优化你的布局吧](http://blog.csdn.net/lmj623565791/article/details/78011599)
- [解析ConstraintLayout的性能优势](https://mp.weixin.qq.com/s/gGR2itbY7hh9fo61SxaMQQ)
- [Android 约束布局（ConstraintLayout）1.1.0 版详解](https://juejin.im/post/5adc8bd251882567236e58c1)
