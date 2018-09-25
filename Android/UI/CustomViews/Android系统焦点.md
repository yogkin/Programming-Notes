# Android系统的焦点

大多数Android设备都是触摸屏的，但是实际上Android设备也支持键盘操作，允许通过键盘来完成导航，点击，输入等。

操作模式：

- 键盘，轨迹球
- 触摸操作

## 两种模式的区别

**当用户处于键盘，轨迹球操作模式时**，就有必要聚焦当前的UI控件元素，例如，高亮（聚焦）某个按钮，让用户知道当前正在操作的UI元素是哪个。**但是，当用户使用触摸屏与设备交互的时候**，始终聚焦当前UI元素就没有必要了，而且很丑陋；用户点击哪个元素，哪个元素就是当前元素，无需高亮标识。并且，通过触摸屏与设备交互的时候，点击某个UI元素也不会导致该元素聚焦，此时的高亮效果是由Pressed状态来完成的。也就是说，在Touch Mode模式之下，UI元素是不会进入聚焦状态的，即使调用requestFocus也不会。所以为了区分这两种模式，Android定义了 **Touch Mode**：当用户开始通过键盘与设备交互的时候，设备就退出Touch Mode模式；当用户开始通过触摸屏与设备交互的时候，设备就进入Touch Mode模式。可以通过调用View的isInTouchMode来判断设备当前是否处于Touch Mode模式。

### Edittext可以在触摸模式获取焦点

但是，也有例外情况。有些UI元素，即使是在Touch Mode的状态之下，也需要获得焦点，典型的就是Edittext。那么，这种情况该如何处理呢？

### 如何处理触摸模式下的焦点

答案就是做特殊处理。Android规定，某些元素，即使是在Touch Mode模式下，也可以获得焦点。调用View的setFocusableInTouchMode(true)可以使View在Touch Mode模式之下仍然可获得焦点（像Edittext就是在内部设置了这个属性），调用isFocusableInTouchMode可以判断View是否可在Touch Mode模式下聚焦。

### 如何使用焦点

没设置这个属性的控件在用户触摸交互时是不会获得focus的，也就是说focus在touch过程中是不会改变的，只是其onClickListener如果设置了的话会在up事件到来时触发。而如果设置了focusableInTouchMode属性的话，它的行为是首先尝试获得focus，如果获得成功的话其onClickListener是不会触发的，只有当你第2次再点击它时，才会执行onClickListener，所以一般情况下不推荐设置空间可以在触摸模式下获取焦点。

---
## 引用

- [官方博客](http://android-developers.blogspot.jp/2008/12/touch-mode.html)

