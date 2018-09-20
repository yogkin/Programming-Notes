# 四大组件的运行状态概述

Android四大组件除了广播接收者可以不在Manifest文件中注册，其他三大组件都必须在Manifest中注册，在调用方式上除了只有ContentProvider不需要Itent，其他都需要Intent。下面简单的对四大组件进行一个分类：

### Activity

Activity是一个**展示型**组件，用于向用户直接的展示一个界面，并且可以接受用户的输入从而进行交互，对于用户来讲，Activity就是一个应用的全部，因为其他组件都是不可被感知的，所以可以是，Activity在app中扮演者一个前台角色。

### Service

Service是一个**计算型**组件，用于在后台执行一系列的任务，Service的运行状态有两种，启动状态和绑定状态，也可以通过stopService和unBindService结束调Service

### Broadcast

Broadcast是一种**消息型**组件，用于在不同的组件乃至应用中传递消息。Broadcast同样无法被用户感知，可以通过Manifest文件注册也可以通过代码注册，代码注册的Broadcast优先级较高，但是只用应用启动了才能接收到系统广播

### ContentProvider

ContentProvider是一种**数据共享型组件**，用于向其他组件或其他应用共享数据，无法被用户感知，需要注意的是，ContentProvider的curd方法都运行在binder线程池中，需要做好同步工作，虽然ContentProvider一般采用数据库对外共享数据，但是并不限于数据库，它可以以多种方式对外共享数据。