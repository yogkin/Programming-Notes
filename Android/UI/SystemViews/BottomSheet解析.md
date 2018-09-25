# BottomSheet解析

---
## 1 BottomSheet

使用的前提是：

1. 在CoordinatorLayout
2. 添加`app:layout_behavior="@string/bottom_sheet_behavior"`
3. 可以使用`app:behavior_peekHeight`属性设置默认显示的高度

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <android.support.design.widget.CoordinatorLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:orientation="vertical">
    
            <LinearLayout
                android:orientation="vertical"
                android:layout_width="match_parent"
                android:layout_height="wrap_content">
    
                <TextView
                    android:background="#fbcadf"
                    android:layout_width="match_parent"
                    android:layout_height="1000dp"/>
    
            </LinearLayout>
    
        <LinearLayout
            android:id="@+id/frag_bottom_ll"
            android:background="@color/colorAccent"
            android:layout_width="match_parent"
            android:layout_height="300dp"
            android:orientation="vertical"
            app:behavior_peekHeight="10dp"
            android:padding="16dp"
            app:layout_behavior="@string/bottom_sheet_behavior">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Dandelion Chocolate"
                android:textAppearance="@style/TextAppearance.AppCompat.Display1"
                android:textColor="@android:color/black"/>
        </LinearLayout>

    </android.support.design.widget.CoordinatorLayout>
```

代码中：

```java
     final BottomSheetBehavior behavior = BottomSheetBehavior.from(bottomSheet);
            behavior.setBottomSheetCallback(new BottomSheetBehavior.BottomSheetCallback() {
                @Override
                public void onStateChanged(@NonNull View bottomSheet, int newState) {
                    //这里是bottomSheet 状态的改变，根据slideOffset可以做一些动画
                    Log.i("cjj", "newState--->" + newState);
                    //ViewCompat.setScaleX(bottomSheet,1);
                    //ViewCompat.setScaleY(bottomSheet,1);
                }
    
                @Override
                public void onSlide(@NonNull View bottomSheet, float slideOffset) {
                    //这里是拖拽中的回调，根据slideOffset可以做一些动画
                    Log.i("cjj", "slideOffset=====》" + slideOffset);
                    //ViewCompat.setScaleX(bottomSheet,slideOffset);
                    //.setScaleY(bottomSheet,slideOffset);
                }
            });
```

- `setHideable(true);`设置默认实现的高度可以隐藏
- 通过回调可以监听BottomSheet的状态
- `STATE_COLLAPSED`: 关闭Bottom Sheets,显示peekHeight的高度，默认是0
- `STATE_DRAGGING`: 用户拖拽Bottom Sheets时的状态
- `STATE_SETTLING`: 当Bottom Sheets view摆放时的状态。
- `STATE_EXPANDED`: 当Bottom Sheets 展开的状态
- `STATE_HIDDEN`: 当Bottom Sheets 隐藏的状态

---
## 2 BottomSheetDialog

BottomSheetDialog的使用更加简单，代码如下：

```
     final BottomSheetDialog dialog = new BottomSheetDialog(this);
            View view = LayoutInflater.from(this).inflate(R.layout.dialog_layout, null);
            RecyclerView recyclerView = (RecyclerView) view.findViewById(R.id.bs_rv);
            recyclerView.setLayoutManager(new LinearLayoutManager(this));
            SimpleStringRecyclerViewAdapter adapter = new SimpleStringRecyclerViewAdapter(this);
            adapter.setItemClickListener(new SimpleStringRecyclerViewAdapter.ItemClickListener() {
                @Override
                public void onItemClick(int pos) {
                    dialog.dismiss();
                    Toast.makeText(BottomSheetDialogActivity.this, "pos--->" + pos, Toast.LENGTH_LONG).show();
                }
            });
            recyclerView.setAdapter(adapter);
            dialog.setContentView(view);
            dialog.show();
```

但是有一个问题，如果不在内容区域拖动BottomSheetDialog。一松手Dialog就会消失，查看起源码实现中有这样一段代码：

```
      if (shouldWindowCloseOnTouchOutside()) {
                final View finalView = view;
                coordinator.setOnTouchListener(new View.OnTouchListener() {
                    @Override
                    public boolean onTouch(View v, MotionEvent event) {
                        if (isShowing() &&
                                MotionEventCompat.getActionMasked(event) == MotionEvent.ACTION_UP &&
                                !coordinator.isPointInChildBounds(finalView,
                                        (int) event.getX(), (int) event.getY())) {
                            cancel();
                            return true;
                        }
                        return false;
                    }
                });
            }
```

在Dialogshow之后，加上下面代码即可解决：

```
      View decorView = dialog.getWindow().getDecorView();
            View viewById = decorView.findViewById(R.id.design_bottom_sheet);
            //parent就是coordinator
            ViewGroup parent = (ViewGroup) viewById.getParent();
            parent.setOnTouchListener(null);
```

当然如果需要更好的体验，可以判断是否为点击事件，是的就cancel也是可以的。

---
## 3 总结

感觉 BottomSheet 没有太大的用处，如果在一个可以滑动的列表控件中，是无法使用 BottomSheet 的，所以使用有局限性，不过 BottomSheetDialog 还是挺有用的。