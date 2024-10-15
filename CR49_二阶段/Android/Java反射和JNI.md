
# 反射
Java 的反射机制是指在运行时动态地获取类的信息，并可以在运行时调用类的方法、访问类的属性等。

使用 `static { System.loadLibrary("libname"); }` 加载动态库。

* 反射常用API：
  * Class.forName(String className)：根据类名获取类对象
  * Class.newInstance()：根据类对象创建实例
  * Class.getMethod(String name, Class... parameterTypes)：根据方法名和参数类型获取方法对象
  * Method.invoke(Object obj, Object... args)：调用方法
  * Field.get(Object obj)：获取属性值
  * Field.set(Object obj, Object value)：设置属性值
  * Constructor.newInstance(Object... args)：根据参数类型和值创建实例

``` Java
// 类对象
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

// 获取类对象
Class cFooClass = Class.forName("com.example.CFoo");
// 创建实例 - 无参数构造函数
CFoo cFooObj = (CFoo)cFooClass.newInstance();
// 创建示例 - 带参数构造函数
Constructor constructor = CFoo.getConstructor(String.class);
CFoo cFooObj2 = (CFoo)constructor.newInstance("hello");
// 调用方法
Method method = cFooClass.getMethod("foo");
method.invoke(cFooObj);
method.invoke(cFooObj2, "world");
// 获取属性值
Field field = cFooClass.getField("n");
int n = (int)field.get(cFooObj);
// 设置属性值
field.set(cFooObj, 20);
```

# JNI

https://docs.oracle.com/en/java/javase/23/docs/specs/jni/index.html

Java Native Interface（Java本地接口）是一种为 Java 编程语言提供调用非 Java 代码的机制。

使用 JAVA 声明，C/C++ 实现。

* 步骤：
* 1. 编写 Java 代码，声明 native 方法。
* 2. 使用 `javac -h. -d. ClassName.java` 生成 JNI 头文件。
* 3. 使用 `gcc -shared -o libClassName.so ClassName.cpp` 生成 JNI 库。
* 4. 使用 `System.loadLibrary("libClassName");` 加载 JNI 库。
* 5. 调用 native 方法。

``` Java
public class JniDemo {
    static {
        System.loadLibrary("JniDemo");
    }

    // C++ 实现对应的 native 方法
    public static native String sayHello();

    public static void main(String[] args) {
        // 调用 native 方法
        sayHello();
    }
}
```

``` C++
#include <jni.h>
#include <string>

extern "C" // 名称粉碎
JNIEXPORT  // 导出函数
jstring    // 返回值类型
Java_com_example_JniDemo_sayHello( // Java_报名_类名_方法名
JNIEnv* env, // JNI 环境
jobject thiz // 调用该方法的 Java 对象
) {
    std::string hello = "Hello from C++";

    return env->NewStringUTF(hello.c_str()); // 返回 Java 字符串
}
```

## JNI_OLoad 和 JNI_OnUnload
`JNI_OnLoad` 和 `JNI_OnUnload` 是两个函数，它们分别在 Java 虚拟机加载和卸载时被调用。

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

Native Development Kit（NDK）是 Android 平台上的开发工具包，它是一套基于 C/C++ 的工具集，可以用来开发 Android 应用的本地代码。

在Android的主活动中声明和调用 C++ 代码。

``` Java
public class MainActivity extends AppCompatActivity {
    // 启动时加载 native 库
    static {
        System.loadLibrary("my_native_lib");
    }
    
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 绑定布局
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // 调用控件的点击事件
        binding.button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 调用 native 方法
                SayHello();
                strTest();
            }
        });
    }

    // 声明 native 方法
    public native void SayHello();
    // 字符串操作
    public native String strTest(String str);
    // 通过 native 调用反射对象
    public native void reflectObject();
}
```

使用 C++ 实现 native 方法。

``` C++
#include <jni.h>
#include <string>
#include <android/log.h>

extern "C"
JNIEXPORT void JNICALL
Java_com_example_jnitest_MainActivity_sayHello(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++";
    // 打印日志
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "sayHello: %s", hello.c_str());
    return;
}

extern "C"
JNIEXPORT jstring JNICALL
Java_com_example_jnitest_MainActivity_strTest(JNIEnv *env, jobject thiz, jstring str) {
    // 获取 Java 字符串
    const char *sz = env->GetStringUTFChars(str, nullptr);
    // 字符串操作
    std::string str_cpp = sz;
    // 打印日志
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "strTest: %s", str_cpp.c_str());
    // 释放 Java 字符串
    env->ReleaseStringUTFChars(str, sz);
    // 转换为 Java 字符串
    return env->NewStringUTF(str_cpp.c_str());
}

extern "C"
JNIEXPORT void JNICALL
Java_com_example_jnitest_MainActivity_reflectObject(JNIEnv *env, jobject thiz) {
    // 获取类对象
    jclass cFooClass = env->FindClass("com/example/CFoo");
    // 创建实例 - 无参数构造函数
    jobject cFooObj = env->NewObject(cFooClass, env->GetMethodID(cFooClass, "<init>", "()V"));
    // 调用方法 GetMethodID 第一个参数是类对象，第二个参数是方法名，第三个参数是方法签名
    jmethodID methodId = env->GetMethodID(cFooClass, "foo", "()V");
    env->CallVoidMethod(cFooObj, methodId);
    // 释放实例
    env->DeleteLocalRef(cFooObj);
    // 释放类对象
    env->DeleteLocalRef(cFooClass);
    return;
}
```

## 动态注册
https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/functions.html#RegisterNatives

方法签名：
https://docs.oracle.com/en/java/javase/23/docs/specs/jni/types.html#field-and-method-ids

Java 可以使用动态注册机制来注册 native 方法：
1. 在使用 `System.loadLibrary("libname");` 加载 JNI 库时会调用 `JNI_OnLoad` 函数，在该函数中调用 `RegisterNatives` 函数注册 native 方法。
2. 调用 `RegisterNatives` 函数时，需要提供一个 `JNINativeMethod` 数组，其中包含 native 方法的签名、方法名和方法地址。
3. `JNINativeMethod` 数组的元素个数必须与 native 方法的个数一致。

``` C++
#include <jni.h>
#include <string>
#include <android/log.h>

// 实现注册后的方法
void sayHello(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++";
    // 打印日志
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "sayHello: %s", hello.c_str());
    return;
}

jstring strTest(JNIEnv *env, jobject thiz, jstring str) {
    // 获取 Java 字符串
    const char *sz = env->GetStringUTFChars(str, nullptr);
    // 字符串操作
    std::string str_cpp = sz;
    // 打印日志
    __android_log_print(ANDROID_LOG_VERBOSE, "JNITest", "strTest: %s", str_cpp.c_str());
    // 释放 Java 字符串
    env->ReleaseStringUTFChars(str, sz);
    // 转换为 Java 字符串
    return env->NewStringUTF(str_cpp.c_str());
}

void reflectObject(JNIEnv *env, jobject thiz) {
    // 获取类对象
    jclass cFooClass = env->FindClass("com/example/CFoo");
    // 创建实例 - 无参数构造函数
    jobject cFooObj = env->NewObject(cFooClass, env->GetMethodID(cFooClass, "<init>", "()V"));
    // 调用方法 GetMethodID 第一个参数是类对象，第二个参数是方法名，第三个参数是方法签名
    jmethodID methodId = env->GetMethodID(cFooClass, "foo", "()V");
    env->CallVoidMethod(cFooObj, methodId);
    // 释放实例
    env->DeleteLocalRef(cFooObj);
    // 释放类对象
    env->DeleteLocalRef(cFooClass);
    return;
}

JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    // 获取 JNI 环境
    JNIEnv *env = nullptr;
    vm->GetEnv((void **)&env, JNI_VERSION_1_6);

    // 查找 MainActivity 类
    jclass activityClass = env->FindClass("com/example/jnitest/MainActivity");

    // 注册 native 方法
    static JNINativeMethod methods[] = {
            {"sayHello", "()V", (void*)sayHello},
            {"strTest", "(Ljava/lang/String;)Ljava/lang/String;", (jstring*)strTest},
            {"reflectObject", "()V", (void*)reflectObject}
    };
    env->RegisterNatives(activityClass, methods, sizeof(methods) / sizeof(methods[0]));

    // 释放 MainActivity 类
    env->DeleteLocalRef(activityClass);

    return JNI_VERSION_1_6;
}
```
