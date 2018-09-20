# IPC之ContentProvider

ContentProvider本来就是用来进行进程间通信的，通过另外一个进程提供的ContentProvider我们可以访问另外一个进程的数据，ContentProvider并不仅仅是可以操作数据库，其他数据都可以。只要我们愿意，可以随意实现它的五个方法。**需要注意的是除了onCreate方法，其他方法都运行在系统的binder线程池中**。

使用起来非常简单，代码如下：

配置：

```java
       <provider
                android:name=".provider.BookProvider"
                android:authorities="com.ztiany.androidipc.provider"
                android:permission="com.ztiany.androidipc.book.provider_access"
                android:process=":book_provider"/>
```

代码：

```java
    public class BookProvider extends ContentProvider {

        public static final int TABLE_NAME_BOOK = 1;
        public static final int TABLE_NAME_USER = 2;
        public static final String AUTHORITY = "com.ztiany.androidipc.provider";
        public static final String TABLE_BOOK = "book";
        public static final String TABLE_USER = "user";
        private final static Uri mBookUri = Uri.parse("content://" + AUTHORITY + "/book");
        private final static Uri mUserUri = Uri.parse("content://" + AUTHORITY + "/user");
    
        private final static UriMatcher mUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    
    //静态注册可以暴露的数据表格
        static {
            mUriMatcher.addURI(AUTHORITY, "book", TABLE_NAME_BOOK);
            mUriMatcher.addURI(AUTHORITY, "user", TABLE_NAME_USER);
        }
    
    
        @Override
        public boolean onCreate() {
            Log.d("BookProvider", "onCreate");
            return true;
        }
    
        @Nullable
        @Override
        public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    
            String tableName = getTableName(uri);
    
            if (uri == null) {
                throw new IllegalArgumentException("Unsupported uri");
            }
    
            return null;
        }
    
        /**
         * @param uri
         * @return 返回一个Uri请求所对应的MIME类型，
         */
        @Nullable
        @Override
        public String getType(Uri uri) {
            String typeName = null;
            int match = mUriMatcher.match(uri);
            switch (match) {
                case TABLE_NAME_BOOK:
                    typeName = "text/plain";
                    break;
                case TABLE_NAME_USER:
                    typeName = "image/jpeg";
                    break;
            }
            return typeName;
        }
    
        @Nullable
        @Override
        public Uri insert(Uri uri, ContentValues values) {
            return null;
        }
    
        @Override
        public int delete(Uri uri, String selection, String[] selectionArgs) {
            return 0;
        }
    
        @Override
        public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
            return 0;
        }
    
    
        public String getTableName(Uri uri) {
            String name = null;
            int match = mUriMatcher.match(uri);
            switch (match) {
                case TABLE_NAME_BOOK:
                    name = TABLE_BOOK;
                    break;
                case TABLE_NAME_USER:
                    name = TABLE_USER;
                    break;
            }
            return name;
        }
    }
```