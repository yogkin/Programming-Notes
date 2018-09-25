## ImageView的缩放类型详解

ImageView的Scaletype决定了图片在View上显示时的样子，如进行何种比例的缩放，及显示图片的整体还是部分，等等。

设置的方式包括：

- 在layout xml中定义android:scaleType="CENTER"
- 或在代码中调用imageView.setScaleType(ImageView.ScaleType.CENTER);

每一种类型的行为：

- ScaleType.CENTER
按照图片原来的尺寸进行居中显示，如果图片的宽高超出View的宽高，则截取图片的居中部分显示
**效果是：图片的大小不会发生改变**
- CENTER_CROP
按比例放大图片，使其居中显示，使得图片的长（宽）大于或等于View的长（宽）
**效果是：图片的宽高都要大于或等于View的宽高**
- CENTER_INSIDE
按比例缩小图片尺寸，使其居中显示，使得图片的长（宽）小于或等于View的长（宽）
**效果是：图片的宽高顶多与view的宽高相等，要么都小于view的宽高**
- FIT_CENTER(默认效果)
按比例缩小或者放大图片，使得图片居中显示
**效果是：图片宽高和view的宽高至少有一个相等**
- FIT_START和FIT_END
在图片缩放效果上与FIT_CENTER一样，只是显示的位置不同，FIT_START是置于顶部，FIT_CENTER居中，FIT_END置于底部。
- FIT_XY
 不按比例缩放图片，目标是把图片塞满整个View。