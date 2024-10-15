- [内容提供者（Content provider）](#内容提供者content-provider)
  - [创建内容提供者](#创建内容提供者)
  - [访问内容提供者](#访问内容提供者)
  - [内容观察者](#内容观察者)
  - [访问联系人](#访问联系人)
  - [访问短信](#访问短信)

# 内容提供者（Content provider）

https://developer.android.google.cn/guide/topics/providers/content-providers?hl=zh

Content provider 是将自己指定的数据共享给其他应用程序的一种方式，是四大组件之一，需要在清单文件里面声明。

内容 URI 用来在内容提供程序中标识数据。内容 URI 包含整个内容提供程序的名称以及路径。例如 `mytable` 是内容提供者的表，则 `mytable` 表的完整 URI 是：

```
content://com.example.android.test/mytable
```

## 创建内容提供者
创建内容提供者需要继承 `ContentProvider` 类，并重写以下方法：
  * `onCreate()`：创建内容提供者时调用。
    * 创建 SQLite 数据库，将数据存储到 SQLite 数据库中。
  * `query()`：查询内容提供者中的数据。
  * `insert()`：向内容提供者中插入数据。
  * `update()`：更新内容提供者中的数据。
  * `delete()`：删除内容提供者中的数据。
  * `getType()`：获取内容提供者的数据类型。

``` Java
public class MyContentProvider extends ContentProvider {

    // 内容 Uri
    public static final UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    // 内容 Uri 匹配码
    public static final int MATCH_CODE = 1;
    static {
        // 匹配 content://com.example.android.test/mytable 的内容 Uri
        uriMatcher.addURI("com.example.android.test", "mytable", MATCH_CODE);
    }

    // 创建数据库
    class MySqliteHelper extends SQLiteOpenHelper {

        public MySqliteHelper(Context context) {
            super(context, "my.db", null, 1); // 建库
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("create table mytable (id varchar(255) primary key, name varchar(255))"); // 建表
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        }
    }

    // 创建 SQLite 数据库
    private MySqliteHelper helper = null;

    @Override
    public boolean onCreate() {
        helper = new MySqliteHelper(getContext());
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        // 查看内容 Uri 是否匹配
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // 查询数据库
            SQLiteDatabase db = helper.getReadableDatabase();
            Cursor cursor = db.query("mytable", projection, selection, selectionArgs, null, null, sortOrder);
            return cursor;
        }
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // 查看内容 Uri 是否匹配
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // 插入数据库
            SQLiteDatabase db = helper.getWritableDatabase();
            db.insert("mytable", null, values);
            return uri;
        }
        return null;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        // 查看内容 Uri 是否匹配
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // 更新数据库
            SQLiteDatabase db = helper.getWritableDatabase();
            return db.update("mytable", values, selection, selectionArgs);
        }
        return 0;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // 查看内容 Uri 是否匹配
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // 删除数据库
            SQLiteDatabase db = helper.getWritableDatabase();
            return db.delete("mytable", selection, selectionArgs);
        }
        return 0;
    }

    @Override
    public String getType(Uri uri) {
        // 查看内容 Uri 是否匹配
        if (uriMatcher.match(uri) == MATCH_CODE) {
            return "vnd.android.cursor.dir/vnd.com.example.android.test.mytable";
        }
        return null;
    }
}
```

清单文件中声明内容提供者：
``` XML
<provider
    android:authorities="com.example.android.test"
    android:name=".MyContentProvider"
    android:enabled="true"
    android:exported="true" />
```

## 访问内容提供者
使用 `ContentResolver` 类来访问内容提供者。

``` Java
// 内容 Uri
Uri uri = Uri.parse("content://com.example.android.test/mytable");


// 访问内容提供者
ContentResolver resolver = getContentResolver();

// 向内容提供者中插入数据
ContentValues values = new ContentValues();
values.put("id", "1");
values.put("name", "张三");
resolver.insert(uri, values);

// 从内容提供者中查询数据
Cursor cursor = resolver.query(uri, null, null, null, null);
while (cursor.moveToNext()) {
    String id = cursor.getString(cursor.getColumnIndex("id"));
    String name = cursor.getString(cursor.getColumnIndex("name"));
    System.out.println("id: " + id + ", name: " + name);
}

// 从内容提供者中删除数据
resolver.delete(uri, "id=?", new String[]{"1"});

// 从内容提供者中更新数据
ContentValues values = new ContentValues();
values.put("name", "李四");
resolver.update(uri, values, "id=?", new String[]{"1"});
```

## 内容观察者
在内容提供者中设置回调，当内容提供者中的数据发生变化时，会调用回调。
* 在内容提供者中调用 `getContentResolver().notifyChange(uri, null)` 方法来通知内容观察者。
* 在内容观察者中调用 `registerContentObserver` 方法来注册内容观察者。

内容提供者
``` Java  
public Uri insert(Uri uri, ContentValues values) {
    if (uriMatcher.match(uri) == MATCH_CODE) {
        SQLiteDatabase db = helper.getWritableDatabase();
        db.insert("mytable", null, values);
        // 通知内容观察者
        getContentResolver().notifyChange(uri, null);
        return uri;
    }
    return null;
}
```

内容观察者
``` Java
// 注册内容观察者
resolver.registerContentObserver(Uri.parse("content://com.example.android.test/mytable"), true, new ContentObserver(new Handler()) {
    @Override
    public void onChange(boolean selfChange) {
        super.onChange(selfChange);
        // 内容提供者中的数据发生变化时，会调用该方法
    }
});
```

## 访问联系人
https://developer.android.google.cn/guide/topics/providers/contacts-provider?hl=zh-cn

``` Java
// 访问联系人
ContentResolver resolver = getContentResolver();
Cursor cursor = resolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
while (cursor.moveToNext()) {
    String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
    String phone = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
    System.out.println("name: " + name + ", phone: " + phone);
}
```

## 访问短信

基于短信消息的操作：
https://developer.android.google.cn/reference/android/provider/Telephony.Sms

短信字段信息：
https://developer.android.google.cn/reference/android/provider/Telephony.TextBasedSmsColumns
* `_COUNT`：短信的数量。
* `_id`：短信的 ID。
* `address`：对方的电话号码。
* `body`：短信的内容。
* `date`：短信的时间。
* `read`：短信是否已读。

``` Java
// 访问短信
ContentResolver resolver = getContentResolver();

// 获取当前默认的短信应用
String defaultSmsApp = Telephony.Sms.getDefaultSmsPackage(this);
System.out.println("defaultSmsApp: " + defaultSmsApp);

// 获取所有短信
Cursor cursor = resolver.query(Telephony.Sms.CONTENT_URI, null, null, null, null);
while (cursor.moveToNext()) {
    String address = cursor.getString(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.ADDRESS));
    String body = cursor.getString(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.BODY));
    long date = cursor.getLong(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.DATE));
    int read = cursor.getInt(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.READ));
    System.out.println("address: " + address + ", body: " + body + ", date: " + date + ", read: " + read);
}

// 发送短信
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage("10086", null, "Hello", null, null);
```