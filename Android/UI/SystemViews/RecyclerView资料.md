# RecyclerView 资料

## 1 入门

### 1.1 RecyclerView相关组件介绍

- `Adapter` 托管数据集合，为每个Item创建视图
- `ViewHolder`  承载Item视图的子视图
- `LayoutManager` 负责Item视图的布局
- `ItemDecoration`  为每个Item视图添加子视图，比如用来绘制Divider，`onDrawOver`绘制在最上面
- `ItemAnimator`  负责添加、删除数据时的动画效果，默认使用内部的DefaultAnimator，可以参考DefaultAnimator来实现自己的ItemAnimator
- `ItemTouchHelper`用来帮助实现RecyclerView Item的滑动帮助类
- `OnItemTouchListener`用来处理Item的触摸事件接口
- `SnapHelper`  用来帮助实现Snap滑动
- `RecycledViewPool` 多个RecyclerView可以共享一个RecycledViewPool
- `SortedList`如果你的列表需要**批量更新或者频繁删改**，且刚好有明确的**先后顺序**，可以使用`SortedList`。`SortedList`在RecyclerView中，可以很方便的实现这样的功能。如果使用`SortedList`作为RecyclerView数据源，在往集合中添加或者删除数据时，SortedList会自动的通知RecyclerView的Adapter然后RecyclerView调整视图。这一点是很方便的。
- `DiffUtil`当Adapter的数据源发生变化是，使用DiffUtil提供的算法算出数据源的差异，然后让DiffUtil通知Adapter哪些Item需要被刷新
- `AsyncListUtil`用于异步从数据库价值数据

---
### 1.2 使用步骤

- 实例化RecyclerView
- 为RecyclerView设置LayoutManager
- 为RecyclerView设置Adapater
- 如果有需求，可以设置一个或多个ItemDecorations，当然，也可以不设置
- 如果有需求，可以设置ItemAnimator

---
### 1.3 有用的方法

#### RecyclerView
```
    //滑动到当前Scroll + offset
    mRecyclerView.smoothScrollBy(0, 300);
    //滑动到指定位置，position已经显示则不会移动，如果从上往下移动到目标位置则靠上,否则靠下
    mRecyclerView.smoothScrollToPosition();
    //在给定的点上找到最顶端的视图
    findChildViewUnder(float x, float y)
    //根据view找对对应的ItemView
    findContainingItemView(View view)
    //根据位置找到对应的ViewHolder
    findViewHolderForAdapterPosition(int position)
```

#### LayoutManager

```
findFirstVisibleItemPosition() 返回当前第一个可见Item的position
findFirstCompletelyVisibleItemPosition() 返回当前第一个完全可见Item的position
findLastVisibleItemPosition() 返回当前最后一个可见Item的position
findLastCompletelyVisibleItemPosition() 返回当前最后一个完全可见Item的position
```

---
## 2 `stableIds(true)` 与 `getItemId(int position)` 的作用

- [How to Avoid That RecyclerView’s Views are Blinking when notifyDataSetChanged()](https://medium.com/@hanru.yeh/recyclerviews-views-are-blinking-when-notifydatasetchanged-c7b76d5149a2)

在调用 notifyDataSetChanged 的时候可能会导致 Item 闪屏，`StableIds(true)` 与 `getItemId(int position)` 可以避免这样的问题：

- `stableIds(true)`：一般子构造 Adapter 时我们将其设置为 true，然后 Adapter 应该为数据源中的分配一个 key，key的作用是在在调用 notifyDataSetChanged 后，用于标识这个位置上的 item 是不是原来的那个。
- `getItemId(int position)`：用于返回 Item 的唯一 id，不推荐返回 positon，而应该返回具有唯一性的业务 id。

使用 stableIds 后，对应相同的 ID，RecyclerView 会尝试使用相同的 ViewHolder 和 View。

---
## 3 相关资源

### LayoutManager

- [greedo-layout-for-android-可以顶边Item宽度百分比的布局管理器](https://github.com/500px/greedo-layout-for-android)
- [RecyclerView的CarouselLayoutManager布局管理器，旋转木马](https://github.com/Azoft/CarouselLayoutManager)
- [CarouselLayoutManager](https://github.com/Azoft/CarouselLayoutManager)
- [ChipsLayoutManager](https://github.com/BelooS/ChipsLayoutManager)
- [flexbox-layout](https://github.com/google/flexbox-layout)
- [turn-layout-manager](https://github.com/cdflynn/turn-layout-manager)
- [GalleryLayoutManager](https://github.com/BCsl/GalleryLayoutManager)

### 打造属于自己的LayoutManager

*   [第一部分](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-9/%E5%88%9B%E5%BB%BA-RecyclerView-LayoutManager-Part-1.md)
*   [第二部分](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-13/%E5%88%9B%E5%BB%BA-RecyclerView-LayoutManager-Part-2.md)
*   [第三部分](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-13/%E5%88%9B%E5%BB%BA-RecyclerView-LayoutManager-Part-3.md)
*   [第四部分](https://github.com/hehonghui/android-tech-frontier/blob/master/issue-13/%E5%88%9B%E5%BB%BA-RecyclerView-LayoutManager-Redux.md)

### 使用与原理

- [RecyclerView剖析](http://blog.csdn.net/qq_23012315/article/details/50807224)
- [RecyclerView源码剖析](https://blog.saymagic.tech/2016/10/21/understand-recycler.html)
- [【腾讯Bugly干货分享】RecyclerView 必知必会](https://www.cnblogs.com/bugly/p/6264751.html)
