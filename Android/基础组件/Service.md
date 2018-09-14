# service

---
## 1 服务的两种开启方式

### startService()开启服务

一旦开启，服务就会长期的后台运行，即使开启式Activity退出了，服务还是后台继续运行，直到用户手工停止服务为止

### bindService()绑定服务

如果开启者Activity退出了，服务也会跟着挂断leaked漏气，没有解除绑定Activity就退出了，服务找不到绑定的对象。解除绑定服务unBindService()服务只能被解绑一次，多次解绑就会抛异常服务应当只绑定一次，注意：如果开启了服务的同时也绑定了服务，在没有接触绑定的情况下，调用stopService是无法停止Service的。

---
## 2 服务的生命周期

### 采用startService和stopService方法去开启和停止服务

1. 服务只会被创建一次，如果服务已经被创建就不会被重新创建了，不会执行onCreate方法
2. 服务只会被停止一次，如果服务已经停止了就不会重新执行onDestroy方法
3. 如果服务已经开启，多次调用startService方法服务只会执行onStartCommand方法和onStart方法

### 采用bindService方法和unbindService方法开启和停止服务

1. bindService()如果服务不存在 onCreate--->onbind方法
bindService(intent, conn, Service.BIND_AUTO_CREATE);
2. unbindService() onunbind--->ondestroy方法 如果没有开启服务，只绑定服务，解除绑定会调用服务的onDestroy方法 会销毁服务
3. 绑定服务不会执行onstart方法和onstartCommand方法
4. 服务只可以被解绑一次，多次解绑抛异常
5. 多次调用 Context.bindService()方法并不会导致onBind方法被多次调用

### 混合方式开启服务

1. startservice 开启服务 （保证服务长期后台运行）
2. bindservice 绑定服务  （调用服务的方法）
3. unbindService 解除绑定服务 （停止调用服务的方法）
4. stopService 停止服务   （停止服务，播放器停止）




