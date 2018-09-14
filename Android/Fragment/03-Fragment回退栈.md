# Fragment 回退栈

利用BackStack可以实现Fragment的进栈、出栈。像Activity的一样，backStack的应用场景还是比较多的。

---
## 1 backStack

使用`addToBackStack`即可以把Fragment加入到BackStack中，这个方法可以接受一个String，可以作为Fragment在BackStack中的标识，如果不需要标识，可以传null,但是我一般都会使用。

```
     mFragmentManager.beginTransaction().
    .replace(mLayoutId, mCurrentFragment,mCurrentFragment.getClass().getName());
    .replace.addToBackStack(mCurrentFragment.getClass().getName());
    .replace.commit();
```

被加入到BackStack中的Fragment就可以响应返回键了，当执行`onBackPress`时，系统首先判断是否有Fragment在BackStack中，如果有，则先让Fragment出栈，直到所有的Fragment出栈了，Activity才会退出。


还有两个方法也与Fragment的BackStack操作有关，但是不常用：

```
    setBreadCrumbShortTitle(CharSequence/StringRes)
    setBreadCrumbTitle(CharSequence/StringRes)
```

使用这些方法设置的信息，我们都可以使用FragmentManager获取到：

---
## 2 FragmentManager

FragmentManager就是Fragment的管理者了(是不是废话)，所有的Fragment操作以及信息的获取都要通过FragmentManager。

```
    getFragments()//获取FragmentManager管理的Fragments(不管是否在栈中)
    mFragmentManager.getBackStackEntryCount();//获取BackStack中Fragment的个数
    mFragmentManager.getBackStackEntryAt(1);//通过索引获取BackStack中对应Fragment的信息
    mFragmentManager.getFragment(Bundle bunde, String key);
```

### BackStackEntry

`getBackStackEntryAt(1)`返回一个BackStackEntry对象，这个通过这个BackStackEntry可以获取Fragment在BackStack中的一些信息

```
    backStackEntryAt.getName()//对应addToBackStack是设置的参数
    backStackEntryAt.getBreadCrumbShortTitle()//对应setBreadCrumbShortTitle(CharSequence)
    backStackEntryAt.getBreadCrumbShortTitleRes()//对应 setBreadCrumbShortTitle(StringRes)
    backStackEntryAt.getBreadCrumbTitle()//对应setBreadCrumbTitle(CharSequence)
    backStackEntryAt.getBreadCrumbTitleRes()//对应setBreadCrumbTitle(StringRes)
    backStackEntryAt.getId()//返回BackStack中Fragment唯一的id，其实就是Fragment在BackStack中索引值，比如0，1，2，这跟Fragement.getId()不是一回事
```

### 出栈

Fragment的出栈需要通过FragmentManager实现，出栈的方法如下：

```
    popBackStack();
    popBackStack(String name,int flags);
    popBackStack(int id , int flags);
    popBackStackImmediate()
    popBackStackImmediate(String name,int flags)
    popBackStackImmediate(int id , int flags)
```

方法很多，一个一个来介绍：

- popBackStack表示栈顶的Fragment出栈，但是我们知道Fragment的操作都是基于事务的，所有popBackStack只是把一个出栈操作添加到操作队列的队尾，也就是不一定立马就执行了。
- popBackStack(String name,int flags);表示多个Fragmen出栈，具体出栈到哪个位置，有两个参数决定，name就是我们调用`addToBackStack`方法时设置的标识，表示出栈到对应标识为name的Fragment的位置，至于flags表示是否包含这个Fragment，如果需要包含就传`POP_BACK_STACK_INCLUSIVE`，如果不希望包含传0即可。
- popBackStack(int id , int flags)与popBackStack(String name,int flags)一样，只是参数name变为id了，这个id就是对应BackStack中Fragment的索引。如果id传0则表示所有的fragment都出栈。

与popBackStack对应的还有一系列popBackStackImmediate方法，popBackStackImmediate系类方法表示多个Fragment立即出栈，而不是把出栈操作添加在操作队列的队尾。

>如果有多个Fragment栈，并且在出栈后需要立即添加新的Fragment，则推荐使用popBackStackImmediate方法。

### isDestory

利用FragmentManager的 `isDestroyed()` 方法可以判断对应的Activity是否被已经destory了。

---
## 3 Fragment BackStack需要注意的地方

### getFragments

getFragments返回有FragmentManager管理的Fragment列表，不要以为这个List中的一定都是Fragment，其实它很有可能有很多null值的。

比如我把四个Fragment添加的backStack中，然后让两个Fragment出栈，再去调用getFragments，这时就getFragments返回的一个size为4的list，其中有两个null值哦！

```
      List<Fragment> fragments = mFragmentManager.getFragments();
            if (!Checker.isEmpty(fragments)) {
                for (Fragment fragment : fragments) {
                    Log.d(TAG, "fragment->" + fragment);
                }
            }
    
            int backStackEntryCount = mFragmentManager.getBackStackEntryCount();
            for (int i = 0; i < backStackEntryCount; i++) {
                FragmentManager.BackStackEntry backStackEntryAt = mFragmentManager.getBackStackEntryAt(i);
                Log.d(TAG, "backStackEntryAt-id:" + backStackEntryAt.getId());
            }
```

打印的信息为：

```
    fragment->Opt1Fragment
    fragment->Opt2Fragment
    fragment->null
    fragment->null

    backStackEntryAt-id:0
    backStackEntryAt-id:1
```

所以一定要注意了做遍历的时候加上空判断。

### 2 remove 无法出栈

- backStack中的Fragment无法使用remove来出栈，此时remove方法只是销毁栈中Fragment的视图，调用生命周期方法如下：
```
    Opt1Fragment-->onPause
    Opt1Fragment-->onStop
    Opt1Fragment-->onDestroyView
```

- 被replace的Fragment(非backStack中)不会被remove，而是被销毁视图，当Fragment出栈后，被replace的Fragment会自动重建视图。

- add一个Fragment到backStack中，然后多次replace同一个Fragment，不会出错，被repalce了多少次，就可以响应多少次onBackPress