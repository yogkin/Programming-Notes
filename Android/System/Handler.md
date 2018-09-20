# Handler 消息机制

---
## 1 概述

一般的操作系统中都存在消息机制，发生的一切都可以用消息来表示，消息用于告诉操作系统发生了什么，Android系统也有自己的消息机制，而Handler就是Android消息机制的上层接口，一般Handler用于在子线程发生消息到主线程更新ui，但Handler并不只是来更新ui的，它的本质是线程间通信，也可以说将一个任务切换到指定的线程去执行。很显然，我们平时使用到Handler大都还是用于更新UI，那么为什么只能在主线程更新ui呢？为什么不设计成子线程也可以更新UI呢？

**问题1：** 我们都知道，View树的遍历都是通过ViewRootImpl发起的，而每次都遍历的发起都会执行一个方法：

```java
    @Override
        public void requestFitSystemWindows() {
            checkThread();
            mApplyInsetsRequested = true;
            scheduleTraversals();
        }
    
        @Override
        public void requestLayout() {
            if (!mHandlingLayoutInLayoutRequest) {
                checkThread();
                mLayoutRequested = true;
                scheduleTraversals();
            }
        }
    
    
    void checkThread() {
            if (mThread != Thread.currentThread()) {
                throw new CalledFromWrongThreadException(
                        "Only the original thread that created a view hierarchy can touch its views.");
            }
        }
```

重要的是这个checkThread方法，ViewRootImpl的mThread即主线程，也就是说非主线程更新ui就会抛出异常。

**问题2：** 如果设计成任何线程都可以更新ui的话，那么多线程并发的去访问ui很有可能造成不可预期的结果，这是可想而知的，虽然加锁可以解决并发访问的问题，但是加上锁会让ui访问逻辑变得很复杂，其次锁降低了ui访问的访问效率。


---
## 2 消息机制相关的几个类

Handler的运行需要MessageQueue和Looper支撑：

- MessageQueue：消息队列，内部存储着一组消息，并负责对消息进行排序，但它并不是真的使用队列存储消息，而是使用单链表结构
- Looper：消息轮询器，它负责不断的从消息队列中取出消息，交给对应的Handler处理，没有消息则当前线程进入等待状态。
- ThreadLocal：ThreadLocal是java层面的东西，是一个线程内数据共享类，它在消息机制中起到非常重要的作用。

### ThreadLocal

为了搞清楚Handler机制的内部原理，我们需要先弄清除ThreadLocal的原理。

ThreadLocal是一个线程内数据共享类，通过它可以在指定线程存储数据，数据存储以后，只有指定线程可以获取到存储的数据，而其他线程则无法获取到，在Android系统中，Looper，ActivityThread，AMS都有用到ThreadLocal，具体到ThreadLocal的使用场景，可以总结为：**当某些数据是以线程为作用域，并且不同线程具有不同的数据副本时，可以考虑采用ThreadLocal**，而对于Handler来说，他要获取的就是当前线程的Looper，很显然Looper的作用域就是线程，不同的线程具有不同的Looper。

ThreadLocal还有一个使用场景，复杂逻辑下的对象传递，比如监听器的传递，有时候一个线程的逻辑过于复杂，调用链路比较深，这时如果让每个方法都提供一个参数去接收监听器对象,很显然这样的设计是很糟糕的，这时候就可以使用到ThreadLocal。

下面一个通过一个Demo来ThreadLocal的使用

```java
    public class ThreadLocalTest {
    
        private ThreadLocal<Integer> mThreadLocal;
        
        public  ThreadLocalTest() {
            mThreadLocal = new ThreadLocal<>();
            mThreadLocal.set(0);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            testThread1();
            
            testThread2();
            System.out.println("testThread main "+mThreadLocal.get());
        }
        
        public void testThread1() {
            new Thread(){
                public void run() {
                    mThreadLocal.set(1);
                    System.out.println("testThread1 "+mThreadLocal.get());
                }
            }.start();
            
        }
        
        public void testThread2() {
            new Thread(){
                public void run() {
                    mThreadLocal.set(2);
                    System.out.println("testThread2 "+mThreadLocal.get());
                }
            }.start();
        }
        
        /**
         * @param args
         */
        public static void main(String[] args) {
            ThreadLocalTest threadLocalTest= new ThreadLocalTest();
        }
    
    }
```

    打印结果：
    testThread1 1
    testThread main 0
    testThread2 2
    

可见每个线程都通过ThreadLocal存储了自己的数据，而其他线程无法访问。ThreadLocal只暴露了get，set和remove方法，通过set可以给当前线程存储一个临时变量，通过get可以获取存储的临时变量，只要搞清楚这两个方法，就可以明白其原理。

**set方法：**

```java
        public void set(T value) {
            Thread currentThread = Thread.currentThread();
            Values values = values(currentThread);
            if (values == null) {
                values = initializeValues(currentThread);
            }
            values.put(this, value);
        }
        
        Values initializeValues(Thread current) {
        return current.localValues = new Values();
    }
```
可以看到initializeValues方法初始化了线程的成员变量localValues，这个localValues用来存储线程的ThreadLoca变量，这样每个线程都有自己的专门的域来存储数据。Values内部维护着一个Object数组，用来存储数据，具体就不再分析了。

**get方法：**

```java
    public T get() {
            // Optimized for the fast path.
            Thread currentThread = Thread.currentThread();
            Values values = values(currentThread);
            if (values != null) {
                Object[] table = values.table;
                int index = hash & values.mask;
                if (this.reference == table[index]) {
                    return (T) table[index + 1];
                }
            } else {
                values = initializeValues(currentThread);
            }
    
            return (T) values.getAfterMiss(this);
        }

       Values values(Thread current) {
        return current.localValues;
    }
```
values方法获取当前线程的localValues对象，然后通过ThreadLocal的内部变量的一些计算，获取当前ThreadLocal存储的对象对应Thread中localValues中的索引角标，最后通过索引来获取存储的对象。从ThreadLocal的set和get方法来看，它们操作的都是Thread的成员变量localValues对象，所以对于不同线程对同一个ThreadLocal的set，get方法的操作，仅限于各自线程的内部。

### 消息队列工作原理

消息队列即MessageQuque，它主要有有两个操作，插入消息和读取消息(读取的同时删除)，对应的方法是enqueueMessage和next方法，其实我们我通过handler发生消息和post发送任务，最终都会以消息的形式加入到MessageQuque中，接下来看一下这两个方法：

**enqueueMessage：**

```java
    final boolean enqueueMessage(Message msg, long when) {
            if (msg.when != 0) {
                throw new AndroidRuntimeException(msg
                        + " This message is already in use.");
            }
            if (msg.target == null && !mQuitAllowed) {
                throw new RuntimeException("Main thread not allowed to quit");
            }
            final boolean needWake;
            synchronized (this) {
                if (mQuiting) {
                    RuntimeException e = new RuntimeException(
                        msg.target + " sending message to a Handler on a dead thread");
                    Log.w("MessageQueue", e.getMessage(), e);
                    return false;
                } else if (msg.target == null) {
                    mQuiting = true;
                }
    
                msg.when = when;
                //Log.d("MessageQueue", "Enqueing: " + msg);
                Message p = mMessages;
                if (p == null || when == 0 || when < p.when) {
                    msg.next = p;
                    mMessages = msg;
                    needWake = mBlocked; // new head, might need to wake up
                } else {
                    Message prev = null;
                    while (p != null && p.when <= when) {
                        prev = p;
                        p = p.next;
                    }
                    msg.next = prev.next;
                    prev.next = msg;
                    needWake = false; // still waiting on head, no need to wake up
                }
            }
            if (needWake) {
                nativeWake(mPtr);
            }
            return true;
        }
```

从 enqueueMessage 方法来看，主要是把对消息进行一定的验证，然后根据消息要求的时间把消息插入到消息队列中去，如果消息插入失败，就会返回false，最后如果有需要就会唤醒线程去读取消息。

**接着看next方法**

```java
         Message next() {
            // Return here if the message loop has already quit and been disposed.
            // This can happen if the application tries to restart a looper after quit
            // which is not supported.
            final long ptr = mPtr;
            if (ptr == 0) {
                return null;
            }
    
            int pendingIdleHandlerCount = -1; // -1 only during first iteration
            int nextPollTimeoutMillis = 0;
            for (;;) {
                if (nextPollTimeoutMillis != 0) {
                    Binder.flushPendingCommands();
                }
    
                nativePollOnce(ptr, nextPollTimeoutMillis);
    
                synchronized (this) {
                    // Try to retrieve the next message.  Return if found.
                    final long now = SystemClock.uptimeMillis();
                    Message prevMsg = null;
                    Message msg = mMessages;
                    if (msg != null && msg.target == null) {
                        // Stalled by a barrier.  Find the next asynchronous message in the queue.
                        do {
                            prevMsg = msg;
                            msg = msg.next;
                        } while (msg != null && !msg.isAsynchronous());
                    }
                    if (msg != null) {
                        if (now < msg.when) {
                            // Next message is not ready.  Set a timeout to wake up when it is ready.
                            nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                        } else {
                            // Got a message.
                            mBlocked = false;
                            if (prevMsg != null) {
                                prevMsg.next = msg.next;
                            } else {
                                mMessages = msg.next;
                            }
                            msg.next = null;
                            if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                            msg.markInUse();
                            return msg;
                        }
                    } else {
                        // No more messages.
                        nextPollTimeoutMillis = -1;
                    }
    
                    // Process the quit message now that all pending messages have been handled.
                    if (mQuitting) {
                        dispose();
                        return null;
                    }
    
                    // If first time idle, then get the number of idlers to run.
                    // Idle handles only run if the queue is empty or if the first message
                    // in the queue (possibly a barrier) is due to be handled in the future.
                    if (pendingIdleHandlerCount < 0
                            && (mMessages == null || now < mMessages.when)) {
                        pendingIdleHandlerCount = mIdleHandlers.size();
                    }
                    if (pendingIdleHandlerCount <= 0) {
                        // No idle handlers to run.  Loop and wait some more.
                        mBlocked = true;
                        continue;
                    }
    
                    if (mPendingIdleHandlers == null) {
                        mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                    }
                    mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
                }
    
                // Run the idle handlers.
                // We only ever reach this code block during the first iteration.
                for (int i = 0; i < pendingIdleHandlerCount; i++) {
                    final IdleHandler idler = mPendingIdleHandlers[i];
                    mPendingIdleHandlers[i] = null; // release the reference to the handler
    
                    boolean keep = false;
                    try {
                        keep = idler.queueIdle();
                    } catch (Throwable t) {
                        Log.wtf(TAG, "IdleHandler threw exception", t);
                    }
    
                    if (!keep) {
                        synchronized (this) {
                            mIdleHandlers.remove(idler);
                        }
                    }
                }
    
                // Reset the idle handler count to 0 so we do not run them again.
                pendingIdleHandlerCount = 0;
    
                // While calling an idle handler, a new message could have been delivered
                // so go back and look again for a pending message without waiting.
                nextPollTimeoutMillis = 0;
            }
        }
```

next方法中是一个无限循环，如果消息队列没有消息，当前线程就会阻塞。由于现场的唤醒和阻塞都是native实现的，这里就不做深入研究了。

#### 消息轮询器的工作原理

消息轮询器即Looper，我们知道如果要在子线程使用Looper，必须先调用Looper.prepare()初始化线程对应的Looper和MeeageQueue,否则会报错，而主线程不需要是因为系统已经初始化好了

而且Looper的文档注释中已经给出了Looper的使用示例：

```java
    class LooperThread extends Thread {
          public Handler mHandler;
          
          public void run() {
              Looper.prepare();
              
              mHandler = new Handler() {
                  public void handleMessage(Message msg) {
                      // process incoming messages here
                  }
              };
              
              Looper.loop();
          }
      }
```

前面说到Hander机制依赖于Looper和MessageQueue，由于默认的线程没有Looper和MessageQueue，当我们想在子线程使用Handler时，必须先调用Lopper.prepare方法，否则会报错

```java
        public Handler() {
           //......
            mLooper = Looper.myLooper();
            if (mLooper == null) {
                throw new RuntimeException(
                    "Can't create handler inside thread that has not called Looper.prepare()");
            }
            mQueue = mLooper.mQueue;
            mCallback = null;
        }
```

先来看一下Looper的prepare方法


**Looper.prepare()：**

```
        public static final void prepare() {
            if (sThreadLocal.get() != null) {
                throw new RuntimeException("Only one Looper may be created per thread");
            }
            sThreadLocal.set(new Looper());
        }
```

方法很简单，如果Looper已经初始化则抛出异常。否则初始化好一个Looper，利用ThreadLocal存储起来。

```
        private Looper() {
            mQueue = new MessageQueue();
            mRun = true;
            mThread = Thread.currentThread();
        }
```

而Looper的构造方法也很简单，初始化对应的MessageQueue，修改内部标识，记录它所在线程。这样Looper就初始化好了，我们也可以得出一个结论：**一个线程只能对应一个Looper和MessageQueue**，Looper除了提供preparer方法外，还提供了quit和quitSafely方法，quit用于直接退出loop循环，而quitSafey只是设定一个标识，当Looper轮询完所有消息后才自动退出**，Looper退出后，通过Handler发送消息会发送失败，这时Handler的sendXXX方法返回false**，在子线程中如果不再需要使用Looper，最后是让Looper退出消息轮询结束线程。

其次，Looper还有一个方法用于获取主线程的Loope，这是因为主线程的Looper笔记特殊：

```java
     public synchronized static final Looper getMainLooper() {
            return mMainLooper;
        }
    //创建主线程的Looper，并且主线程消息队列是不可退出的
     public static final void prepareMainLooper() {
            prepare();
            setMainLooper(myLooper());
            if (Process.supportsProcesses()) {
                myLooper().mQueue.mQuitAllowed = false;
            }
        }
    //设置主线程的looper
    private synchronized static void setMainLooper(Looper looper) {
            mMainLooper = looper;
    }
```

当Looper创建完毕后，我们需要调用其loop方法，让Looper启动：

```java
           public static void loop() {
            final Looper me = myLooper();
            if (me == null) {
                throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
            }
            final MessageQueue queue = me.mQueue;
    
            // Make sure the identity of this thread is that of the local process,
            // and keep track of what that identity token actually is.
            Binder.clearCallingIdentity();
            final long ident = Binder.clearCallingIdentity();
    
            for (;;) {
                Message msg = queue.next(); // might block
                if (msg == null) {
                    // No message indicates that the message queue is quitting.
                    return;
                }
    
                // This must be in a local variable, in case a UI event sets the logger
                Printer logging = me.mLogging;
                if (logging != null) {
                    logging.println(">>>>> Dispatching to " + msg.target + " " +
                            msg.callback + ": " + msg.what);
                }
    
                msg.target.dispatchMessage(msg);
    
                if (logging != null) {
                    logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
                }
    
                // Make sure that during the course of dispatching the
                // identity of the thread wasn't corrupted.
                final long newIdent = Binder.clearCallingIdentity();
                if (ident != newIdent) {
                    Log.wtf(TAG, "Thread identity changed from 0x"
                            + Long.toHexString(ident) + " to 0x"
                            + Long.toHexString(newIdent) + " while dispatching to "
                            + msg.target.getClass().getName() + " "
                            + msg.callback + " what=" + msg.what);
                }
    
                msg.recycleUnchecked();
            }
        }
```

可以看到，Looper就是一个无限循环，不断的从消息队列中取出消息，然后分发给Message对应的Hander处理消息，最后消息处理完毕，会自动回收消息，平时我们获取一个Message对象时，最好是使用Message.obtain()，这样可以才能真正循环利用Message对象。

#### Looper的退出

当消息队列的next方法返回null时，循环就会自动退出，这里我们可以想到Looper的quit方法：

```java
        //Looper
        public void quit() {
            mQueue.quit(false);
        }
    
    
        public void quitSafely() {
            mQueue.quit(true);
        }

     //MessageQueue
     void quit(boolean safe) {
            if (!mQuitAllowed) {
                throw new IllegalStateException("Main thread not allowed to quit.");
            }
    
            synchronized (this) {
                if (mQuitting) {
                    return;
                }
                mQuitting = true;
    
                if (safe) {
                    removeAllFutureMessagesLocked();
                } else {
                    removeAllMessagesLocked();
                }
    
                // We can assume mPtr != 0 because mQuitting was previously false.
                nativeWake(mPtr);
            }
        }
```

Looper的quit方法调用MessageQueue的quit方法，首先验证是否是主线程(除了主线程其他Looper都是可以退出的)，然后把mQuitting置为true并唤醒线程。线程被唤醒又会去读取消息，这时由于mQuitting已经是true，所以MessageQueue的next方法会返回null，那么looper的死循环就会结束。


```java
          //MessageQueue的next方法片段
          // Process the quit message now that all pending messages have been handled.
   
                   if (mQuitting) {
                        dispose();
                        return null;
                    }
```

### Handler的工作原理

分析完消息队列和消息轮询器，最后来看一下Hanlder的内部原理，刚刚说到Looper在loop方法中，不断的取出消息，然后通过下面代码发送给对于的handler处理：

```java
    //loop方法片段
    msg.target.dispatchMessage(msg);
```

msg即Message，target即Handler，那么msg.target是什么时候被赋值的呢？那就需要分析Handler的sendMessage系列方法了，虽说Handler有很多发生消息的方法，但最终都会调用sendMessageAtTime方法：

```java
        public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
            MessageQueue queue = mQueue;
            if (queue == null) {
                RuntimeException e = new RuntimeException(
                        this + " sendMessageAtTime() called with no mQueue");
                Log.w("Looper", e.getMessage(), e);
                return false;
            }
            return enqueueMessage(queue, msg, uptimeMillis);
        }

              private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
            msg.target = this;
            if (mAsynchronous) {
                msg.setAsynchronous(true);
            }
            return queue.enqueueMessage(msg, uptimeMillis);
        }
```

到这里我们就可以连接前面分析的MessageQueue和Looper了，Handler发生的消息，最终还是通过轮询器分发给自己处理。

**处理消息**

```java
        public void dispatchMessage(Message msg) {
            if (msg.callback != null) {
                handleCallback(msg);
            } else {
                if (mCallback != null) {
                    if (mCallback.handleMessage(msg)) {
                        return;
                    }
                }
                handleMessage(msg);
            }
        }
```
dispatchMessage方法接收到消息后，先检查优先交给msg.callback处理，msg.callback就是一个Runnable对象，对于处理消息就是调用它的run方法，

```java
      private static void handleCallback(Message message) {
            message.callback.run();
        }
```

平时我们调用Handler.post(Runnale r)的话，r就会被这样处理。

```java
       public final boolean post(Runnable r){
           return  sendMessageDelayed(getPostMessage(r), 0);
        }
    
        private static Message getPostMessage(Runnable r) {
            Message m = Message.obtain();
            m.callback = r;
            return m;
        }
```

然后，当msg.callback为null时。则会判断mCallback是否为null，如果不为null，则消息有限给mCallBack处理，mCallback是个什么角色呢？

```java
       public interface Callback {
            public boolean handleMessage(Message msg);
        }
    
        public Handler(Callback callback) {
            this(callback, false);
        }
```

Callback是一个回调接口，可以作为Handler的构造参数传入，其作用就是创建一个Handler时，如果传入CallBack,我们就没有必要复写Handler的handMessage方法了。也可以说不用派生出Handler的子类。最后如果mCallback也为null，就会调用Handler自己的handMessage了。


### 消息的移除

我们可以通过Handler发送消息，同时也可以通关过Handler移除已经发送的没有处理消息，移除消息的方法有很多，用途也不一样：

```java
       //Remove any pending posts of messages with code 'what' that are in the message queue.
        public final void removeMessages(int what) {
            mQueue.removeMessages(this, what, null);
        }
    
       //Remove any pending posts of messages with code 'what' and whose obj is 'object' that are in the message queue. If object is null, all messages will be removed.
        public final void removeMessages(int what, Object object) {
            mQueue.removeMessages(this, what, object);
        }
    
        //Remove any pending posts of callbacks and sent messages whose obj is token. If token is null, all callbacks and messages will be removed.
        public final void removeCallbacksAndMessages(Object token) {
            mQueue.removeCallbacksAndMessages(this, token);
        }
```

最终都是调用MessageQueue的相关方法，如果要移除Handler发送的所有消息，则可以使用Handler的removeCallbacksAndMessages方法，传入null即可


### 主线程的消息循环

Android的主线程就是ActivityThread，主线程的入口即main方法：

```java
     public static void main(String[] args) {
           ......
            Looper.prepareMainLooper();
    
            ActivityThread thread = new ActivityThread();
            thread.attach(false);
    
            if (sMainThreadHandler == null) {
                sMainThreadHandler = thread.getHandler();
            }
            AsyncTask.init();
            if (false) {
                Looper.myLooper().setMessageLogging(new
                        LogPrinter(Log.DEBUG, "ActivityThread"));
            }
            Looper.loop();
            throw new RuntimeException("Main thread loop unexpectedly exited");
        }
```

main方法中，初始化了主线程的Looper，并开始消息轮询，而ActivityThread的内部Handler H主要用来处理处理四大组件的启动和停止过程。

```java
    private class H extends Handler {
            public static final int LAUNCH_ACTIVITY         = 100;
            public static final int PAUSE_ACTIVITY          = 101;
            public static final int PAUSE_ACTIVITY_FINISHING= 102;
            public static final int STOP_ACTIVITY_SHOW      = 103;
            public static final int STOP_ACTIVITY_HIDE      = 104;
            public static final int SHOW_WINDOW             = 105;
            public static final int HIDE_WINDOW             = 106;
            public static final int RESUME_ACTIVITY         = 107;
            public static final int SEND_RESULT             = 108;
            public static final int DESTROY_ACTIVITY        = 109;
            public static final int BIND_APPLICATION        = 110;
            public static final int EXIT_APPLICATION        = 111;
            public static final int NEW_INTENT              = 112;
            public static final int RECEIVER                = 113;
            public static final int CREATE_SERVICE          = 114;
            public static final int SERVICE_ARGS            = 115;
            ......
    }
```

可以看到H类内部定义的消息类型跟四大组件的工作密切相关。ActivityThread主要通过内部类ApplicationThread与AMS进行通信，AMS以进程间通信方法未完成ActivityThread的请求后回调ApplicationThread，由于进程间通信回调都运行在Binder线程池中，所以ApplicaionThread通过发消息到H，H收到消息后切换到主线程ActivityThread中去执行相关操作，这就是主线程的消息循环模型。
















