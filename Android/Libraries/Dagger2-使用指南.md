# Dagger2学习

---
## 1 依赖注入与Dagger2

Dagger2是一个依赖注入框架，在学习Dagger2前先来了解一下什么是依赖注入。依赖注入首先肯定有一种依赖关系，比如士兵开火需要使用枪来射击。好吧，假如有一个士兵他可开枪射击，这时他肯定需要一把枪：

```
    public class Soldiers {
    
        private HandGun mHandGun;
    
        public Soldiers() {
            mHandGun = new HandGun();
        }
    
        public void fire() {
            mHandGun.shoot();
        }
    
    }
```

但是上面这个程序必然是有问题的，因为Soldiers可以使用很多种枪，不仅仅是手枪，而且Soldiers不是制造手枪的，所以他也不应该知道手枪的创建过程。
这时候我们就有了注入之说，提供一个方法，来给Soldiers提供枪，而Soldiers只需要用枪射击即可。

注入的方式有很多种，比如说**构造器注入**，**setter注入**，**注解注入**
这里肯定不能使用构造器注入，因为Soldiers的独立存在不依赖于枪，

```
    public class Soldiers {
    
        private Gun mGun;
    
        public void setGun(Gun gun) {
            mGun = gun;
        }
    
        public void fire() {
            if (mGun != null)
                mGun.shoot();
        }
    
    }
    
其实注入是很好理解的。然后呢还有更厉害的注解注入：

    public class Soldiers {
    
        @Inject
        private Gun mGun;
        public void setGun(Gun gun) {
            xxx.inject(this);
        }
    
        public void fire() {
            if (mGun != null)
                mGun.shoot();
        }
    }
```

通过inject实现注入，这样连setter都省了。

Dagger2是使用生成代码实现完整依赖注入的框架，极大减少了使用者的编码负担，Dagger2由google维护，更多的资料可以自行google。

在学习Dagger2前需要配置好Dagger2需要的依赖：

```groovy
    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:2.0.0-beta6'
            classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
        }
    }
    ......
    apply plugin: 'com.android.application'
    apply plugin: 'com.neenbedankt.android-apt'
    ......
    dependencies {
        def daggerVersion = "2.5"
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile "com.google.dagger:dagger:${daggerVersion}"
        apt "com.google.dagger:dagger-compiler:${daggerVersion}"
    }
```

---
## 2 Dagger2的注解

首先来了解一下Dagger2的注解：


| 名称  | 作用  |
| ------------ | ------------ |
| `@Inject`  |  通常在需要依赖的地方使用这个注解。比如给一个字段加上`@Inject`注解，就表示该字段将被注入|
|`@Module`|Modules类里面的方法专门提供依赖，定义一个类，用@Module注解，这样Dagger在构造类的实例的时候，就知道从哪里去找到需要的依赖|
|`@Provide`|在Modules中，定义的方法加上这个注解，以此来告诉Dagger我们想要构造对象并提供这些依赖。|
|`@Component`|Components就是一个注入器，是@Inject和@Module的桥梁|
|`@Scope`|Dagger2可以通过自定义注解限定注解作用域，与注入对象的生命周期有关|
|`@Named`|当同一个注入器有多个返回相同类型的方法时，使用`@Named`来区分不同依赖的注入|
|`@Qualifier`|与Named相似，当类的类型不足以鉴别一个依赖的时候，我们就可以使用这个注解标示，更加强大|

实现一次注入，需要三个元素：

- modules：Modules类里面的方法专门提供依赖
- 注入器：component
- 使用依赖注入的容器

基本的使用规则：

- Module上必须使用`@Module`注解，注明本类属于Module，Module里面的方法必须使用@Provides注解，注明该方法是用来提供依赖对象的特殊方法
- Component上必须使用`@Component`注解， 指明Component在哪些Module中查找依赖，component必须声明为接口，最后提供注入方法
- 如果在使用注入的容器中声明了需要inject的依赖，而用来注入的Component有提供对此容器进行注入的方法，如果所指定的Module中没有提供返回容器需要的依赖的使用`@Providers`注解的方法，则编译无法通过

编写好代码之后，使用AndroidStudio的`make app`命令，即可生成代码，比如为定义的Component生成具体实现，其名称为：DaggerYourComponentName

### Dagger 示例

```
    //Module
    @dagger.Module
    public class PeopleModule {

        @Provides
        People getPeople() {
            return new ChinesePeople();
        }
    }

    //注入器
    @Component(modules = {PeopleModule.class})//指定在哪些module中查找依赖
    public interface PeopleComponent {
        void inject(DaggerFragment daggerFragment);//用于注入，只能有一个参数
    }

    //使用依赖注入的容器
    public class DaggerFragment extends Fragment {

        @Inject
        People mPeople;

         @Override
        public void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            DaggerPeopleComponent.builder().build().inject(this);
        }

        @OnClick(R.id.frag_dagger_people_btn)
        public void showPeople(View view) {
            mPeople.showInfo();
         }
    }
```

---
## 3 Dagger2注入规则

### 如果 Providers 方法需要参数

Module中`@Provides`方法可以带输入参数，这个参数可以有以下方式提供：

- 1：其参数由Module集合中的其他`@Provides`方法提供
```
    //在module中
    @providers
    ObjectA objcetA(ObjectB b){
       return new ObjectA(b);
    }

    @providers
    ObjectB objcetA(){
       return new ObjectB();
    }
```
- 2：如果Module集合中的其他`@Provides`方法没有提供，但是所依赖的参数类型有带`@Inject`注解的构造方法，而自动调用它的构造方法创建。
- 3：如果都没有的话，只能显式的提供方法所需要的参数

**注意是 Module 集合而不是单个 Module，因为 component 可以指定多个 Module**，也就是说多个 Module 可以互补

### 添加多个 Module 与 Module 实例的创建

一个Component可以包含多个Module，这样Component获取依赖时候会自动从多个Module中查找获取，Module间不能有重复方法。添加多个Module有两种方法：

- 1：直接在Component上指定

```
    @Component(modules={ModuleA.class,ModuleB.class,ModuleC.class}) //添加多个Module
    public interface XXComponent{
        ...
    }
```

- 2:在一个新的module中，include多个子module

```
         @Module(includes={ModuleA.class,ModuleB.class,ModuleC.class})
         public class XXModule{
             ...
         }
```

Component的创建有两种方法:

- `DaggerXXX.create();`等价于`DaggerXXX.builder().build();`
- 利用builder模式，可以传入特定的module，而且当Component需要的module没有无参的构造方法时，必须使用buidler模式传入。
```
        mPeopleComponent = DaggerPeopleComponent.builder()
                    .peopleModule(new PeopleModule(
                            new NameFactory()
                    )).build();
```

### 用Named或Qualifier来区分返回类型相同的`@Provides`方法

如果Component指定的Module集合中有多个返回相同类型的@Providers方法，则需要使用Named或Qualifier来对这些方法进行区分：

#### Named

```
    //Module中
          @Named("A")
          @Provides
          People getPeople() {
             return new ChinesePeople();
          }
          @Named("B")
          @Provides
          People getPeopleB() {
          return new JapanPeople();
          }

    //容器中
             @Named("A")
             @Inject
             People mPeople;

    //对于容器中mPeople调用使用了相同@Named的注解的Provides方法来创建依赖。
```

#### Qualifier

Qualifier更加强大，允许我们自定义，注意格式

```
    //实现一个用int类型区分的IntPeopleNamed
    @Qualifier//元注解
    @Documented//规范要求
    @Retention(RetentionPolicy.RUNTIME)//规范要求
    public @interface IntPeopleNamed {
        int value();
    }
    //然后再Module中使用

       @IntPeopleNamed(1)
        @Provides
        People getPeople() {
            return new ChinesePeople(mNameFactory.createName(), mRandom.nextInt(100));
        }

        @IntPeopleNamed(2)
        @Provides
        People getPeopleB() {
            return new ChinesePeople(mNameFactory.createName(), mRandom.nextInt(100));
        }
    //在容器中定义区分
        @IntPeopleNamed(1)
        @Inject
        People mPeople;
        @IntPeopleNamed(2)
        @Inject
        People mPeopleB;
```

### Component中方法定义规则

#### 1 Component 一般定义一个方法，用来提供注入，必须有需要注入的容器作为参数

```
    @Component(modules = {PeopleModule.class})
    public interface PeopleComponent {
        void inject(DaggerFragment daggerFragment);
    }
```

#### 2 如果Component中定义了没有参数的方法，则方法必须有返回值

Component也可以代替Module中的`@Provides`方法来提供依赖，而不需要`@Provides`注解，对于Component中方法的返回值，需要在指定的Moudle集合中提供返回这个类型的`@Provides`方法。但是如果返回值的类型有带有inject注解的构造函数，则会调用这个构造函数返回对象,这时指定的Moudle集合中可以没有返回这个类型值的`@Provides`方法。

#### 3 Component 可以依赖另一个 Component

- 如果 ComponentA 依赖于 ComponentB，ComponentB 必须定义带返回值的方法来提供 ComponentA 缺少的依赖，也就是说 ComponentB 要提供 ComponentA 中缺少的依赖，则必须声明依赖类型返回值的方法。

```
      //假如PeopleComponent中缺少对String的依赖
        @Component(dependencies = JapanComponent.class ,modules = PeopleModule.class)
        public interface PeopleComponent {
            void inject(DaggerFragment daggerFragment);
      }
        //JapanModule有返回String的@Provides方法，如果JapanComponent要提供依赖给PeopleComponent，则在JapanComponent中必须暴露方法提供依赖
        @Component(modules = JapanModule.class)
        public interface JapanComponent {
            String getName();
        }
```

这时也只能用 Builder 的方式构造依赖注入器了：

```
     DaggerPeopleComponent.builder()
            .japanComponent(new JapanComponent())
            .activityModule(new ActivityModule())
            .build();
```

需要注意的是：**有依赖关系的 Component 不能使用相同的 Scope**


---
## 4 关于Inject注解

在需要注入依赖的地方使用@Inject注解
有三种inject方式：constructor,field和method injection

### constructor

1.  在Constructor加上@Inject
2.  表示Constructor的参数需要dependency
3.  这些参数可以被使用在privte或final字段

### Method

1.  在methods上加上@Inject
2.  表示method的参数需要dependency
3.  injection发生在对象被完全建立之后

### Field(在 anroid 中常用)

1.  在fields上加上@Inject
2.  field不能为private或是final，至少是缺省的访问权限
3.  injection发生在对象完全建立之后

---
## 5 Scope 控制注入对象的声明周期

首先要明白Scope的作用范围是同一个Component实例上。

创建某些对象有时候是耗时浪费资源或者没有完全必要的，比如说我们的Android中，有很多的东西是全局的比如：

```
      Context//全局的上下文：ApplicationContext
      ThreadExecutor threadExecutor();//全局的子线程调度器
      PostExecutionThread postExecutionThread();//全局主线程转发器
```

而这些全局的对象需要在多个地方实现注入，ActivitA，ActivitB，FragmentA......等，但是Dagger2默认是对于每一次注入都会创建一些新的依赖对象，这时如果要控制依赖对象的创建，可以使用Scope：

### step 1

在Application中初始化一个全局的注入器ApplicationComponent，暴露方法提供ApplicationComponent给其他模块。

```
    public class AppContext extends Application {
        private static AppContext appContext;
        private AppComponent mAppComponent;
        @Override
        public void onCreate() {
            super.onCreate();
            appContext = this;
            mAppComponent = DaggerAppComponent.create();
        }
        public static AppContext get() {
            return appContext;
        }
        public AppComponent getAppComponent() {
            return mAppComponent;
        }
    }
```

### step 2

给ApplicationComponent定义一个语义上的全局Scope

```
    @Scope
    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    public @interface AppScope {
    }
```

### step 3

使用AppScope

- 第一步在AppComponent使用@AppScope注解

```
    //在AppComponent使用@AppScope注解
    @AppScope
    @Component(modules = AppModule.class)
    public interface AppComponent {
        void inject(BaseActivity activity);
    }
```

- 在被注入的依赖上使用，这里由分为两种方式：

直接在需要注入的类上声明scope注解

```
    @AppScope
    public class JobExecutor implements Executor{
        @Inject
        public JobExecutor() {
        }
        public void exe(Runnable runnable) {
           //......
        }
    }
```

在Module中的@provides方法上使用

```
    @Module
    public class AppModule {

        @Provides
        @AppScope
        Job jobExecutor(JobExecutor jobExecutor) {
            return  jobExecutor;
        }
    }
```

### step 4

使用AppContenxt获取全局的依赖注入器，进行注入

```
    public class BaseActivity extends AppCompatActivity {
        @Inject
        Executor mJobExecutor;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            AppContext.get().getAppComponent().inject(this);
        }
    }
```

这样虽然每个继承了BaseActivity的子Activity都会调用到BaseActivity的onCreate方法，注入Executor对象，但是由于在使用了Scope注解，所以对于同一个Component实例多次注入只会生成一个依赖对象，那么是不是所注入的依赖对象的生命周期是不是与它的Component实例一样呢。应该就是这样的

### 对 scope 的理解

同一个component实例中，对于提供的依赖，如果使用了scope注解，则这个依赖只会被生成一次，这个提供的依赖的生命周期与component保存一致，以后每次使用这个component进行注入，都只会返回同一个依赖对象，而没有使用scope的，则使用同一个component注入的话，每一次注入都会生成新的依赖。

至于 Singleton
```
    @Scope
    @Documented
    @Retention(RUNTIME)
    public @interface Singleton {}
```

它与我们定义的AppScope没有两样，所以名称只是语义上的区别而已，允许我们自定以scope只是为了在名称上区分各种依赖的声明周期，比如ActivityScope，FragmentScope等等，而不管是什么样名称的scope，它们作用就是让依赖于注入器的生命周期保持一致。

我们可以利用全局的component实现单例，但是这个单利只是用起来像单例，并不是真正的单例，对象保存在全局的component中，而不是静态区中。

---
## 6 Subcomponent

一个项目往往是比较复杂的，可以分为多个模块，可能每个模块的组件都有各自的依赖，有些依赖是全局公用的，有些依赖又是模块内私有的，这就需要定义多个 Component 了，同时不同的 Component 之间可能还存在交互，比如某个模块的 Component 依赖全局的 Component，有两种方式可让 Component 之间进行交互：

- 一种办法是使用 Component 上的 dependencies 属性：`Component(dependencies=××.classs)`，上面已经介绍。
- 另外一种是使用 `@Subcomponent` ，Subcomponent 用于拓展原有 Component。让不同 Component 之间有了父子关系，一般是子 Component 依赖父 Component。

### Subcomponent 的特点

- 通过 Subcomponent 功能，子 Component 可以直接使用父 Component 中提供的依赖，而父 Component 中不再需要定义相关依赖类型的返回方法，只需要添加返回子 Component的方法。
- 通过 Subcomponent 功能，子 Component 同时拥有两种 Scope，当注入的元素来自父Component，则这些元素的生命周期由父 Component 决定，当注入的元素来自子 Component，则这些元素的生命周期由子 Component 决定。

### 从 ParentComponent 获取 ChildComponent 的两种方式

- 1 直接在 ParentComponent 中定义返回ChildComponent的方法

```
    
//1 定义子 Component
    @SubScrope
    @Subcomponent(modules=××××)
    public SubComponent{ void inject(SomeActivity activity); }

//2 父 Component 中声明返回子Component
    @ParentScrope
    @Component(modules=××××)
    public interface ParentComponent{
         //如果ChildComponent需要显式地添加Module时，定义在方法参数中
         ChildComponent getChildComponent(ChildModule childModule);
    }

//3 使用
    AppContext.getParentComponent()
        .getChildComponent(new ChildModule())
        .inject(this);

```

- 2 使用 Builder 模式

```
    @Subcomponent(modules = RequestModule.class)
    interface RequestComponent {

      RequestHandler requestHandler();

      @Subcomponent.Builder
      interface Builder {
        Builder requestModule(RequestModule module);
        RequestComponent build();
      }

    }

    //ParentComponent 返回 ChildComponent 的 Builder
    public interface ParentComponent{
           ChildComponent.Builder getChildComponentBuilder();
    }
```

---
## 7 Reusable 和 CanReleaseReferences

- `@Reusable`也是一种Scrope注解，Reusable表示可重用的，`@Reusable`注解返回的对象绑定可能是(但也可能不是)重用，其可用于限制规定类型返回的数量。

- `@CanReleaseReferences`表示可以被释放引用的，配合ForReleasableReferences和ReleasableReferenceManager使用，用于优化内存

---
## 8  Lazy 与 Provider

Lazy 和 Provider都是用于包装 Container 中需要被注入的类型，Lazy 用于延迟加载，Provide 用于强制重新加载：

```
    public class Container{
        @Inject Lazy<PeopleA> mPeopleA; //注入Lazy元素
        @Inject Provider<PeopleB> mPeopleB; //注入Provider元素
        public void init(){
            DaggerComponent.create().inject(this);
            PeopleA pA=mPeopleA.get();  //在这时才创建f1,以后每次调用get会得到同一个f1对象
            PeopleB pB=mPeopleB.get(); //在这时创建f2，以后每次调用get会再强制调用Module的Provides方法一次，根据Provides方法具体实现的不同，可能返回跟f2是同一个对象，也可能不是。
        }
    }
```

值得注意的是，Provider 保证每次重新加载，但是并不意味着每次返回的对象都是不同的。只有Module的Provide方法每次都创建新实例时，Provider 每次 `get()` 的对象才不相同。

---
## 9 Multibindings

Dagger允许将多个对象绑定到集合中，即使使用mutlbindings将对象绑定到不同的模块中也是如此。Dagger组合集合，以便应用程序代码可以注入它而不依赖于单独的绑定。可以使用多绑定来实现插件架构，Multibindings分为set和map。

### 9.1 Set multibindings

关键注解：`@IntoSet, @ElementsIntoSet`

定义Module

```
    @Module
    class MyModuleA {
      @Provides
      @IntoSet//此时ABC将作为Set的元素返回
      static String provideOneString(DepA depA, DepB depB) {
        return "ABC";
      }
    }

    @Module
    class MyModuleB {
      @Provides
      @ElementsIntoSet//此时Set<String>将被添加到新的Set中返回
      static Set<String> provideSomeStrings(DepA depA, DepB depB) {
        return new HashSet<String>(Arrays.asList("DEF", "GHI"));
      }
    }
```

绑定这两个Module来注入Set：

```
    class Bar {
      @Inject Bar(Set<String> strings) {
        assert strings.contains("ABC");
        assert strings.contains("DEF");
        assert strings.contains("GHI");
      }
    }
```

### 9.2 Map multibindings

关键注解：`@MapKey, @IntoMap, @LongKey, @StringKey`等等

Map multibindings支持以下绑定方式：

- 注入Map
- 自定义MapKey
- 使用复合的Key，配合`@AutoAnnotation`自动生成符合Key
- Inherited subcomponent multibindings

具体可以参考：

- [multibindings 文档](https://google.github.io/dagger/multibindings)
- [Activities Subcomponents Multibinding in Dagger 2](http://frogermcs.github.io/activities-multibinding-in-dagger-2/)
- [Dagger2Recipes-ActivitiesMultibinding](https://github.com/frogermcs/Dagger2Recipes-ActivitiesMultibinding)
- [Dagger2-MultiBinding-Android-Example](https://github.com/trevjonez/Dagger2-MultiBinding-Android-Example)


---
## 10 Dagger2 安卓拓展

**Dagger Android**是对Android平台的扩展。提供了一种低耦合的对 Android 四大组件以及 Fragmengt 的注入方式。其内容原理是使用的 `Map multibindings` 。

- `ContributesAndroidInjector`：这是一个非常强大的注解，可以为注入目标生成 Component， 不过只能用于 Android 中的组件(比如Activity、Service、Fragment等)。
- AndroidInjector 支持扩展以对其他组件进行注入，具体参考[如何扩展](https://github.com/Ztiany/Programming-Notes/blob/master/Android/Dagger2AndroidInjection/README.md)

---
## 11 注入nullable对象

有时候需要注入的对象是可null的，默认Dagger2对对注入的对象进行检测，如果是null的，则会抛出异常，标注注入的对象可null有两种方式：

### 使用Nullable

Module中是使用`javax.annotation.Nullable`注解：

```
@Module
public class ServiceModule {

    @Provides
    @javax.annotation.Nullable
    AddressService provideAddressService(AppRouter appRouter) {
        return appRouter.navigation(AddressService.class);
    }

}
```

在被注入对象中也要标注该注入的对象可null

```
public class ShoppingCartRepository   {

    @Inject
    ShoppingCartRepository(ShoppingCartApi shoppingCartApi, @javax.annotation.Nullable AddressService addressService) {
        mShoppingCartApi = shoppingCartApi;
        mAddressService = addressService;
    }
}
```

### 使用BindsOptionalOf

BindsOptionalO提供注入对象的方法上，表示此注入对象可能不存在，即可能不会被注入。使用方式：

```
//step 1
@Module
public interface ApiServiceModule {
    @BindsOptionalOf ApiService bindApiServiceOptional();
}

//step 2 针对不同的Feature定义不同的FeatureModule
@Module(includes = ApiServiceModule.class)
public interface Feature1Module {
    @Binds
    Logger bindLogger(Feature1Logger feature1Logger);
}

@Module(includes = ApiServiceModule.class)
public abstract class Feature2Module {

}

//step 3 在被注入对象中使用Optional声明依赖
Optional<Foo>
Optional<Provider<Foo>>
Optional<Lazy<Foo>>
Optional<Provider<Lazy<Foo>>>
```

Optional支持`com.google.common.base.Optional` 和 `java.util.Optional`。

具体参考：

- [BindsOptionalOf文档](https://google.github.io/dagger/api/latest/dagger/BindsOptionalOf.html)
- [Avoid Nullable dependencies in Dagger2 with @BindsOptionalOf](https://medium.com/@birajdpatel/avoid-nullable-dependencies-in-dagger2-with-bindsoptionalof-c3ad8a8fde2c)

---
## 12 使用 BindsInstance

标记Component构建器或SubComponent构建器上的方法，该方法允许将实例绑定到组件中的某种类型。示例：

```
//定义一个Component，并为其定义一个构建器
@Component(modules = AppModule.class)
interface AppComponent {

  App app();

  @Component.Builder
  interface Builder {

    //在构建器上可以使用BindsInstance来绑定自定义数据，@UserName是自定义的标识
    //然后String类型的userName可以为后面的注入构建所用
    @BindsInstance Builder userName(@UserName String userName);
    AppComponent build();

  }
}

//使用构建器来构建Component，此时可以传入userName
  App app = DaggerAppComponent
      .builder()
      .userName(name)
      .build()
      .app();

  app.run();
```

---
## 13 Producers

Producers是Dagger2的拓展，原有的注入方式都是同步的，Producers模块提供了异步注入的方式。具体参考[文档](https://google.github.io/dagger/producers)。


---
## 14 泛型支持

泛型可以应用于Dagger2中，在基类中定义泛型，然后在具体的子类确定实际参数类型，Dagger2依然可以提供正确的注入：

```java
//Activity
public abstract class BaseActivity<P extends IPresenter> extends AppCompatActivity{

    @Inject
    protected P mPresenter;

}


public class UserActivity extends BaseActivity<UserPresenter>{
    ...
}

public class UserPresenter implement IPresenter{
    
    @Inject
    public UserPresenter(){
        
    }
}
```

---
## 15 Dagger 2.17 更新

Dagger 2.17 的更新可能会引发一些编译错误，官方解释原文为：If you start seeing missing binding errors in this release, check out [this wiki page](https://github.com/google/dagger/wiki/Dagger-2.17-@Binds-bugs) for information on how to debug the issues


---
## 引用

- [Android常用开源工具（1）-Dagger2入门](http://blog.csdn.net/duo2005duo/article/details/50618171)
- [Android常用开源工具（2）-Dagger2进阶](http://blog.csdn.net/duo2005duo/article/details/50696166)
- [Users-Guide](https://google.github.io/dagger/users-guide)
- [让你的Daggers保持锋利](https://juejin.im/entry/5b8252d5e51d4538cd22834c)