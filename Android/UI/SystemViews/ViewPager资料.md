# ViewPager

---
## ViewPager FragmentPagerAdapter 刷新问题

FragmentPagerAdapter 的 destroyItem 中没有 remove Fragment，而是使用的 detach，这样的实现对 Adapter 的刷新肯定是存在问题的。

一种解决方法是把 FragmentPagerAdapter 源码复制到另一个包，然后重写其 destroyItem 方法

```java
  @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }
        if(getItemPosition(object) == PagerAdapter.POSITION_NONE){
            mCurTransaction.remove((Fragment) object);
        } else {
            mCurTransaction.detach((Fragment) object);
        }
    }
```

然后子类在实现新的 FragmentPagerAdapter 时视情况来复写 getItemPosition 方法：

```kotlin
  override fun getItemPosition(fragment: Any): Int {
        /*
        problem
            0-all
            0-my 1-all
            0-my
       fix
            0-all
            0-my(POSITION_NONE remove) 1-all
            0-all
         */
        for ((index, page) in fragmentPages.withIndex()){
            if(fragment == page.fragment/*数据集中的fragment*/){
                return index
            }
        }
        //返回 POSITION_NONE 将会被移除
        return PagerAdapter.POSITION_NONE
    }
```


---
## 相关资料

### 开源库

- [ ViewPager动画系列]( http://www.lightskystreet.com/2014/12/15/viewpager-anim/ )
- [ViewPager3D， not use view pager transformer ](https://github.com/geminiwen/AndroidCubeDemo)
- [DraggedViewPager Pager可拖动-Item可拖动](https://github.com/yueban/DraggedViewPager)
- [Android-ConvenientBanner:通用的广告栏控件，让你轻松实现广告头效果。支持无限循环，可以设置自动翻页和时间](https://github.com/saiwu-bigkoo/Android-ConvenientBanner)
- [CoverFlowPager-ViewPager实现的画廊](https://github.com/Hongchae/CoverFlowPager)
- [WoWo可以优化你的App介绍/引导页面，制作你的App简历。WoWo将动画和viewpager结合起来。](https://github.com/Nightonke/WoWoViewPager/blob/master/README-ZH.md)
- [模仿 Play 报刊的 Demo](https://github.com/naman14/PlayNewsStandDemo)
- [3dViewPager动画](https://github.com/ToxicBakery/ViewPagerTransforms)
- [垂直滑动的ViewPage](https://github.com/Telenav/ExpandablePager)
- [LinkedViewPager——ViewPager联动](https://github.com/jianghejie/LinkedViewPager)
- [四个方向的ViewPager](https://github.com/DevLight-Mobile-Agency/InfiniteCycleViewPager)

### ViewPager同时显示多个Pager

- [florent37 / HollyViewPager](https://github.com/florent37/HollyViewPager)
- [Pixplicity / MultiViewPager](https://github.com/Pixplicity/MultiViewPager)
- [CleverRecyclerView 是一个基于RecyclerView的扩展库，提供了与ViewPager类似的滑动效果并且添加了一些有用的特性。](https://github.com/luckyandyzhang/CleverRecyclerView)


###  ViewPager 动画详解

- [500px-guideview](https://github.com/hanks-zyh/500px-guideview)

### 轮播图

- [DecentBanner](https://github.com/chengdazhi/DecentBanner)
- [仿京东、淘宝，自动轮播图。Auto Play ViewPager](https://github.com/xyzlf/AutoPlayViewPager)
[轮播图View，很强大](https://github.com/sayyam/carouselview)


### Blog

- [ViewPager刷新问题详解](http://www.jianshu.com/p/266861496508)
- [ViewPager源码解惑](http://www.jianshu.com/p/85afaf9e8f6e)
