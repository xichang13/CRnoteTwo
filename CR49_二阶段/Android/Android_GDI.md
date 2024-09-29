- [Android GDI (Graphics Device Interface) ͼ���豸�ӿ�](#android-gdi-graphics-device-interface-ͼ���豸�ӿ�)
  - [View](#view)
  - [����ͼ��](#����ͼ��)
  - [����λͼ](#����λͼ)
  - [������Ӧ](#������Ӧ)
  - [�����](#�����)
  - [������Ƶ](#������Ƶ)
- [������Ϸ�߼�](#������Ϸ�߼�)
  - [֡����](#֡����)
  - [��������ͼ](#��������ͼ)


# Android GDI (Graphics Device Interface) ͼ���豸�ӿ�
## View
* ����һ���̳��� `android.view.View` ���࣬��д `onDraw` ������
``` Java
public class MyView extends View {
    public MyView(Context context) {
        super(context);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // ��������л�ͼ����
    }
}
```
* �ڻ��ʹ���Զ���� `View`��
``` Java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(new MyView(this));
    }
}
```
## ����ͼ��
    https://developer.android.google.cn/reference/android/graphics/Canvas?hl=en#public-methods
* �� `onDraw` ������ʹ�� `Canvas` ��ͼ��
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����һ��Բ��
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    canvas.drawCircle(100, 100, 50, paint);
}
```
* �� `onDraw` �����е��� `invalidate` �������ᴥ�� `onDraw` �����ĵ��ã�ʵ�ֻ�ͼ��ѭ����
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����һ��Բ��
    Paint paint = new Paint();
    paint.setColor(Color.RED);
    canvas.drawCircle(100, 100, 50, paint);
    // ���� onDraw �����ĵ���
    invalidate();
}
```

## ����λͼ
* �� `onDraw` ������ʹ�� `Bitmap` ����λͼ��
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����λͼ
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.my_image);
    // ����λͼ
    canvas.drawBitmap(bitmap, 0, 0, null);
}
```
* λͼ���У����� `Bitmap` �� `createBitmap` ������
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����λͼ
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.my_image);
    // ����λͼ��һ����
    Bitmap croppedBitmap = Bitmap.createBitmap(bitmap, 100, 100, 200, 200);
    // ���Ƽ��к��λͼ
    canvas.drawBitmap(croppedBitmap, 0, 0, null);
}
```
* λͼ���ţ����� `Bitmap` �� `createScaledBitmap` ������
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����λͼ
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.my_image);
    // ����λͼ
    Bitmap scaledBitmap = Bitmap.createScaledBitmap(bitmap, 200, 200, true);
    // �������ź��λͼ
    canvas.drawBitmap(scaledBitmap, 0, 0, null);
}
```
* ����λͼ��һ���֡�
``` Java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // ����λͼ
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.my_image);
    // ����λͼ��һ����
    Rect srcRect = new Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
    Rect dstRect = new Rect(0, 0, getWidth(), getHeight());
    canvas.drawBitmap(bitmap, srcRect, dstRect, null);
}
```

## ������Ӧ
* ���� `View` �� `setOnTouchListener` ���������� `View` �Ĵ����¼����������� `onTouch` �����д������¼���
``` Java
// ���ü�����
public class MyView extends View {
    public MyView(Context context) {
        super(context);
        setOnTouchListener(this);
    }

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        // �������¼�
        float x = event.getX();
        float y = event.getY();
        // �����������Ӧ�Ĳ����������ƶ�������ͼ��
        return true;
    }
}
```

## �����
* ���� `Random` ��� `nextInt` �����������������
``` Java
Random random = new Random();
int randomNumber = random.nextInt(100); // ���� 0 �� 99 ֮��������
```

## ������Ƶ
* ���� `MediaPlayer` ��� `create` ���������� `MediaPlayer` ���� ���� `start` ������������Ƶ������ `setLooping(true)` ����������ѭ�����ţ� ���� `stop` ������ֹͣ���š�
``` Java
// ���� MediaPlayer ����
MediaPlayer mediaPlayer = MediaPlayer.create(this, R.raw.my_audio);
// ������Ƶ
mediaPlayer.start();
// ����ѭ������
mediaPlayer.setLooping(true);
// ֹͣ����
mediaPlayer.stop();
```

# ������Ϸ�߼�
## ֡����
* �� `onDraw` �����У����� `invalidate` ���������� `onDraw` �����ĵ��ã�ʵ��֡���ƣ�60 ֡��ÿ��ˢ�� 60 �Ρ�
``` Java
@Override
public void run(){
    // ��ǰʱ��
    long currentTime = System.currentTimeMillis();
    // ÿ����� 60 ��
    while (true) {
        if (currentTime - lastTime >= 1000 / 60) {
            // ����һ֡
            invalidate();
            // ����ʱ��
            lastTime = currentTime;
        }
        // ��ǰʱ��
        currentTime = System.currentTimeMillis();
    }
}
```

## ��������ͼ
* �� `onDraw` �����У����Ʊ���ͼ�������걳��ͼ�󣬽�����ͼ��λ�������ƶ����ٽ��ƶ�����Ļ��ı���ͼ���»��Ƶ���Ļ�Ϸ���
``` Java
// ��������ͼ
private int bgY = 0;
@Override
public void run(){
    // ��ǰʱ��
    long currentTime = System.currentTimeMillis();
    // ÿ����� 60 ��
    while (true) {
        if (currentTime - lastTime >= 1000 / 60) {
            // ����һ֡
            invalidate();
            // ����ʱ��
            lastTime = currentTime;
            // ����ͼ�����ƶ�
            bgY += 1;
            // ������ͼ�ƶ�����Ļ��ʱ���������»��Ƶ���Ļ�Ϸ�
            if (bgY >= getHeight()) {
                bgY = -getHeight();
            }
        }
        // ��ǰʱ��
        currentTime = System.currentTimeMillis();
    }
}
```