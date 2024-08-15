@[TOC]
## 摘要
在一些项目中需要打包文件夹或者文件，那么如何使用c++代码实现这个需求呢？开源压缩库zlib能实现这个需求， 但是zlib库属于比较底层的库，直接对数据进行压缩，不方面使用。quazip结合了zlib 和qt开源压缩库，提供了方便使用api，能直接压缩文件和文件夹
***

## 一、下载编译zlib库
由于quazip依赖zlib库，需先下载编译zlib，在github上拉取代码

```sh
git clone https://github.com/madler/zlib.git
```

使用cmak-gui.exe打开源码，没有这个工具先下载

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_00.png)

点击配置，选择默认配置，再点击完成

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_01.png)

点击生成Visual Studio解决方案，点击打开工程，自动启动Visual Studio

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_02.png)

生成静态库和动态库

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_03.png)

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_04.png)

生成example,运行测试

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_05.png)
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_06.png)

测试无错

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_07.png)

安装库到默认路径

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_08.png)

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_09.png)

以上步骤没有问题切换到release，直接安装库，再剪切备用
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_10.png)
## 二、下载编译quazip
github下载源码

```sh
git clone https://github.com/stachenov/quazip.git
```

修改代码根路径下CMakeListx.txt文件，注释原来的代码，添加下文的代码

```cmake
#      find_package(ZLIB REQUIRED)  
#      set(QUAZIP_LIB_LIBRARIES ${QUAZIP_LIB_LIBRARIES} ZLIB::ZLIB)  
    include_directories(D:/code/QT/test/zlib/install/include)  #添加zlib库头文件所有路径
if (${CMAKE_BUILD_TYPE} MATCHES "Debug")  
    set(ZLIB_LIBRARY D:/code/QT/test/zlib/install/lib/zlibstaticd.lib)  #Debug版本zlib静态库路径
else ()  
    set(ZLIB_LIBRARY D:/code/QT/test/zlib/install/lib/zlibstatic.lib)  #Release版本zlib静态库路径
endif ()
```

在所搜QT库之前添加Qt库路径
```cmake
set(CMAKE_PREFIX_PATH C:/Programs/Qt6.5.3/6.5.3/msvc2022_64_static)
```
\
修改quazip文件夹下CMakeListx.txt，主要是修改编译静态和动态库\

```
if (BUILD_SHARED_LIBS)
    add_library(${QUAZIP_LIB_TARGET_NAME} SHARED ${QUAZIP_SOURCES})
else ()
    add_library(${QUAZIP_LIB_TARGET_NAME} STATIC ${QUAZIP_SOURCES})
endif ()
```

使用cmake-gui生成配置配置流程与上文相同，修改取消编译动态库库，不添加BZP2库
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_11.png)

打开Visual Studio直接生成解决方案中的INSTALL项目，Debug和Release版本都生成，拷贝备用

## 三、创建quazip_demo项目使用库

zlib库只需要下文件和文件夹其余删除，注意框住的是静态库，有"**d**"后缀的是debug版本库
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_12.png)

quazip库只需要以下文件和文件夹

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_13.png)

准备下源文件，文件路径是情况修改
```cpp
#include <quazip/JlCompress.h>
  
int main(int argc, char ** argv)
{
    JlCompress::compressFile("D:/code/QT/test/quazip_demo/main.cpp.zip", "D:/code/QT/test/quazip_demo/main.cpp");
}
```

准备以下CMakeListx.txt文件

```cmake
cmake_minimum_required(VERSION 3.23)

project(quazip_demo CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)


set(CMAKE_PREFIX_PATH C:/Programs/Qt6.5.3/6.5.3/msvc2022_64_static)
find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)

find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS
        Core
        PrintSupport
        Core5Compat
)

include_directories(QuaZip/include/QuaZip-Qt6-1.4 zlib/include)
add_compile_definitions(QUAZIP_STATIC) //此宏在链接静态库时必须添加，否则链接失败


add_executable(quazip_demo main.cpp)

target_link_libraries(quazip_demo PUBLIC
        Qt${QT_VERSION_MAJOR}::Core
        Qt${QT_VERSION_MAJOR}::PrintSupport
        Qt${QT_VERSION_MAJOR}::Core5Compat
)


if(CMAKE_BUILD_TYPE MATCHES "Release")
    target_link_libraries(${PROJECT_NAME} PUBLIC
            ${CMAKE_SOURCE_DIR}/zlib/lib/zlibstatic.lib
            ${CMAKE_SOURCE_DIR}/QuaZip/lib/quazip1-qt6.lib
    )
else ()
    target_link_libraries(${PROJECT_NAME} PUBLIC
            ${CMAKE_SOURCE_DIR}/zlib/lib/zlibstaticd.lib
            ${CMAKE_SOURCE_DIR}/QuaZip/lib/quazip1-qt6d.lib
    )
endif()
```

文件结构

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_14.png)

用clion打开项目选择MSVC编译器，编译运行能得到下图结构就说明库编译正常

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/qt/03/qt_03_15.png)