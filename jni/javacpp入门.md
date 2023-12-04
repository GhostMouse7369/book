# javacpp入门

windows

#### C++代码

##### 头文件: hello.h

```c++
#ifndef JDLIB_HELLO_H
#define JDLIB_HELLO_H

namespace jdlib {
    class Hello
    {
    public:
        Hello();

        ~Hello();

        void sayHello();
    };
}

#endif //JDLIB_HELLO_H
```
##### 源文件: hello.cpp

```c++
#include "hello.h"
#include<iostream>

using namespace std;

namespace jdlib {

    Hello::Hello()
    {
        printf("Create Hello\n");
    }

    Hello::~Hello()
    {
        printf("Delete Hello\n");
    }

    void Hello::sayHello()
    {
        printf("Hello, Jdlib!\n");
    }

}
```

##### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(jdlib)

set(CMAKE_CXX_STANDARD 14)

include_directories(./include)
include_directories(./src)

aux_source_directory(./src DIR_SRCS)

#add_library(dlib SHARED ${DIR_SRCS})
add_library(dlib STATIC ${DIR_SRCS})

#link_directories(./lib)
#add_executable(${PROJECT_NAME} main.cpp)
#target_link_libraries(${PROJECT_NAME} dlib)
```
配置windows c++编译环境（64位）

![1639462246755](D:\Workspaces\Log\book\jni\assets\1639462246755.png)

编译生成dlib.lib (windows系统静态库)

拷贝hello.h到C:/Users/mouse/Desktop/main/java/include

拷贝dlib.lib到C:/Users/mouse/Desktop/main/java/lib

#### Java代码

```java
package top.laoshuzi.jdlib;

import org.bytedeco.javacpp.Loader;
import org.bytedeco.javacpp.Pointer;
import org.bytedeco.javacpp.annotation.Namespace;
import org.bytedeco.javacpp.annotation.Platform;

@Platform(
        includepath = "C:/Users/mouse/Desktop/main/java/include",
        include = "hello.h",
        linkpath = "C:/Users/mouse/Desktop/main/java/lib",
        link = "dlib"
)
@Namespace("jdlib")
public class Jdlib {

    public static class Hello extends Pointer {

        static {
            Loader.load();
        }

        public Hello() {
            allocate();
        }

        private native void allocate();

        public native void sayHello();

    }

    public static void main(String[] args) {
        Hello hello = new Hello();
        hello.sayHello();
    }

}
```

编译java代码

```shell
javac -cp javacpp-1.5.6.jar .\top\laoshuzi\jdlib\Jdlib.java
```

编译生成jni相关代码与库

```shell
java -jar .\javacpp-1.5.6.jar top.laoshuzi.jdlib.Jdlib
```

测试java代码

```shell
java -cp .;javacpp-1.5.6.jar top.laoshuzi.jdlib.Jdlib
```

