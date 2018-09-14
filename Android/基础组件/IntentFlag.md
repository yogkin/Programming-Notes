[TOC]
# Intent的Flag

Intent的Flag用于指定系统如何启动一个Activity。Intent的Flag还能和清单文件中`<activity>`标签的相关属性配合使用。

---
##  1 `<activity>`中可用的属性

###  1.1 taskAffinity

task的相关性，用于指定一个Activity更加愿意依附于哪一个任务，具有如下特定：

*   这个参数标识了一个Activity所需任务栈的名字，默认情况下，所有Activity所需的任务栈的名字为应用的包名
*   我们可以单独指定每一个Activity的taskAffinity属性覆盖默认值
*   **一个任务的affinity决定于这个任务的根activity**（root activity）的taskAffinity
*   在概念上，具有相同的affinity的activity（即设置了相同taskAffinity属性的activity）属于同一个任务
*   为一个activity的taskAffinity设置一个空字符串，表明这个activity不属于任何task
*   很重要的一点taskAffinity属性不对standard和singleTop模式有任何影响，即时你指定了该属性为其他不同的值，这两种启动模式下不会创建新的task，除非添加`FLAG_ACTIVITY_NEW_TASK`标记


示例：为Activity指定新的taskAffinity，并且添加new_task标志启动它：

```java
            <activity
                android:name=".Main2Activity"
                android:label="@string/title_activity_main2"
                android:taskAffinity="com.ztiany.m2"
                android:theme="@style/AppTheme.NoActionBar">
            </activity>
            
            Intent intent = new Intent(this, Main2Activity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            startActivity(intent);
```

此时Main2Activity运行在一个新的Task中(没有添加NEW_TASK则不会)，

只有指定了taskAffinity并且添加了`FLAG_ACTIVITY_NEW_TASK`才会在新的创建新的task

**affinity主要有以下两种应用场景**：

- 当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了一个FLAG_ACTIVITY_NEW_TASK flag的话(或者该Activity在manifest文件中声明的启动模式是"singleTask")，系统就会尝试为这个Activity单独创建一个任务。但是规则并不是只有这么简单，系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务(上面的测试也验证了这一点)。而同一个程序中所有Activity的affinity默认都是相同的，这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。


- 当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在任务的能力。具体点来说，就是一个Activity现在是处于某个任务当中的，但是它与另外一个任务具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。举一个形象点的例子，比如有一个天气预报程序，它有一个Activity是专门用于显示天气信息的，这个Activity和该天气预报程序的所有其它Activity具体相同的affinity值，并且还将allowTaskReparenting属性设置成true了。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity又会被转移到天气预报程序的任务当中，并显示出来，因为它们拥有相同的affinity值，并且将allowTaskReparenting属性设置成了true。


### 1.2 launchMode 启动模式

- 标准模式(默认)
- singleTop模式
- singleTask模式(单一实例)(比较复杂，需要注意多个task启动同一个singleTask模式的Activity的情况)
- singleInstance模式(单一实例，task中只存在次实例)

任务栈拼接情况：

![](images/diagram_backstack_singletask_multiactivity.png)

### 1.3 Task配置

在清单文件中指定Activity的任务栈配置：

```xml
          <activity
                android:name=".SampleActivity"
                android:alwaysRetainTaskState="true"/>
```

属性解释：

####  allowTaskReparenting

这个属性用于设定Activity能够从启动它的任务中转移到另一个与启动它的任务有亲缘关系的任务中。上面已经说到此属性的使用

#### clearTaskOnLaunch

这个属性用于设定在从主屏中重启任务时，处理根节点的Activity以外，任务中的其他所有的Activity是否要被删除。如果设置为true，那么任务根节点的Activity之上的所有Activity都要被清除，如果设置了false，就不会被清除。默认设置时false。这个属性只对启动新任务（或根Activity）的那些Activity有意义，任务中其他所有的Activity都会被忽略。

#### alwaysRetainTaskState

 这个属性用于设置Activity所属的任务状态是否始终由系统来维护。如果设置为true，则由系统来维护状态，设置为false，那么在某些情况下(系统回收任务栈)，系统会允许重设任务的初始状态。默认值是false。这个属性只对任务根节点的Activity有意义，其他所有的Activity都会被忽略。这个属性用来标记应用的task是否保持原来的状态，“true”表示总是保持，“false”表示不能够保证，默认为“false”。此属性只对task的根Activity起作用，其他的Activity都会被忽略。 **默认情况下，如果一个应用在后台呆的太久例如30分钟，用户从主选单再次选择该应用时，系统就会对该应用的task进行清理，除了根Activity，其他Activity都会被清除出栈**，但是如果在根Activity中设置了此属性之后，用户再次启动应用时，仍然可以看到上一次操作的界面。 这个属性对于一些应用非常有用，例如Browser应用程序，有很多状态，比如打开很多的tab，用户不想丢失这些状态，使用这个属性就极为恰当。

####  finishOnTaskLaunch

  这个属性和clearTaskOnLaunch是比较类似的，不过它不是作用于整个任务上的，而是作用于单个Activity上。如果某个Activity将这个属性设置成true，那么用户一旦离开了当前任务，再次返回时这个Activity就会被清除掉。

---
## 2 Intent当中比较常用的flag

###   FLAG_ACTIVITY_NEW_TASK

设置了这个flag，新启动Activity就会被放置到一个新的任务当中(与"singleTask"有点类似，但不完全一样)，**当然这里讨论的仍然还是启动其它程序中的Activity**。这个flag的作用通常是模拟一种Launcher的行为，即列出一推可以启动的东西，但启动的每一个Activity都是在运行在自己独立的任务当中的。

###   FLAG_ACTIVITY_SINGLE_TOP

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么就不会再次创建这个Activity的实例，而是直接调用它的onNewIntent()方法。这种flag和在launchMode中指定"singleTop"模式所实现的效果是一样的。

###  FLAG_ACTIVITY_CLEAR_TOP

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，就不会再次创建这个Activity的实例，而是会把这个Activity之上的所有Activity全部关闭掉

比如说，一个任务当中有A、B、C、D四个Activity，然后D调用了startActivity()方法来启动B，并将flag指定成FLAG_ACTIVITY_CLEAR_TOP，那么此时C和D就会被关闭掉，现在返回栈中就只剩下A和B了。

此时Activity B会接收到这个启动它的Intent，**你可以决定是让Activity B调用onNewIntent()方法(不会创建新的实例)，还是将Activity B销毁掉并重新创建实例。**：

- 如果Activity B没有在manifest中指定任何启动模式(也就是"standard"模式)，并且Intent中也没有加入一个`FLAG_ACTIVITY_SINGLE_TOP` flag，那么此时Activity B就会销毁掉，然后重新创建实例。

- 如果Activity B在manifest中指定了任何一种启动模式，或者是在Intent中加入了一个FLAG_ACTIVITY_SINGLE_TOP flag，那么就会调用Activity B的onNewIntent()方法。


查看任务栈的adb命令：`adb shell dumpsys activity activities`













