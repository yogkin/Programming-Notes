# Messenger

## Messenger介绍

Messenger可以翻译为信使，它可以在不同的进程中传递对象，其底层是有AIDL实现的
有两个构造方法：

```java
        //用于封装客户端传递过来的Binder
        public Messenger(IBinder target) {
            mTarget = IMessenger.Stub.asInterface(target);
        }
        //把客户端的Handle封装起来，然后发送到服务端
        //可以通过Intent或者Message.replyTo发送
        public Messenger(Handler target) {
            mTarget = target.getIMessenger();
        }
```

## 使用方法
Messenger的使用方法非常简单，下面实现一个Activity和处于另外一个进程的Service进行通信的列子

Activity代码:

```java
    public class MessengerActivity extends AppCompatActivity {
    
    
        private Messenger mServiceMessenger;//服务信使
        private Messenger mClientMessenger;//本地信使，给服务端
        public static final int CLIENT_MESSENGER = 12312;
    
        private void init() {
            mClientMessenger = new Messenger(new MessengerClientHandler());
            bindService(new Intent(this, MessengerService.class), mMessengerActivity, Context.BIND_AUTO_CREATE);
        }
    
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_messenger);
            init();
        }
    
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            unbindService(mMessengerActivity);
        }
    
    
        private ServiceConnection mMessengerActivity = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                mServiceMessenger = new android.os.Messenger(service);
                Message obtain = Message.obtain();
                obtain.what = CLIENT_MESSENGER;
                //在跨进程通信中，message的obj必须是安卓系统的parcelable类型对象
                Bundle bundle = new Bundle();
                bundle.putString("key", "this is client");
                obtain.setData(bundle);
                //通过replyTo传递messenger
                obtain.replyTo = mClientMessenger;
                try {
                    mServiceMessenger.send(obtain);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
    
            @Override
            public void onServiceDisconnected(ComponentName name) {
                Log.d("MessengerActivity", "name:" + name);
            }
        };
    
    
        private static class MessengerClientHandler extends Handler {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                Log.d("MessengerClientHandler", "msg.what:" + msg.what);
                Log.d("MessengerClientHandler", "msg.getData()" + msg.getData().get("key"));
            }
        }
    
    
        public void send(View view) {
            Message obtain = Message.obtain();
            obtain.what = 4234;
            Bundle bundle = new Bundle();
            bundle.putString("key", "打你");
            obtain.setData(bundle);
            try {
                mServiceMessenger.send(obtain);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    }
```

Service代码和配置：

```java
    public class MessengerService extends Service {
        public MessengerService() {
        }
    
    
        private Messenger mMessenger = new Messenger(new MessengerHandler(this));//本地信使，传递给客户端
        private Messenger mClientMessenger;//客户端信使，用于向客户端发送消息
    
        @Override
        public IBinder onBind(Intent intent) {
            return mMessenger.getBinder();
        }
    
    
        public void initClientMessenger(Messenger messenger) {
            mClientMessenger = messenger;
            Message message = Message.obtain();
            message.what = 2;
            Bundle bundle = new Bundle();
            bundle.putString("key", "你好，我是客服务端端");
            message.setData(bundle);
            try {
                mClientMessenger.send(message);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    
        public static class MessengerHandler extends Handler {
    
            public WeakReference<MessengerService> mServiceWeakReference;
    
            public MessengerHandler(MessengerService messengerService) {
                mServiceWeakReference = new WeakReference<>(messengerService);
            }
    
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                Log.d("MessengerHandler", "msg.what:" + msg.what);
                Log.d("MessengerHandler", "msg.obj:" + msg.getData().get("key"));
                switch (msg.what) {
                    case MessengerActivity.CLIENT_MESSENGER:
                        if (mServiceWeakReference.get() != null) {
                            mServiceWeakReference.get().initClientMessenger(msg.replyTo);
                        }
                        break;
                    case 4234:
                        if (mServiceWeakReference.get() != null) {
                            mServiceWeakReference.get().sendMessage();
                        }
                        break;
                }
            }
        }
    
        private void sendMessage() {
            Message message = Message.obtain();
            message.what = 2;
            Bundle bundle = new Bundle();
            bundle.putString("key", "有本事就来啊，傻吊");
            message.setData(bundle);
            try {
                mClientMessenger.send(message);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    }
```

配置

```xml
        <service
            android:name=".service.MessengerService"
            android:enabled="true"
            android:exported="true"
            android:process=":remote_messenger">
        </service>
```


需要注意的是：

- 在跨进程通信中，message的obj必须是安卓系统的parcelable类型对象，非安卓系统的parcelable类型对象通过下面方式传递：
```java
                Bundle bundle = new Bundle();
                bundle.putString("key", "this is client");
                obtain.setData(bundle);
                //通过replyTo传递messenger
                obtain.replyTo = mClientMessenger;
```
- `Message.replyTo`可以把Messenger发生到服务端，这样可以建立双向通信

```java
                    obtain.replyTo = mClientMessenger;
                    mServiceMessenger.send(obtain);
```

另外Messenger还以使用Intent来传递：
```java
       private Messenger mClientMessenger;//本地信使，给服务端
       Intent intent = new Intent(this, BookManagerActivity.class);
       intent.putExtra("messenger", mClientMessenger);
```
被启动的组建可以通过Intent来获取传递过来的Intent。







