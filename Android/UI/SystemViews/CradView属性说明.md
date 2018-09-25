# CardView 属性说明

```xml
<resources>
    <declare-styleable name="CardView">
        <!-- Background color for CardView. -->
        <!-- 背景色 -->
        <attr name="cardBackgroundColor" format="color" />
        <!-- Corner radius for CardView. -->
        <!-- 边缘弧度数 -->
        <attr name="cardCornerRadius" format="dimension" />
        <!-- Elevation for CardView. -->
        <!-- 高度 -->
        <attr name="cardElevation" format="dimension" />
        <!-- Maximum Elevation for CardView. -->
        <!-- 最大高度 -->
        <attr name="cardMaxElevation" format="dimension" />
        <!-- Add padding in API v21+ as well to have the same measurements with previous versions. -->
        <!-- 设置内边距，v21+的版本和之前的版本仍旧具有一样的计算方式 -->
        <attr name="cardUseCompatPadding" format="boolean" />
        <!-- Add padding to CardView on v20 and before to prevent intersections between the Card content and rounded corners. -->
        <!-- 在v20和之前的版本中添加内边距，这个属性是为了防止卡片内容和边角的重叠 -->
        <attr name="cardPreventCornerOverlap" format="boolean" />
        <!-- 下面是卡片边界距离内部的距离-->
        <!-- Inner padding between the edges of the Card and children of the CardView. -->
        <attr name="contentPadding" format="dimension" />
        <!-- Inner padding between the left edge of the Card and children of the CardView. -->
        <attr name="contentPaddingLeft" format="dimension" />
        <!-- Inner padding between the right edge of the Card and children of the CardView. -->
        <attr name="contentPaddingRight" format="dimension" />
        <!-- Inner padding between the top edge of the Card and children of the CardView. -->
        <attr name="contentPaddingTop" format="dimension" />
        <!-- Inner padding between the bottom edge of the Card and children of the CardView. -->
        <attr name="contentPaddingBottom" format="dimension" />
    </declare-styleable>
</resources>
```

