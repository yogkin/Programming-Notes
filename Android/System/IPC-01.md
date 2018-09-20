# Androd IPC概述

IPC即 **Inter-Process Communication**，进程间通信即 **两个进程间进行数据交换**。

Android使用的是的Linux内核，Linux的IPC方式包括`命名管道、共享内存、信号量`等，但是其IPC方式并没有采用Linux IPC方式。Android的IPC大部分采用的是Binder机制，当然通过以下方式也可以实现进程间通讯

- Intent传递数据
- 共享文件
- 基于Binder的Messenger，AIDL
- 内容提供者
- Socket

IPC 的使用场景

- 为了分配更多内存，有时会把一个应用拆分成两个或多个进程
- 需要获取其他进程的数据

## 1 Android开启多线程模式

唯一的方法(除用c语言fork)，四大组件在Manifest文件中指定process属性，如下面

```xml
            <service
                android:name=".service.IPCService"
                android:process=":ipc_service"
                android:enabled="true"
                android:exported="true">
            </service>
```

### `:`与`.`的区别

- `:`表示当前的进程名前面附加应用的包名，`:`指定的进程术语私有进程，其他应用的组建不可以与他跑在同一进程中，
- " `.`是一种完整的命名方式，不会附加包名信息，`.`指定的进程是一个全局进程，其他应用可以通过ShareUID方式合同跑在同一个进程

通过ShareUID前提提交是两个应用必须使用相同的签名打包

### 多进程造成的影响

- 静态变量和单利模式完全失效
- 线程同步机制完全失效
- SharedPreference的可靠性降低
- Application会被创建多次

---
## 2 相关概念

Android IPC涉及到Binder、Messenger、Socket、内容提供者、序列化、共享文件等。

### 序列化

进程间传递数据，需要先将对象序列化，然后才能通过Intent和Binder传输，Android支持两种序列化方式

- Serializable接口：Java提供的序列化接口，序列化到文件，实现这个接口最好定义SerialVersionUID，特点是：实现简单，效率不高
- Parcelable接口：Parcelable接口是android提供的对象序列化接口，序列化到内存，特点是：实现麻烦(但是有了AndroidStudio后，会自动帮助实现)、效率高

下面是一个实现例子：

```java
    public class User implements Parcelable{

        private String name;
        private String pwd;

        protected User(Parcel in) {
            name = in.readString();
            pwd = in.readString();
        }
    
        public static final Creator<User> CREATOR = new Creator<User>() {
            @Override
            public User createFromParcel(Parcel in) {
                return new User(in);
            }
    
            @Override
            public User[] newArray(int size) {
                return new User[size];
            }
        };
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public String getPwd() {
            return pwd;
        }
    
        public void setPwd(String pwd) {
            this.pwd = pwd;
        }
    
        @Override
        public int describeContents() {
            return 0;
        }
    
        @Override
        public void writeToParcel(Parcel dest, int flags) {
            dest.writeString(name);
            dest.writeString(pwd);
        }
    }
```

可见Parcelable是用过Parcel类的一系列`read`和`write`方法来完成对象序列化的，`describeContents`方法几乎所有情况都返回`0`，仅当对象中存在存在文件描述符时返回`-1`。需要注意的是，序列化前的对象与序列化后的对象可能不是同一个对象，只是对象的内容相同而已

### Intent 传递 Bundle

Android系统的Intent架构本来就有跨进程传递的功能，只是在Intent中只能传递小数据，像占用内存较大的Bitmap就不适合

### 共享文件

使用文件可以实现进程间通信，因为不同进程都可以对同一文件进行读写，需要注意好同步，不推荐使用SharedPreferences，由于系统对SharedPreferences有一定的缓存策略，所以在多进程模式下，对它的读写并不可靠。


---
## 3 IPC方式的使用场景

各种IPC方式的优缺点和适用场景

|  名称 | 优点  | 缺点  | 适用场景  |
| ------------ | ------------ | ------------ | ------------ |
| Bundle  |  简单易用 | 只能传输Bundle规定的数据  | 四大组件之间的数据传递  |
|  文件共享 |  简单易用 | 不适用与高并发场景，无法进程间及时通讯  | 无高并发，交互简单数据  |
| AIDL  | 功能强大，支持实时通信，一对多高并发  | 实现稍微复杂，处理好线程同步  | 都可以适用  |
|  Messenger | 功能一般，支持一对多串行通信，可实时通信  | 不能很好处理高并发，数据通过message传递，只能传递Bundle类型  | 低并发的一对多通信  |
| ContentProvider  | 在数据源访问方法功能强大， 支持一对多，高并发 | 只能提供单向访问  | 一对多进程间数据共享  |
| Socket  | 功能强大  |  实现较为复杂 |  网络数据交换 |
