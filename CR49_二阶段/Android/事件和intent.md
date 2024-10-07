- [�¼�](#�¼�)
  - [������](#������)
  - [�ص�](#�ص�)
  - [��Ϣ���У�Handler��](#��Ϣ����handler)
- [Intent](#intent)
  - [����������`Activity`�](#����������activity�)
  - [���ݴ���](#���ݴ���)
  - [��ʽIntent](#��ʽintent)

# �¼�
## ������
View�ṩ�˷ḻ�ļ������ӿڣ�����������û��Ľ�����
  * OnClickListener - ����¼�
  * OnLongClickListener - �����¼�
  * OnTouchListener - �����¼������Դ������������������Ȳ���
  * OnFocusChangeListener - ����ı��¼�
  * OnKeyListener - �����¼�
  * OnDragListener - ��ק�¼�
  * OnHoverListener - ��ͣ�¼�

## �ص�
### �����¼�
��`OnTouchListener`һ����`View`�ṩ��`onTouchEvent()`�������������¼���
``` Java
public class MyView extends View {
    public MyView(Context context) {
        super(context);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // �������¼�
        Log.v("MyView", event.toString());
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // ����
                break;
            case MotionEvent.ACTION_MOVE:
                // �ƶ�
                break;
            case MotionEvent.ACTION_UP:
                // ̧��
                break;
        }
        return true;
    }
}
```

### �����¼�
  * ���̻ص�����
    * `onKeyDown()` - ��������
    * `onKeyUp()` - ����̧��
  * ���̻ص���������Ҫ`View`�ܻ�ȡ���㣬����`setFocusable()`�������á�
``` Java
public class MyView extends View {
    public MyView(Context context) {
        super(context);
        // ���û�ȡ����
        setFocusable(true);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        // �����������¼�
        Log.v("MyView", "onKeyDown" + keyCode + " " + event.toString());
        return super.onKeyDown(keyCode, event);
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        // ������̧���¼�
        Log.v("MyView", "onKeyUp" + keyCode + " " + event.toString());
        return super.onKeyUp(keyCode, event);
    }
}
```

## ��Ϣ���У�Handler��
  * `Handler`��Android�д����첽��Ϣ�Ĺ��ߣ������Խ�һ�������л������߳������С�
  * `Handler`���Է��ͺʹ�����Ϣ��Ҳ����ִ����ʱ����
  * `Handler`���̺߳����߳�֮��������������Խ������л������߳���ִ�У��Ӷ������������߳��н���UI���������⡣
  * `Handler`��ʹ�ò��裺
    * ����`Handler`����
    * ��д`handleMessage()`������������յ�����Ϣ
    * ʹ��`sendEmptyMessage()`��`sendMessage()`����������Ϣ
``` Java
public class MyView extends View {
    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            // ������յ�����Ϣ
            Log.v("MyView", "handleMessage" + msg.what);
        }
    }

    public MyView(Context context) {
        super(context);
    }

    public void doSomething() {
        // ִ�к�ʱ����
        new Thread(new Runnable() {
            @Override
            public void run() {
                // ִ�к�ʱ����
                // ������Ϣ
                handler.sendEmptyMessage(0);
            }
        }).start();
    }
}
```

# Intent
  * `Intent`��Android����������Activity������Service�����͹㲥�Ȳ����Ĺ��ߡ�
  * `Intent`����Я�����ݣ�Ҳ����ָ��Ŀ�������

## ����������`Activity`�
������кܶ෽ʽ��
  * ʹ��`startActivity()`���������
  * ʹ��`startActivityForResult()`������������ȴ����ؽ��
    * �»�رպ󣬵�ǰ����յ�`onActivityResult()`�����Ļص�
  * ʹ��`startActivityIfNeeded()`������������ȴ����ؽ��
    * �»�رպ󣬵�ǰ����յ�`onActivityResult()`�����Ļص�
    * ����»�Ѿ����ڣ��򲻻����´����»������ֱ��ʹ���Ѵ��ڵĻ

###�����
  * ����һ���µĻ�࣬�̳���`Activity`��������
``` Java
public class MyActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);
    }
}
```
  * ��`AndroidManifest.xml`�ļ���ע��
``` XML
<activity android:name=".MyActivity" />
```
### ʹ��`startActivity()`���������
``` Java
Intent intent = new Intent(this, MyActivity.class);
startActivity(intent);
```
### ʹ��`startActivityForResult()`������������ȴ����ؽ��
  * ��������ȴ����ؽ��
``` Java
Intent intent = new Intent(this, MyActivity.class);
startActivityForResult(intent, 1);
```
  * ����������ؽ��
``` Java
public class MyActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        // ��ť����¼�
        findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // ���ؽ��
                Intent intent = new Intent();
                intent.putExtra("key", "value");
                setResult(RESULT_OK, intent);
                finish();
            }
        });
    }
}
```
  * �����ؽ��
``` Java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == 1 && resultCode == RESULT_OK) {
        // �����ؽ��
        String value = data.getStringExtra("key");
        Log.v("MyActivity", "requestCode: " + requestCode + " resultCode: " + resultCode + " value: " + value);
        }
    }
}
```

## ���ݴ���
�������������ַ�ʽ��
  * ʹ��`Intent`Я������
  * ʹ��`Bundle`Я������
### ʹ��`Intent`Я������
  * ��������
``` Java
Intent intent = new Intent(this, MyActivity.class);
intent.putExtra("key", "value");
startActivity(intent);
```
  * ��������
``` Java
// ��ȡIntent
Intent intent = getIntent();
// ��ȡ����
String value = intent.getStringExtra("key");
```
### ʹ��`Bundle`Я������
  * ��������
``` Java
Intent intent = new Intent(this, MyActivity.class);
// ��������
Bundle bundle = new Bundle();
bundle.putString("key", "value");
intent.putExtras(bundle);
startActivity(intent);
```
  * ��������
``` Java
// ��ȡIntent
Intent intent = getIntent();
// ��ȡ����
Bundle bundle = intent.getExtras();
String value = bundle.getString("key");
```

## ��ʽIntent
    ��ʽ�����������Ҫָ��Ŀ�������ֻ��Ҫָ���������������ͣ�ϵͳ����ݶ��������������Զ�ƥ��Ŀ��������������������ʵ�ֿ�Ӧ�õ����������
### ��������Ӧ�õĻ
  * �������
``` Java
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://baidu.com"));
startActivity(intent);
```
  * �򿪲�����
``` Java
Intent intent = new Intent(Intent.ACTION_DIAL, Uri.parse("tel:10086"));
startActivity(intent);
```
  * ���Ͷ���
``` Java
Intent intent = new Intent(Intent.ACTION_SENDTO, Uri.parse("smsto:10086"));
intent.putExtra("sms_body", "Hello World");
startActivity(intent);
```

### ������ʽ��ʽ�����Ļ
  * ����һ��������
``` Java
public class BrowserActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_browser);

        // ��ȡIntent
        Intent intent = getIntent();
        // ��ȡ����
        String url = intent.getData().toString();
        // ������ҳ
        WebView webView = findViewById(R.id.webView);
        webView.loadUrl(url);
    }
}
```
  * ��`AndroidManifest.xml`�ļ���ע��
``` XML
<activity android:name=".BrowserActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="http" />
        <data android:scheme="https" />
    </intent-filter>
</activity>
```
  * �����Զ����������
``` Java
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://baidu.com"));
startActivity(intent);
```