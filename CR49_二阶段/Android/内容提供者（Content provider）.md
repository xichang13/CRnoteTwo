- [�����ṩ�ߣ�Content provider��](#�����ṩ��content-provider)
  - [���������ṩ��](#���������ṩ��)
  - [���������ṩ��](#���������ṩ��)
  - [���ݹ۲���](#���ݹ۲���)
  - [������ϵ��](#������ϵ��)
  - [���ʶ���](#���ʶ���)

# �����ṩ�ߣ�Content provider��

https://developer.android.google.cn/guide/topics/providers/content-providers?hl=zh

Content provider �ǽ��Լ�ָ�������ݹ��������Ӧ�ó����һ�ַ�ʽ�����Ĵ����֮һ����Ҫ���嵥�ļ�����������

���� URI �����������ṩ�����б�ʶ���ݡ����� URI �������������ṩ����������Լ�·�������� `mytable` �������ṩ�ߵı��� `mytable` ������� URI �ǣ�

```
content://com.example.android.test/mytable
```

## ���������ṩ��
���������ṩ����Ҫ�̳� `ContentProvider` �࣬����д���·�����
  * `onCreate()`�����������ṩ��ʱ���á�
    * ���� SQLite ���ݿ⣬�����ݴ洢�� SQLite ���ݿ��С�
  * `query()`����ѯ�����ṩ���е����ݡ�
  * `insert()`���������ṩ���в������ݡ�
  * `update()`�����������ṩ���е����ݡ�
  * `delete()`��ɾ�������ṩ���е����ݡ�
  * `getType()`����ȡ�����ṩ�ߵ��������͡�

``` Java
public class MyContentProvider extends ContentProvider {

    // ���� Uri
    public static final UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    // ���� Uri ƥ����
    public static final int MATCH_CODE = 1;
    static {
        // ƥ�� content://com.example.android.test/mytable ������ Uri
        uriMatcher.addURI("com.example.android.test", "mytable", MATCH_CODE);
    }

    // �������ݿ�
    class MySqliteHelper extends SQLiteOpenHelper {

        public MySqliteHelper(Context context) {
            super(context, "my.db", null, 1); // ����
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("create table mytable (id varchar(255) primary key, name varchar(255))"); // ����
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        }
    }

    // ���� SQLite ���ݿ�
    private MySqliteHelper helper = null;

    @Override
    public boolean onCreate() {
        helper = new MySqliteHelper(getContext());
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        // �鿴���� Uri �Ƿ�ƥ��
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // ��ѯ���ݿ�
            SQLiteDatabase db = helper.getReadableDatabase();
            Cursor cursor = db.query("mytable", projection, selection, selectionArgs, null, null, sortOrder);
            return cursor;
        }
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // �鿴���� Uri �Ƿ�ƥ��
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // �������ݿ�
            SQLiteDatabase db = helper.getWritableDatabase();
            db.insert("mytable", null, values);
            return uri;
        }
        return null;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        // �鿴���� Uri �Ƿ�ƥ��
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // �������ݿ�
            SQLiteDatabase db = helper.getWritableDatabase();
            return db.update("mytable", values, selection, selectionArgs);
        }
        return 0;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // �鿴���� Uri �Ƿ�ƥ��
        if (uriMatcher.match(uri) == MATCH_CODE) {
            // ɾ�����ݿ�
            SQLiteDatabase db = helper.getWritableDatabase();
            return db.delete("mytable", selection, selectionArgs);
        }
        return 0;
    }

    @Override
    public String getType(Uri uri) {
        // �鿴���� Uri �Ƿ�ƥ��
        if (uriMatcher.match(uri) == MATCH_CODE) {
            return "vnd.android.cursor.dir/vnd.com.example.android.test.mytable";
        }
        return null;
    }
}
```

�嵥�ļ������������ṩ�ߣ�
``` XML
<provider
    android:authorities="com.example.android.test"
    android:name=".MyContentProvider"
    android:enabled="true"
    android:exported="true" />
```

## ���������ṩ��
ʹ�� `ContentResolver` �������������ṩ�ߡ�

``` Java
// ���� Uri
Uri uri = Uri.parse("content://com.example.android.test/mytable");


// ���������ṩ��
ContentResolver resolver = getContentResolver();

// �������ṩ���в�������
ContentValues values = new ContentValues();
values.put("id", "1");
values.put("name", "����");
resolver.insert(uri, values);

// �������ṩ���в�ѯ����
Cursor cursor = resolver.query(uri, null, null, null, null);
while (cursor.moveToNext()) {
    String id = cursor.getString(cursor.getColumnIndex("id"));
    String name = cursor.getString(cursor.getColumnIndex("name"));
    System.out.println("id: " + id + ", name: " + name);
}

// �������ṩ����ɾ������
resolver.delete(uri, "id=?", new String[]{"1"});

// �������ṩ���и�������
ContentValues values = new ContentValues();
values.put("name", "����");
resolver.update(uri, values, "id=?", new String[]{"1"});
```

## ���ݹ۲���
�������ṩ�������ûص����������ṩ���е����ݷ����仯ʱ������ûص���
* �������ṩ���е��� `getContentResolver().notifyChange(uri, null)` ������֪ͨ���ݹ۲��ߡ�
* �����ݹ۲����е��� `registerContentObserver` ������ע�����ݹ۲��ߡ�

�����ṩ��
``` Java  
public Uri insert(Uri uri, ContentValues values) {
    if (uriMatcher.match(uri) == MATCH_CODE) {
        SQLiteDatabase db = helper.getWritableDatabase();
        db.insert("mytable", null, values);
        // ֪ͨ���ݹ۲���
        getContentResolver().notifyChange(uri, null);
        return uri;
    }
    return null;
}
```

���ݹ۲���
``` Java
// ע�����ݹ۲���
resolver.registerContentObserver(Uri.parse("content://com.example.android.test/mytable"), true, new ContentObserver(new Handler()) {
    @Override
    public void onChange(boolean selfChange) {
        super.onChange(selfChange);
        // �����ṩ���е����ݷ����仯ʱ������ø÷���
    }
});
```

## ������ϵ��
https://developer.android.google.cn/guide/topics/providers/contacts-provider?hl=zh-cn

``` Java
// ������ϵ��
ContentResolver resolver = getContentResolver();
Cursor cursor = resolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
while (cursor.moveToNext()) {
    String name = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
    String phone = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
    System.out.println("name: " + name + ", phone: " + phone);
}
```

## ���ʶ���

���ڶ�����Ϣ�Ĳ�����
https://developer.android.google.cn/reference/android/provider/Telephony.Sms

�����ֶ���Ϣ��
https://developer.android.google.cn/reference/android/provider/Telephony.TextBasedSmsColumns
* `_COUNT`�����ŵ�������
* `_id`�����ŵ� ID��
* `address`���Է��ĵ绰���롣
* `body`�����ŵ����ݡ�
* `date`�����ŵ�ʱ�䡣
* `read`�������Ƿ��Ѷ���

``` Java
// ���ʶ���
ContentResolver resolver = getContentResolver();

// ��ȡ��ǰĬ�ϵĶ���Ӧ��
String defaultSmsApp = Telephony.Sms.getDefaultSmsPackage(this);
System.out.println("defaultSmsApp: " + defaultSmsApp);

// ��ȡ���ж���
Cursor cursor = resolver.query(Telephony.Sms.CONTENT_URI, null, null, null, null);
while (cursor.moveToNext()) {
    String address = cursor.getString(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.ADDRESS));
    String body = cursor.getString(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.BODY));
    long date = cursor.getLong(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.DATE));
    int read = cursor.getInt(cursor.getColumnIndex(Telephony.TextBasedSmsColumns.READ));
    System.out.println("address: " + address + ", body: " + body + ", date: " + date + ", read: " + read);
}

// ���Ͷ���
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage("10086", null, "Hello", null, null);
```