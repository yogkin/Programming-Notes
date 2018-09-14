# PendingIntent

## 1 PendingIntent概述

在RemoteView中多次使用到PendingIntent，PendingIntent是什么东西呢？

PendingIntent代表一种延期的意图，和是在将来的某个时刻发生的，而Intent是立刻发生的，也可以说Intent是直接告诉一个人叫他去做什么事，而PendingIntent是通过中间人间接的告诉一个人需要做什么事，在RemoteViews中的用法就是setOnClickPendingIntent，它不能直接设置onClickLinstener，因为它不是普通的view。


## 2 PendingIntent支持的三种待定意图

| 方法  |  说明 |
| ------------ | ------------ |
| `getActivity(Context context, int requestCode,Intent intent, @Flags int flags)`   |  意图发生时，相当于context.StartActivity()  |
| `getBroadcast(Context context, int requestCode, Intent intent, @Flags int flags)`  | 意图发生时相当于context.sendBroadcase()  |
| `getService(Context context, int requestCode, @NonNull Intent intent, @Flags int flags)`   | 意图发生时，相当于content.startService()  |


上面方法的requestCode和flags说明
- requestCode表示pendingIntent的请求码，一般情况下为0即可，requestCode的值会影响
- flags的常见类型有四种，但是在说这四个标识之前，下来说一下PendingIntent的匹配规则

### PendingIntent的匹配规则

PendingIntent的匹配规则即在什么情况下，认为两个PendingIntent是相同的：

如果两个PendingIntent的Intent相同并且RequestCode那么这两个PendingIntent就是相同的，requestCode比较好理解，那么什么情况下Intent相同呢？如果两个Intent的ComponentName和IntentFilter相同，那么这两个Intent相同，即使他们的Extras不同，

### 四个flags

#### FLAG_ONE_SHOT

当前描述的pending只能被使用一次，然后他会自动取消，后续如果还有相同的pendingIntent，那么他们的send方法就会调用失败，对于消息栏通知来说，如果采用此标志位，那么同类消息只能发送一次，后续的通知单击后无法打开。

#### FLAG_NO_CREATE

当前描述的PendingIntent不会主动创建，如果当前PendingIntent不存在，则getActivity，getBrroadcase,getService返回null，这个标志位很少用，它无法单独使用，实际开发中没有太多实际意义

#### FLAG_CANCEL_CURRENT

当前描述的PendingIntent如果已经存在，那么他们会被cancel，然后系统创建一个新的PendingIntent，对于消息栏来说，被cancel的消息点击后无法被打开

#### FLAG_UPDATE_CURRENT
当前描述的PendingIntent如果已经存在，他们会被更新，即他们的Intent的Extraxs会被替换成最新的


###  具体应用场景描述

上面四种flags应用到通知栏的情况如下：

1. 如果NotificationManager.notify(1,notification)的第一个参数id是常量，那多次调用只会弹出一个通知，，后续的通知会把前面的通知完全替换掉。
2. 如果id每次都不一样
 - 那么当pendingIntent不匹配时(requestCode和Intent不同)，那么不管采用什么标志位，PendintIntent之间都不会互相干扰
 - 如果pendingIntent匹配，采用FLAG_ONE_SHOT，后续的通知会和第一条通知保持完全一致，保存七种的Extras，单击任何一条通知，剩下的通知都无法再打开，当所有通知都被取消后，重复这个过程；采用FLAG_CANCEL_CURRENT标识，只有最新的通知才会被打开，之前的通知都无法在打开，采用FLAG_UPDATE_CURRENT那么之前的pendingintent会被更新，最终这些通知会和最新的一条通知保持完全一致，包括extras，而且他们都可被打开。





















