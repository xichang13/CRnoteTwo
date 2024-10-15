
# 将Java工程改为C++工程

在 `build.gradle.kts` 文件中，添加如下代码：
``` Java
externalNativeBuild {
    cmake {
        // 这里是CMakeLists.txt文件所在的路径
        path = file("CMakeLists.txt")
        // CMake版本
        version = "3.10.2"
    }
}

externalNativeBuild {
    cmake {
        cppFlags("-std=c++11") // 添加C++11编译选项
        }
    }
}
```

`CMakeLists.txt` 是功能库的CMake配置文件，在 `CMakeLists.txt` 文件中，添加如下代码：
``` CMake
// CMake版本
cmake_minimum_required(VERSION 3.10.2)

// 项目名称
project(cr49_native)

// 生成动态库 类似 DLL
add_library(cr49_native SHARED src/main.cpp)
// 链接依赖库 类似 #programe comment(lib, libname)
target_link_libraries(cr49_native android log)

// 生成可执行文件 类似 EXE 可以在命令行运行
add_executable(cr49_native_test src/main.cpp)
```

`src/main.cpp` 是功能库的源代码文件，在 `src/main.cpp` 文件中，添加如下代码：
``` C++
#include <jni.h>
#include <string>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_cr49_MainActivity_stringFromJNI(JNIEnv *env, jobject thiz) {
    std::string hello = "Hello from C++!";
    return env->NewStringUTF(hello.c_str());
}
```

编译后会生成 `cr49_native.so` 文件，该文件是功能库的动态库。

## 使用 `cr49_native.so` 库
使用 `System.loadLibrary("cr49_native");` 加载 `cr49_native.so` 库，并调用 `Java_com_example_cr49_MainActivity_stringFromJNI` 方法，即可调用功能库的 `stringFromJNI` 方法。