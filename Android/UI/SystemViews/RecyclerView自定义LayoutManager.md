# 自定义RecyclerView LayoutManager

---
## 1 RecyclerView缓存机制

RecyclerView内部View的回收与利用由Recycler实现，当需要从一个可能再生的前子View中回收旧的view或者获取新的view时，LayoutManager都需要通过Recycler实例获取。

当LayoutManager需要一个新的子View时，只要调用getViewForPosition()这个方法，Recycler会决定到底是从头创建一个新的视图View还是重用一个已存在的废弃View。 注意：自定义LayoutManager时需要及时将不再显示的视图传递给Recycler， 以避免Recycler创建不必要的View对象。


### 两级缓存

布局更新时有两个方法处理已存在的子视图：detach(分离)和remove(移除)

- detach：一个轻量的记录view操作。被detach的视图在你的代码返回前能够重新连接。可以通过Recycler在不重新绑定/重新构建子视图的情况下修改已连接子视图的索引。
- remove：意味着这个view已经不需要了。任何被永久移除的view都应该放到Recycler中，方便以后重用，不过API并没有强制要求。 被remove的视图是否被回收取决于你。


Scrap和Recycle缓存：

- Scrap：一个轻量的集合，视图可以不经过适配器直接返回给LayoutManager 。通常被detach但会在同一布局重新使用的视图会临时储存在这里
- Recycle：存放的是那些假定并没有得到正确数据(相应位置的数据)的视图， 因此它们都要经过适配器重新绑定后才能返回给LayoutManager。


当要给 LayoutManager 提供一个新 view 时，Recycler 首先会 检查 scrap heap 有没有对应的 position/id；如果有对应的内容， 就直接返回数据不需要通过适配器重新绑定。如果没有的话， Recycler 就会从 recycle pool 里弄一个合适的视图出来， 然后用 adapter 给它绑定必要的数据 (就是调用 RecyclerView.Adapter.bindViewHolder()) 再返回。 如果 recycle pool 中也不存在有效 view ，就会在绑定数据前 创建新的 view (就是 RecyclerView.Adapter.createViewHolder())， 最后返回数据。


对应的API：

- removeAndRecycleAllViews
- detachAndScrapAttachedViews
- removeAndRecycleView
- detachAndScrapView

---
## 2 实现方法

### generateDefaultLayoutParams()

返回一个你想要默认应用给所有从Recycler中获得的子视图做参数的 RecyclerView.LayoutParams 实例。 这些参数会在对应的 getViewForPosition() 返回前赋值给相应的子视图。


### onLayoutChildren()

onLayoutChildren() 是 LayoutManager 的主入口。 它会在view需要初始化布局时调用，当适配器的数据改变时(或者整个适配器被换掉时)会再次调用。 注意！这个方法不是在每次你对布局作出改变时调用的。 它是初始化布局或者在数据改变时重置子视图布局的好位置。


### canScrollHorizontally() & canScrollVertically()

实现布局的滑动交互


