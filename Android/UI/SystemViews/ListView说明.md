
---
## 1 ListView的滑动相关方法

```
   //滑动到目标位置，始终是把目标位置移动到顶部
  mListView.setSelection(int position);

   //滑动到目标位置10 偏移顶部100px ，始终是把目标位置移动到顶部
  mListView.setSelectionFromTop(10, 100);

   //position已经显示 listView不会移动，可见及所选，    如果从上往下移动到目标位置则靠上  否则靠下
  mListView.smoothScrollToPosition(30);

  mListView.smoothScrollByOffset(int offset);//滑动到当前位置 + offset 的位置

  mListView.smoothScrollBy(int distance, int duration);//y方向滑动多少 滑动时间  

  //滑动到区间， 区间内已经显示 listView不会移动  ，可见及所选，  如果从上往下移动到目标位置则靠上  否则靠下
  mListView.smoothScrollToPosition(20 , 30);

  //滑动到指定位置并且靠上 ，偏移顶部距离， 滑动时间
  mListView.smoothScrollToPositionFromTop(10 , 100 , 1000);

  //当有新的数据添加到集合的底部，listview自动滑动到底部，添加到集合前面不会
  mListView.setTranscriptMode(AbsListView.TRANSCRIPT_MODE_ALWAYS_SCROLL);

  //回到顶部
  requestFocusFromTouch()
  lv.setSelection(0);
```

---
## 2 ListView的一些重要属性

### 使用transcriptMode和stackFromBottom属性

ListView有两个比较特殊的属性android:transcriptMode和android: stackFromBottom。使用transcriptMode属性可以在有数据变化的时候让listView自动滚动到底部。transcriptMode可设置为一下三个不同的值：

  - disabled：禁用transcriptMode属性；
  - normal：如果最后一个item可见，滚动到底部；
  - alwaysScroll：总是自动滚动到底部；

### stackFromBottom

使用stackFromBottom属性可以设置item从底部向上开始排列。通常在聊天、短信类型的应用中使用stackFromBottom和transcriptMode属性可以得到很好的显示效果。

### cacheColorHint属性

cacheColorHint属性，很多人希望能够改变一下它的背景，使他能够符合整体的UI设计，改变背景背很简单只需要准备一张图片然后指定属性 `android:background="@drawable/bg"`，不过不要高兴地太早，当你这么做以后，发现背景是变了，但是当你拖动，或者点击list空白位置的时候发现ListItem都变成黑色的了，破坏了整体效果。
