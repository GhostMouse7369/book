## cmake编译bcm2835库

### 准备

bcm2835源代码:http://www.airspayce.com/mikem/bcm2835/



### 编译

##### 配置编译器



##### 导入bcm2835源代码

![1571135793416](https://github.com/GhostMouse7369/book/tree/master/rpi/assets/1571135793416.png)

##### cmake编译bcm2835库

```cmake
cmake_minimum_required(VERSION 3.13)
project(bcm2835 C)

set(CMAKE_C_STANDARD 99)

SET(LIBRARY_OUTPUT_PATH ./libs)

aux_source_directory(./src DIR_SRCS)

add_library(bcm2835 SHARED ${DIR_SRCS})
```

##### 编译成功



git:<https://github.com/GhostMouse7369/rpi-lib-bcm2835> 
