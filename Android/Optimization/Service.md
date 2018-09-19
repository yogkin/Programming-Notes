# Service

service是一个后台服务，专门用来处理常驻后台的工作的组件。

## 一、优先级

进程的重要性优先级：（越往后的就越容易被系统杀死）

1. 前台进程：Foreground process
   - 用户正在交互的Activity（onResume()）
   - 当某个Service绑定正在交互的Activity。
   - 被主动调用为前台Service（startForeground()）
   - 组件正在执行生命周期的回调（onCreate()/onStart()/onDestroy()）
   - BroadcastReceiver 正在执行onReceive();
2. 可见进程：isible process
   - 我们的Activity处在onPause()（没有进入onStop()）,比如一个Activity启动了一个对话框样式的Activity
   - 绑定到前台Activity的Service。
3. 服务进程：Service process，简单的startService()启动。
4. 后台进程：Background process，对用户没有直接影响的进程----Activity处于onStop()的时候。
5. 空进程：Empty process，不含有任何的活动的组件。（android设计的，为了第二次启动更快，采取的一个权衡）


## 二、如何提升进程的优先级（尽量做到不轻易被系统杀死）

1. 在锁屏的时候启动一个1个像素的Activity，当用户解锁以后将这个Activity结束掉（顺便同时把自己的核心服务再开启一次）。
2. app运营商和手机厂商可能有合作关系---白名单。
3. 双进程守护，可以防止单个进程杀死，同时可以防止第三方的手机管家清理掉。一个进程被杀死，另外一个进程又被他启动。相互监听启动。
4. JobScheduler，把任务加到系统调度队列中，当到达任务窗口期的时候就会执行，我们可以在这个任务里面启动我们的进程。这样可以做到将近杀不死的进程。
5. 监听QQ,微信，系统应用，友盟，小米推送等等的广播，然后把自己启动了。
6. 利用账号同步机制唤醒我们的进程AccountManager
7. NDK来解决，Native进程来实现双进程守护。








