# ActivityLifecycleCallbacks、FragmentLifecycleCallbacks、Activity 与 Fragment 生命周期交互

---
## 1 Activity 与 Fragment 生命周期交互

平时做项目一般都会封装一个基类的Activity，比如下面方式：

```
    public abstract class BaseActivity extends AbsActivity {
    
        @Override
        protected void onCreate(@Nullable Bundle savedInstanceState) {
            //调用父类的方法
            super.onCreate(savedInstanceState);
            //然后封装自己的逻辑
            initialize(savedInstanceState);
            int layoutId = layoutId();
            setContentView(layoutId);
            ButterKnife.bind(this);
            setupView(savedInstanceState);
        }
```

一般情况下这样没有什么问题的。但是很多时候Activity中需要和Fragment交互，而有些时候Fragment又需要Activity提供一些依赖，但是Fragment中总不能直接`getActivity()`强转成具体的某一个Activity吧，这样的Fragment就只能和此Activity交互了，耦合性太高了，所以一般我们会在Fragmeg中写一个接口，让需要使用此Fragment的Activity实现此接口，然后Fragment就只需要通过它提供的接口来获取依赖就可以了，但这不是这里的重点。

**重点是，在Activity中什么时候初始化Fragment需要的依赖。**

有时候Fragment需要在onCreate中就获取Activit提供的依赖，比如我们在使用Dagger2时经常这么做：

```
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
    
            InjectHelper.getActivityComponent(this, MainComponent.class)
                    .userCenterComponent().inject(this);
            mPresenter.bindView(this);
    
        }

        public static <C> C getActivityComponent(Fragment fragment, Class<C> tClass) {
            return tClass.cast(  ((HasComponent<C>)fragment.getActivity()).getComponent() );
        }
```

因为不希望重复的注入，所以在onCreate方法中获取Activity提供的Component()进行注入，而我们也习惯贼Activity封装的initialize方法中初始化容器。这样做一般情况下确实没有什么问题，但是当使用屏幕旋转来模拟Activity后系统回收后重建的情况下，可能就会引发异常，空指针异常。原因在于Activity生命周期方法回调在重建时与一般情况下有一点不一样。

正常情况下的生命周期调用
```
    MainActivity---->onCreate after call super  bundle = null
    MainActivity---->onStart
    MainActivity---->onResume

    MallFragment-->onAttach
    MallFragment-->onCreate  savedInstanceState   =   null
    MallFragment-->onViewCreated  savedInstanceState   =   null
    MallFragment-->onActivityCreated savedInstanceState   =   null
    MallFragment-->onStart

    MainActivity---->onStart
    MainActivity---->onResume

    MallFragment-->onResume
```

界面销毁
```
    MallFragment-->onPause
    MainActivity---->onPause
    MallFragment-->onStop
    MainActivity---->onStop
    MallFragment-->onDestroyView
    MallFragment-->onDestroy
    MallFragment-->onDetach
    MainActivity---->onDestroy
```

重建时的生命周期调用：
```
    MallFragment-->onAttach
    MallFragment-->onCreate

    MainActivity---->onCreate

    MallFragment-->onViewCreated
    MallFragment-->onActivityCreated
    MallFragment-->onStart
    MainActivity---->onStart
    MainActivity---->onResume
    MallListFragment-->onResume
```

看到这里，我们下意识的就会认为**在重建的时候，Activity 的onCreate方法原来在Fragment之后执行，那么显然就不是在Activity 的onCreate方法中初始化Fragmeng的依赖了**，但是事实并不是如何，因为我们平时的编码习惯关系，一般我们打印Activity生命周期方法都会这样：

```
    protected void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            if (printLifeCycle) {
                Log.d(tag(), tag() + "---->onCreate after call super  " + "bundle = " + savedInstanceState);
            }
        }
```

但是有没有想过这样呢，有没有考虑过调用 super.onCreate() 时都做了写什么呢？

```
    protected void onCreate(@Nullable Bundle savedInstanceState) {
            if (printLifeCycle) {
                Log.d(tag(), tag() + "---->onCreate before call super    ");
            }
            super.onCreate(savedInstanceState);
            if (printLifeCycle) {
                Log.d(tag(), tag() + "---->onCreate after call super  " + "bundle = " + savedInstanceState);
            }
        }
```

再来看一下log：
```
    MainActivity---->onCreate before call super
    MainActivity---->onCreate after call super
    
    MainActivity---->onStart before call super
    
    MallFragment-->onAttach
    MallFragment-->onCreate
    MallFragment-->onViewCreated
    MallFragment-->onActivityCreated
    MallFragment-->onStart
    
    MainActivity---->onStart
    MainActivity---->onResume before call super
    MainActivity---->onResume
    
    MallFragment-->onResume
```

界面销毁：
```
    MainActivity---->onPause before call super
    
    MallFragment-->onPause
    MainActivity---->onPause
    
    MainActivity---->onStop before call super
    
    MallFragment-->onStop
    
    MainActivity---->onStop
    MainActivity---->onDestroy before call super
    
    MallFragment-->onDestroyView
    MallFragment-->onDestroy
    MallFragment-->onDetach
    
    MainActivity---->onDestroy
```

重建时的生命周期调用：
```
    MainActivity---->onCreate before call super//super之前
    
    MallFragment-->onAttach
    
    MainActivity---->onCreate after call super//super之后
    
    
    MainActivity---->onStart//super之后
    
    MallFragment-->onStart
    
    MainActivity---->onStart before call super//super之前
    
    MallFragment-->onViewCreated
    MallFragment-->onActivityCreated

    MainActivity---->onResume before call super
    MainActivity---->onResume
    MallFragment-->onResume
```

看到生命周期方法的调用之后，发现还时可以在Activity的onCreate方法中初始化Fragment需要的依赖的，因为Activity的onCreate方法必然在Fragment的onCreate方法之前执行，只是需要注意`super.onCreate`方法，当界面重建时，Fragment的onCreate方法在其后面执行。于是把代码修改成如下方式即可：

```
     public abstract class BaseActivity extends AbsActivity {
    
            @Override
            protected void onCreate(@Nullable Bundle savedInstanceState) {
    
                //初始化一下必要的对象
                initialize(savedInstanceState);
                //调用父类的方法
                super.onCreate(savedInstanceState);
    
                //然后才是逻辑
                int layoutId = layoutId();
                setContentView(layoutId);
                ButterKnife.bind(this);
                setupView(savedInstanceState);
            }
```

另外**在Fragment中嵌套Fragment的情况也是类似的**

---
## 2 ActivityLifecycleCallbacks、FragmentLifecycleCallbacks

>补充于 dagger2 android 扩展

dagger2 android 扩展让注入不必那么麻烦了，可以使用 ActivityLifecycleCallbacks、FragmentLifecycleCallbacks实现自动注入，而 Activity 和 Fragment 只需要实现相关的标记接口即可：

```kotlin
interface Injectable

fun Application.applyAutoInjector() = registerActivityLifecycleCallbacks(
        object : Application.ActivityLifecycleCallbacks {

            override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {
                Log.d("Injectable", "Application onActivityCreated  $activity")
                handleActivity(activity)
            }

            override fun onActivityStarted(activity: Activity) {

            }

            override fun onActivityResumed(activity: Activity) {

            }

            override fun onActivityPaused(activity: Activity) {

            }

            override fun onActivityStopped(activity: Activity) {

            }

            override fun onActivitySaveInstanceState(activity: Activity, outState: Bundle?) {

            }

            override fun onActivityDestroyed(activity: Activity) {

            }
        })

private fun handleActivity(activity: Activity) {
    if (activity is Injectable || activity is HasSupportFragmentInjector) {
        AndroidInjection.inject(activity)
        Log.d("Injectable", "Application inject activity-> $activity")
    }
    if (activity is FragmentActivity) {
        activity.supportFragmentManager.registerFragmentLifecycleCallbacks(
                object : FragmentManager.FragmentLifecycleCallbacks() {

                    override fun onFragmentAttached(fm: FragmentManager?, f: Fragment?, context: Context?) {
                        Log.d("Injectable", "Application onFragmentAttached $f")
                        if (f is Injectable) {
                            Log.d("Injectable", "Application inject fragment-> $f")
                            AndroidSupportInjection.inject(f)
                        }
                    }

                    override fun onFragmentCreated(fm: FragmentManager, f: Fragment, s: Bundle?) {
                        Log.d("Injectable", "Application onFragmentCreated $f")
                    }
                }, true)
    }
}
```
Activity 和 Fragment：

```kotlin
class MainActivity : AppCompatActivity(), HasSupportFragmentInjector {

    @Inject
    lateinit var androidInjector: DispatchingAndroidInjector<Fragment>

    override fun supportFragmentInjector() = androidInjector

    override fun onCreate(savedInstanceState: Bundle?) {
        Log.d("Injectable", "this on Activity Create before $this")
        super.onCreate(savedInstanceState)
        Log.d("Injectable", "this on Activity Create after $this")
        setContentView(R.layout.main_activity)
        setupFragment(R.id.containerLayout) { MainFragment.newInstance() }
    }
}

class MainFragment : Fragment(), Injectable {

  @Inject lateinit var userViewModel: UserViewModel

  override fun onAttach(context: Context?) {
    Log.d("Injectable", "this on Fragment onAttach before $this")
    super.onAttach(context)
    Log.d("Injectable", "this on Fragment onAttach after $this")

  }

  override fun onCreate(savedInstanceState: Bundle?) {
    Log.d("Injectable", "this on Fragment onCreate before $this")
    super.onCreate(savedInstanceState)
    Log.d("Injectable", "this on Fragment onCreate before $this")
  }
  
  companion object {
    fun newInstance() = MainFragment()
  }
}
```

这样子在 Activity 重建时有没有问题呢？

```
//重新流程
this on Activity Create before MainActivity@80485c3
Application onActivityCreated  MainActivity@80485c3
Application inject activity-> MainActivity@80485c3 // activity 注入
this on Activity Create after MainActivity@80485c3
this on Fragment onAttach before MainFragment{4f40770 #0 id=0x7f080030}
this on Fragment onAttach after MainFragment{4f40770 #0 id=0x7f080030}
Application onFragmentAttached MainFragment{4f40770 #0 id=0x7f080030}
Application inject fragment-> MainFragment{4f40770 #0 id=0x7f080030} // fragment 注入
this on Fragment onCreate before MainFragment{4f40770 #0 id=0x7f080030}
this on Fragment onCreate before MainFragment{4f40770 #0 id=0x7f080030}
Application onFragmentCreated MainFragment{4f40770 #0 id=0x7f080030}

//重建流程
this on Activity Create before MainActivity@bb1bbd4
Application onActivityCreated  MainActivity@bb1bbd4
Application inject activity-> MainActivity@bb1bbd4 // activity 注入
this on Fragment onAttach before MainFragment{bdfc17d #0 id=0x7f080030}
this on Fragment onAttach after MainFragment{bdfc17d #0 id=0x7f080030}
Application onFragmentAttached MainFragment{bdfc17d #0 id=0x7f080030}
Application inject fragment-> MainFragment{bdfc17d #0 id=0x7f080030}  // fragment 注入
this on Fragment onCreate before MainFragment{bdfc17d #0 id=0x7f080030}
this on Fragment onCreate before MainFragment{bdfc17d #0 id=0x7f080030}
Application onFragmentCreated MainFragment{bdfc17d #0 id=0x7f080030}
this on Activity Create after MainActivity@bb1bbd4
```

通过日志可以看出 Activity 总会先执行注入，所以不存在什么问题，只是切记在 Fragment 中，不要在 onAttach 方法内使用被注入的对象即可。