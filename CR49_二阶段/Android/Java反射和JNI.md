
# ����
Java �ķ��������ָ������ʱ��̬�ػ�ȡ�����Ϣ��������������ʱ������ķ���������������Եȡ�

ʹ�� `static { System.loadLibrary("libname"); }` ���ض�̬�⡣

* ���䳣��API��
  * Class.forName(String className)������������ȡ�����
  * Class.newInstance()����������󴴽�ʵ��
  * Class.getMethod(String name, Class... parameterTypes)�����ݷ������Ͳ������ͻ�ȡ��������
  * Method.invoke(Object obj, Object... args)�����÷���
  * Field.get(Object obj)����ȡ����ֵ
  * Field.set(Object obj, Object value)����������ֵ
  * Constructor.newInstance(Object... args)�����ݲ������ͺ�ֵ����ʵ��

``` Java
// �����
public class CFoo {
    public CFoo() {
        System.out.println("CFoo()");
    }

    public CFoo(String str) {
        System.out.println("CFoo(String)");
    }

    public void foo() {
        System.out.println("CFoo.foo()");
    }

    public String foo(String str) {
        System.out.println("CFoo.foo(String)");
    }

    public int n = 10;
}

// ��ȡ�����
Class cFooClass = Class.forName("com.example.CFoo");
// ����ʵ�� - �޲������캯��
CFoo cFooObj = (CFoo)cFooClass.newInstance();
// ����ʾ�� - ���������캯��
Constructor constructor = CFoo.getConstructor(String.class);
CFoo cFooObj2 = (CFoo)constructor.newInstance("hello");
// ���÷���
Method method = cFooClass.getMethod("foo");
method.invoke(cFooObj);
method.invoke(cFooObj2, "world");
// ��ȡ����ֵ
Field field = cFooClass.getField("n");
int n = (int)field.get(cFooObj);
// ��������ֵ
field.set(cFooObj, 20);
```

# JNI

https://docs.oracle.com/en/java/javase/23/docs/specs/jni/index.html

Java Native Interface��Java���ؽӿڣ���һ��Ϊ Java ��������ṩ���÷� Java ����Ļ��ơ�

ʹ�� JAVA ������C/C++ ʵ�֡�

* ���裺
* 1. ��д Java ���룬���� native ������
* 2. ʹ�� `javac -h. -d. ClassName.java` ���� JNI ͷ�ļ���
* 3. ʹ�� `gcc -shared -o libClassName.so ClassName.cpp` ���� JNI �⡣
* 4. ʹ�� `System.loadLibrary("libClassName");` ���� JNI �⡣
* 5. ���� native ������

``` Java
public class JniDemo {
    static {
        System.loadLibrary("JniDemo");
    }

    // C++ ʵ�ֶ�Ӧ�� native ����
    public static native String sayHello();

    public static void main(String[] args) {
        // ���� native ����
        sayHello();
    }
}
```

``` C++
#include <jni.h>
#include <string>

extern "C" // ���Ʒ���
JNIEXPORT  // ��������
jstring    // ����ֵ����
Java_com_example_JniDemo_sayHello( // Java_����_����_������
JNIEnv* env, // JNI ����
jobject thiz // ���ø÷����� Java ����
) {
    std::string hello = "Hello from C++";

    return env->NewStringUTF(hello.c_str()); // ���� Java �ַ���
}
```

## JNI_OLoad �� JNI_OnUnload
`JNI_OnLoad` �� `JNI_OnUnload` ���������������Ƿֱ��� Java ��������غ�ж��ʱ�����á�

``` C++
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    System.out.println("JNI_OnLoad");
    return JNI_VERSION_1_6;
}

JNIEXPORT void JNI_OnUnload(JavaVM *vm, void *reserved) {
    System.out.println("JNI_OnUnload");
}
```

# NDK

https://developer.android.google.cn/ndk/reference

Native Development Kit��NDK���� Android ƽ̨�ϵĿ������߰�������һ�׻��� C/C++ �Ĺ��߼��������������� Android Ӧ�õı��ش��롣

��Android������������͵��� C++ ���롣

``` Java
public class MainActivity extends AppCompatActivity {
    // ����ʱ���� native ��
    static {
        System.loadLibrary("my_native_lib");
    }
    
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // �󶨲���
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // ���ÿؼ��ĵ���¼�
        binding.button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // ���� native ����
                SayHello();
                strTest();
            }
        });
    }

    // ���� native ����
    public native void SayHello();
    // �ַ�������
    public native String strTest(String str);
    // ͨ�� native ���÷������
    public native void reflectObject();
}
```

ʹ�� C++ ʵ�� native ������

``` C++
#include <jni.h>
#include <string>
#include <android/log.h>

extern "C"
JNIEXPORT void JNICALL
Java_com_example_jnitest_MainActivity_sayHello(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++";
    // ��ӡ��־
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "sayHello: %s", hello.c_str());
    return;
}

extern "C"
JNIEXPORT jstring JNICALL
Java_com_example_jnitest_MainActivity_strTest(JNIEnv *env, jobject thiz, jstring str) {
    // ��ȡ Java �ַ���
    const char *sz = env->GetStringUTFChars(str, nullptr);
    // �ַ�������
    std::string str_cpp = sz;
    // ��ӡ��־
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "strTest: %s", str_cpp.c_str());
    // �ͷ� Java �ַ���
    env->ReleaseStringUTFChars(str, sz);
    // ת��Ϊ Java �ַ���
    return env->NewStringUTF(str_cpp.c_str());
}

extern "C"
JNIEXPORT void JNICALL
Java_com_example_jnitest_MainActivity_reflectObject(JNIEnv *env, jobject thiz) {
    // ��ȡ�����
    jclass cFooClass = env->FindClass("com/example/CFoo");
    // ����ʵ�� - �޲������캯��
    jobject cFooObj = env->NewObject(cFooClass, env->GetMethodID(cFooClass, "<init>", "()V"));
    // ���÷��� GetMethodID ��һ������������󣬵ڶ��������Ƿ������������������Ƿ���ǩ��
    jmethodID methodId = env->GetMethodID(cFooClass, "foo", "()V");
    env->CallVoidMethod(cFooObj, methodId);
    // �ͷ�ʵ��
    env->DeleteLocalRef(cFooObj);
    // �ͷ������
    env->DeleteLocalRef(cFooClass);
    return;
}
```

## ��̬ע��
https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/functions.html#RegisterNatives

����ǩ����
https://docs.oracle.com/en/java/javase/23/docs/specs/jni/types.html#field-and-method-ids

Java ����ʹ�ö�̬ע�������ע�� native ������
1. ��ʹ�� `System.loadLibrary("libname");` ���� JNI ��ʱ����� `JNI_OnLoad` �������ڸú����е��� `RegisterNatives` ����ע�� native ������
2. ���� `RegisterNatives` ����ʱ����Ҫ�ṩһ�� `JNINativeMethod` ���飬���а��� native ������ǩ�����������ͷ�����ַ��
3. `JNINativeMethod` �����Ԫ�ظ��������� native �����ĸ���һ�¡�

``` C++
#include <jni.h>
#include <string>
#include <android/log.h>

// ʵ��ע���ķ���
void sayHello(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++";
    // ��ӡ��־
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "sayHello: %s", hello.c_str());
    return;
}

jstring strTest(JNIEnv *env, jobject thiz, jstring str) {
    // ��ȡ Java �ַ���
    const char *sz = env->GetStringUTFChars(str, nullptr);
    // �ַ�������
    std::string str_cpp = sz;
    // ��ӡ��־
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "strTest: %s", str_cpp.c_str());
    // �ͷ� Java �ַ���
    env->ReleaseStringUTFChars(str, sz);
    // ת��Ϊ Java �ַ���
    return env->NewStringUTF(str_cpp.c_str());
}

void reflectObject(JNIEnv *env, jobject thiz) {
    // ��ȡ�����
    jclass cFooClass = env->FindClass("com/example/CFoo");
    // ����ʵ�� - �޲������캯��
    jobject cFooObj = env->NewObject(cFooClass, env->GetMethodID(cFooClass, "<init>", "()V"));
    // ���÷��� GetMethodID ��һ������������󣬵ڶ��������Ƿ������������������Ƿ���ǩ��
    jmethodID methodId = env->GetMethodID(cFooClass, "foo", "()V");
    env->CallVoidMethod(cFooObj, methodId);
    // �ͷ�ʵ��
    env->DeleteLocalRef(cFooObj);
    // �ͷ������
    env->DeleteLocalRef(cFooClass);
    return;
}

JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    // ��ȡ JNI ����
    JNIEnv *env = nullptr;
    vm->GetEnv((void **)&env, JNI_VERSION_1_6);

    // ���� MainActivity ��
    jclass activityClass = env->FindClass("com/example/jnitest/MainActivity");

    // ע�� native ����
    static JNINativeMethod methods[] = {
            {"sayHello", "()V", (void*)sayHello},
            {"strTest", "(Ljava/lang/String;)Ljava/lang/String;", (jstring*)strTest},
            {"reflectObject", "()V", (void*)reflectObject}
    };
    env->RegisterNatives(activityClass, methods, sizeof(methods) / sizeof(methods[0]));

    // �ͷ� MainActivity ��
    env->DeleteLocalRef(activityClass);

    return JNI_VERSION_1_6;
}
```
