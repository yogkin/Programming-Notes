# 桌面Widget

AppWidgetProvder用于桌面小部件的开发，其本质是一个广播，Appwidget的通信机制是广播，AppWidget的开发步骤可以参考开发文档。注意只有安装在手机内存的的应用才能显示Widget，可以用下面方式配置**installLocation**：

```xml
    <manifest
        package="com.ztiany.remoteviews"
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:installLocation="internalOnly"></manifest>
```

## 简单示例


### 定义AppWidgetProvder

```java
    public class TWidgetProvider extends AppWidgetProvider {
    
        public static final String TAG = TWidgetProvider.class.getSimpleName();
        private String action = "com.ztiany.remote_view.widget.action";
    
    
        @Override
        public void onEnabled(Context context) {
            super.onEnabled(context);
            Log.d(TAG, "onEnabled");
        }
    
    
        @Override
        public void onReceive(final Context context, Intent intent) {
            super.onReceive(context, intent);
            Log.d(TAG, "onReceive");
            if (action.equals(intent.getAction())) {
                Toast.makeText(context, action, Toast.LENGTH_SHORT).show();
    
                new Thread(new Runnable() {
                    @Override
                    public void run() {
    
                        int i = 0;
                        while (true) {
                            RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.layout_widget);
                            Intent intent = new Intent();
                            intent.setAction(action);
                            PendingIntent broadcast = PendingIntent.getBroadcast(context, 100, intent, 0);
                            remoteViews.setOnClickPendingIntent(R.id.id_widget_btn, broadcast);
                            remoteViews.setTextViewText(R.id.id_widget_title_tv, "title-" + i);
                            remoteViews.setTextViewText(R.id.id_widget_content_tv, "content-" + i);
                            ComponentName componentName = new ComponentName(context, TWidgetProvider.class);
                            AppWidgetManager.getInstance(context).updateAppWidget(componentName, remoteViews);
                            i++;
                            if (i == 50) {
                                break;
                            }
                            SystemClock.sleep(3000);
                        }
                    }
                }).start();
            }
        }
    
        @Override
        public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
            Log.d(TAG, "onUpdate");
            Log.d(TAG, "onUpdate:"+Arrays.toString(appWidgetIds));
    
            for (int widgetId : appWidgetIds) {
                RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.layout_widget);
                Intent intent = new Intent();
                intent.setAction(action);
                PendingIntent broadcast = PendingIntent.getBroadcast(context, 100, intent, 0);
                remoteViews.setOnClickPendingIntent(R.id.id_widget_btn, broadcast);
                AppWidgetManager.getInstance(context).updateAppWidget(widgetId, remoteViews);
            }
        }
    
    
        @Override
        public void onDeleted(Context context, int[] appWidgetIds) {
            super.onDeleted(context, appWidgetIds);
            Log.d(TAG, "onDeleted");
        }
    
        @Override
        public void onDisabled(Context context) {
            super.onDisabled(context);
            Log.d(TAG, "onDisabled   ");
        }
    }
```


### 添加配置信息

清单文件

```xml
         <receiver android:name=".widget.TWidgetProvider">
                <intent-filter>
                    <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
                    <action android:name="com.ztiany.remote_view.widget.action"/>
                </intent-filter>
                <meta-data
                    android:name="android.appwidget.provider"
                    android:resource="@xml/widget_info"/>
            </receiver>
```

xml配置信息

```xml
    <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
                        android:minWidth="100dp"
                        android:minHeight="100dp"
                        android:updatePeriodMillis="86400000"
                        android:previewImage="@mipmap/ic_launcher"
                        android:initialLayout="@layout/layout_widget"
                        android:resizeMode="horizontal|vertical"
                        android:widgetCategory="home_screen">
    </appwidget-provider>
```


布局文件

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:layout_width="match_parent"
                  android:layout_height="wrap_content"
                  android:orientation="horizontal">

        <LinearLayout
            android:id="@+id/id_widget_ll"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1"
            android:gravity="center"
            android:orientation="vertical">
    
            <TextView
                android:id="@+id/id_widget_title_tv"
    
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="标题"/>
    
    
            <TextView
                android:id="@+id/id_widget_content_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="内容"/>
    
    
        </LinearLayout>
    
    
        <ImageButton
            android:id="@+id/id_widget_btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@mipmap/ic_launcher"/>
    </LinearLayout>
```

## 引用

- [官方文档](http://developer.android.com/intl/zh-cn/guide/topics/appwidgets/index.html)
- [设计指南](http://developer.android.com/intl/zh-cn/guide/practices/ui_guidelines/widget_design.html#anatomy_determining_size)