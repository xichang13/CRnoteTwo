
# ��Java���̸�ΪC++����

�� `build.gradle.kts` �ļ��У�������´��룺
``` Java
externalNativeBuild {
    cmake {
        // ������CMakeLists.txt�ļ����ڵ�·��
        path = file("CMakeLists.txt")
        // CMake�汾
        version = "3.10.2"
    }
}

externalNativeBuild {
    cmake {
        cppFlags("-std=c++11") // ���C++11����ѡ��
        }
    }
}
```

`CMakeLists.txt` �ǹ��ܿ��CMake�����ļ����� `CMakeLists.txt` �ļ��У�������´��룺
``` CMake
// CMake�汾
cmake_minimum_required(VERSION 3.10.2)

// ��Ŀ����
project(cr49_native)

// ���ɶ�̬�� ���� DLL
add_library(cr49_native SHARED src/main.cpp)
// ���������� ���� #programe comment(lib, libname)
target_link_libraries(cr49_native android log)

// ���ɿ�ִ���ļ� ���� EXE ����������������
add_executable(cr49_native_test src/main.cpp)
```

`src/main.cpp` �ǹ��ܿ��Դ�����ļ����� `src/main.cpp` �ļ��У�������´��룺
``` C++
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_cr49_MainActivity_stringFromJNI(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++!";
    return env->NewStringUTF(hello.c_str());
}
```

���������� `cr49_native.so` �ļ������ļ��ǹ��ܿ�Ķ�̬�⡣

## ʹ�� `cr49_native.so` ��
ʹ�� `System.loadLibrary("cr49_native");` ���� `cr49_native.so` �⣬������ `Java_com_example_cr49_MainActivity_stringFromJNI` ���������ɵ��ù��ܿ�� `stringFromJNI` ������