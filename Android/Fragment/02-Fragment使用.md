#  Fragment 的使用

**内存重启**：指 Activity 在后台因内存不足被回收，当重新回到 Activity 时，系统会重新创建 Activity 与 Activity 中的 fragments，利用屏幕旋转可以模拟这种情况。

---
## 1 Fragment的方式

Fragment的需要通过 FragmentTransaction 来操作，主要的操作方法如下：

| 方法名  |  详述 |
| ------------ | ------------ |
| add  |  添加一个Fragment，最常用的方法 |
| replace  | 如果之前的布局中已经存在Fragment，那么先remove之前的Fragment，在添加此Fragment  |
| remove  |  彻底的移除之前添加Fragment(如果Fragment在backStack中，则无法完全移除) |
| detach  |  从Activity中销移除Fragment的视图，但是此时Fragment依然和Activity关联 |
| attach  |  此方法是与deatch对应的，应该只在detach操作后才调用此方法，作用是重建Fragment的视图，并重新添加到Activity的视图中 |
| hide  | 隐藏Fragment的视图，Fragment的生命周期不会被调用  |
| show  | 与hide方法对应，重新显示Fragment的视图，Fragment的生命周期不会被调用  |


---
## 2 操作Fragment时生命周期回调

### add

当Fragment被add时，它的生命周期调用如下：

```
    Opt1Fragment-->onAttach
    Opt1Fragment-->onCreate
    onCreateView()
    Opt1Fragment-->onViewCreated
    Opt1Fragment-->onActivityCreated
    Opt1Fragment-->onStart
    Opt1Fragment-->onResume
```

### replace

当我们replace一个新的Fragment时，生命周期如下：

```
    Opt2Fragment-->onAttach
    Opt2Fragment-->onCreate
    
    Opt1Fragment-->onPause
    Opt1Fragment-->onStop
    Opt1Fragment-->onDestroyView
    Opt1Fragment-->onDestroy
    Opt1Fragment-->onDetach
    
    onCreateView()
    Opt2Fragment-->onViewCreated
    Opt2Fragment-->onActivityCreated
    Opt2Fragment-->onStart
    Opt2Fragment-->onResume
```

可以看到，之前的Fragment已经被完成移除，而新的Fragment被添加

### remove

```
    Opt2Fragment-->onResume
    Opt2Fragment-->onPause
    Opt2Fragment-->onStop
    Opt2Fragment-->onDestroyView
    Opt2Fragment-->onDestroy
    Opt2Fragment-->onDetach
```

很简单，就是完全移除

### detach

当我们add一个fragment后，可以使用detach来销毁Fragment的视图。
有一点需要注意，detach后Fragment的onStart,onResume，onPause,onStop已经不再和Activity关联了。

```
    //add operation
    Opt1Fragment-->onAttach
    Opt1Fragment-->onCreate
    onCreateView()
    Opt1Fragment-->onViewCre
    Opt1Fragment-->onActivit
    Opt1Fragment-->onStart
    Opt1Fragment-->onResume
    
    //detach operation
    Opt1Fragment-->onPause
    Opt1Fragment-->onStop
    Opt1Fragment-->onDestroyView
```

当然在detach后，可以继续使用remove完全移除Fragment：

```
     Opt1Fragment-->onDestroy
     Opt1Fragment-->onDetach
```

### attach

attach应该只在用在使用了detach销毁了视图的Fragment：

```
    onCreateView()
    Opt1Fragment-->onViewCreated
    Opt1Fragment-->onActivityCreated
    Opt1Fragment-->onStart
    Opt1Fragment-->onResume
```


### show/hide

show和hide是对应的两个方法，用于显示和隐藏fragment的视图，Fragment的生命周期不会被调用，但是Fragment的`onHiddenChanged(boolean hidden)`会被调用。

需要注意的是`onHiddenChanged(boolean hidden)`只会在show和hide操作时回调，其他方式都不会，比如Fragment的添加、Fragment被回收重建。

很多app的主界面主界面采用tab+多个Fragment方式，大部分都会使用show/hide来实现，此种实现需要处理好因**内存重启**造成的Fragment的重叠与错乱问题，因为重建后的Fragments的hide状态不会被FragmentManager记住，所有的Fragment都会以show的方式显示。

1. 当**内存重启**后，先从FragmentManager中根据tag找对应的Fragment，如果没有找到，才去创建Fragment
2. 当**内存重启**后，获取所有被重建的Fragment，根据之前的使用saveInstanceState保存的状态隐藏不应该显示的Fragment
3. 如果是多个Fragment有层级之分则可以使用getFragmens，遍历所有的fragment，只显示最顶层的fragment.

---
## 3 Fragment 中一些方便的方法

```
    isAdded
    isDetached
    isHidden
    isInLayout
    isMenuVisible
    isRemoving
    isVisible
    isResumed
    setTargetFragment
```

---
## 4 需要注意的地方

- Fragment 一般分为两类，一类是有 UI 的 Fragment，可以作为页面，作为 View 来展示，另一类是用没有 UI 的 Fragment，一般用作保存数据。

- 使用静态工厂方法 `newInstance(...)` 来获取Fragment实例，可以在 Google 的示例代码中发现这种写法，好处是接收确切的参数，返回一个Fragment实例，避免了在创建Fragment的时候无法在类外部知道所需参数的问题，在合作开发的时候特别有用。
```
public static WeatherFragment newInstance(String cityName) {
    Bundle args = new Bundle();
    args.putString(cityName, CITY_NAME_KEY);
    WeatherFragment fragment = new WeatherFragment();
    fragment.setArguments(args);
    return fragment;
}
```

- 避免错误操作导致 Fragment 的视图重叠，前面已经提到