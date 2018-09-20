#  Binder中服务端与客户端通信细节与权限验证

在之前的例子中简单的实现了一个BookManagerService的例子，但是现在情况变了，客户端需要监听服务端的书籍变化情况，很明显这是一个观察者模式，服务端需要提供注册监听与发注册的方法，并把数据的变化情况通过一个接口分发给客户端，这涉及到以下几个问题：

---
## 普通的接口回调无法在AIDL中使用

```java
    // IOnNewBookArrivedListener.aidl
    package com.ztiany.androidipc;
    // Declare any non-default types here with import statements
    import java.util.List;
    import com.ztiany.androidipc.model.Book;

    interface IOnNewBookArrivedListener {
        /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
        void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
                double aDouble, String aString);
                //此方法用来通知客户端书籍发生变化
                void onNewBookArrived(in Book newBook);
    }
```

IBookManager.aidl增加两个方法：

```java
    // IBookManager.aidl
    package com.ztiany.androidipc;
    // Declare any non-default types here with import statements
    import java.util.List;
    import com.ztiany.androidipc.model.Book;
    import  com.ztiany.androidipc.IOnNewBookArrivedListener;
    interface IBookManager {
        /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
        void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
                double aDouble, String aString);
    
        void addBook(in Book book);
        List<Book> getBookList();
        //注册
        void registerListener(IOnNewBookArrivedListener listener);
        //反注册
        void unregisterListener(IOnNewBookArrivedListener listener);
    }
```

在Activity退出的时候，通过反注册来取消订阅，但是在服务端使用传统的List来保存监听是不行的，因为对象在进程间传递是通过序列化与反序列化的，其实传递之后的对象已经不再是用一个对象，只是对象的内容一样而已，所以这样服务端通过在List对已经注册的对象进行移除。好在Android提供了一个适用于此情景的容器`RemoteCallBackList`，它内部保存的key是共享内存中的Binder对象，这个Binder不管在服务端和客户端都是同一个对象，所以可以作为key来保存订阅者，也可以通过key正常移除，而且它内部已经做好了同步，并且当客户端进程结束后它会自动移除注册,其内部实现：

```java
    public class RemoteCallbackList<E extends IInterface> {
        /*package*/ ArrayMap<IBinder, Callback> mCallbacks
                = new ArrayMap<IBinder, Callback>();
        private Object[] mActiveBroadcast;
         .......
    }

    private RemoteCallbackList<IOnNewBookArrivedListener> mListeners;

      //遍历方法，beginBroadcast和finishBroadcast必须配对调用
        private void onNewBookAdd(Book book) throws RemoteException {
            final int N = mListeners.beginBroadcast();
            for (int i = 0; i < N; i++) {
                IOnNewBookArrivedListener listener = mListeners.getBroadcastItem(i);
                if (listener != null) {
                    listener.onNewBookArrived(book);
                }
            }
            mListeners.finishBroadcast();
        }
```

---
## 权限验证

我们不希望所有的程序都可以访问我们的数据，所以需要进行权限验证，有两种方法：

### 在onBinder中验证权限

```java
    //声明权限
    <uses-permission android:name="com.ztiany.androidipc.IBookManager"/>
    //定义权限
    <permission
            android:name="com.ztiany.androidipc.IBookManager"
            android:protectionLevel="normal"/>

```

验证：
```java
     @Override
        public IBinder onBind(Intent intent) {
            int check = checkCallingOrSelfPermission("com.ztiany.androidipc.IBookManager");
            if (check == PackageManager.PERMISSION_DENIED) {
                Log.d("BookManagerService", "PERMISSION_DENIED");
                return null;
            }
            Log.d("BookManagerService", "PERMISSION_OK");
            return new BookManager();
        }
```

### 在onTransact中进行验证

比如验证权限和包名等：
```java
      @Override
            public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
                int check = checkCallingOrSelfPermission("com.ztiany.androidipc.IBookManager");
                if (check == PackageManager.PERMISSION_DENIED) {
                    Log.d("BookManagerService", "PERMISSION_DENIED");
                    return false;
                }
                Log.d("BookManagerService", "PERMISSION_OK");
    
                String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
                if (packages != null && packages.length > 0) {
                    if(!packages[0].startsWith("com.ztiany")){
                        Log.d("BookManagerService", "package name error");
                        return false;
                    }
                }
                Log.d("BookManagerService", "package name right");

                return super.onTransact(code, data, reply, flags);
            }
```

---
## 全部代码

Activity代码：在onDestroy中用`mIBookManager.asBinder().isBinderAlive()`判断binder是否挂掉，没有挂掉则反订阅。
```java
    public class BookManagerActivity extends AppCompatActivity {
    
        private IBookManager mIBookManager;
    
        private EditText mBookIdEt;
        private EditText mBookNameEt;
        private InterHandler mInterHandler;
        private String TAG = BookManagerActivity.class.getSimpleName();
    
    
        private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
            @Override
            public void binderDied() {
                Log.d("BookManagerActivity", Thread.currentThread().getName());
    
                if (mIBookManager == null) {
                    return;
                }
                mIBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
                mIBookManager = null;
                //重连
                bindService(null);
    
            }
        };
    
    
        public static class InterHandler extends Handler {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                Log.d("InterHandler", "new book add" + msg.obj);
            }
        }
    
        private IOnNewBookArrivedListener.Stub mIOnNewBookArrivedListener = new IOnNewBookArrivedListener.Stub() {
    
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {
    
            }
    
    
    
            @Override
            public void onNewBookArrived(Book newBook) throws RemoteException {
                Message message = Message.obtain();
                message.obj = newBook;
                mInterHandler.sendMessage(message);
            }
        };
    
    
        private ServiceConnection mServiceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                if (service != null) {
                    mIBookManager = IBookManager.Stub.asInterface(service);
                    try {
                        mIBookManager.registerListener(mIOnNewBookArrivedListener);
                        service.linkToDeath(mDeathRecipient, 0);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
    
            }
    
            @Override
            public void onServiceDisconnected(ComponentName name) {
                Log.d(TAG, "" + name.getPackageName());
                Log.d("BookManagerActivity", Thread.currentThread().getName());
            }
        };
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_book_manager);
            initView();
            mInterHandler = new InterHandler();
        }
    
        private void initView() {
            mBookIdEt = (EditText) findViewById(R.id.id_act_book_id_et);
            mBookNameEt = (EditText) findViewById(R.id.id_act_book_name_et);
        }
    
    
        public void bindService(View view) {
    
            Intent intent = new Intent(this, BookManagerService.class);
            bindService(intent, mServiceConnection, Service.BIND_AUTO_CREATE);
        }
    
    
        public void addBook(View view) {
            String id = mBookIdEt.getText().toString();
            String name = mBookNameEt.getText().toString();
            if (TextUtils.isEmpty(id) || TextUtils.isEmpty(name)) {
                return;
            }
    
            if (mIBookManager != null) {
                try {
                    mIBookManager.addBook(new Book(id, name));
                    Toast.makeText(BookManagerActivity.this, "add success", Toast.LENGTH_SHORT).show();
                } catch (RemoteException e) {
                    e.printStackTrace();
                    Toast.makeText(BookManagerActivity.this, "add fail " + e.getCause(), Toast.LENGTH_SHORT).show();
                }
            }
        }
    
        public void getBookList(View view) {
            if (mIBookManager != null) {
                try {
                    List<Book> bookList = mIBookManager.getBookList();
                    if (bookList != null) {
                        Log.e(TAG, bookList.toString());
                    }
                } catch (RemoteException e) {
                    e.printStackTrace();
                    Toast.makeText(BookManagerActivity.this, "get book list fail " + e.getCause(), Toast.LENGTH_SHORT).show();
    
                }
            }
        }
    
        @Override
        protected void onDestroy() {
    
            super.onDestroy();
            if (mIBookManager == null) {
    
                return;
            }
    
            if (mIBookManager.asBinder().isBinderAlive()) {
                try {
                    mIBookManager.unregisterListener(mIOnNewBookArrivedListener);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
            mIBookManager = null;
    
            unbindService(mServiceConnection);
        }
    }
```
Service代码：
```java
    public class BookManagerService extends Service {
    
        private CopyOnWriteArrayList<Book> mBooks;
        private RemoteCallbackList<IOnNewBookArrivedListener> mListeners;
        private AtomicBoolean mSelfIsRunning = new AtomicBoolean(true);
    
    
        public BookManagerService() {
    
        }
    
        @Override
        public void onCreate() {
            super.onCreate();
            mSelfIsRunning.set(false);
            mListeners = new RemoteCallbackList<>();
            mBooks = new CopyOnWriteArrayList<>();
            mBooks.add(new Book("001", "Android 群英传"));
            mBooks.add(new Book("002", "Android 开发艺术探索"));
            new Thread(new AddBookRunnable()).start();
        }
    
    
        @Override
        public void onDestroy() {
            mSelfIsRunning.set(true);
            super.onDestroy();
        }
    
        @Override
        public IBinder onBind(Intent intent) {
            int check = checkCallingOrSelfPermission("com.ztiany.androidipc.IBookManager");
            if (check == PackageManager.PERMISSION_DENIED) {
                Log.d("BookManagerService", "PERMISSION_DENIED");
                return null;
            }
            Log.d("BookManagerService", "PERMISSION_OK");
            return new BookManager();
        }
    
    
        public class BookManager extends IBookManager.Stub {
    
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {
    
            }
    
            @Override
            public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
    
                int check = checkCallingOrSelfPermission("com.ztiany.androidipc.IBookManager");
                if (check == PackageManager.PERMISSION_DENIED) {
                    Log.d("BookManagerService", "PERMISSION_DENIED");
                    return false;
                }
                Log.d("BookManagerService", "PERMISSION_OK");
    
    
                String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
                if (packages != null && packages.length > 0) {
                    if(!packages[0].startsWith("com.ztiany")){
                        Log.d("BookManagerService", "package name error");
                        return false;
                    }
                }
                Log.d("BookManagerService", "package name right");
    
    
    
    
    
                return super.onTransact(code, data, reply, flags);
            }
    
            @Override
            public void addBook(Book book) throws RemoteException {
                mBooks.add(book);
                onNewBookAdd(book);
            }
    
            @Override
            public List<Book> getBookList() throws RemoteException {
                return mBooks;
            }
    
            @Override
            public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
                if (listener != null) {
                    mListeners.register(listener);
                    Log.d("BookManager", "register success");
                }
            }
    
            @Override
            public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
                if (listener != null) {
                    mListeners.unregister(listener);
                    Log.d("BookManager", "unregister success");
                } else {
                    Log.d("BookManager", "unregister fail");
                }
            }
        }
    
    
        /**
         * add
         *
         * @param book
         * @throws RemoteException
         */
        private void onNewBookAdd(Book book) throws RemoteException {
            final int N = mListeners.beginBroadcast();
            for (int i = 0; i < N; i++) {
                IOnNewBookArrivedListener listener = mListeners.getBroadcastItem(i);
                if (listener != null) {
                    listener.onNewBookArrived(book);
                }
    
            }
            mListeners.finishBroadcast();
        }
    
    
        /**
         * 不断添加书的任务
         */
        public class AddBookRunnable implements Runnable {
    
            int i = 4;
    
            @Override
            public void run() {
                while (!mSelfIsRunning.get()) {
                    i++;
                    Book book = null;
                    try {
                        if (mBooks != null) {
                            book = new Book(String.valueOf(i), "book haha" + i);
                            mBooks.add(book);
                        }
                        Thread.sleep(3000);
    
                        try {
                            if (book != null)
                                onNewBookAdd(book);
                        } catch (RemoteException e) {
                            e.printStackTrace();
                        }
    
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
    
        }
    }
```
