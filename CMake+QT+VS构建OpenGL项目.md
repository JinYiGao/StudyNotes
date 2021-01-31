---
typora-copy-images-to: image
---

# CMake+QT+VS构建OpenGL项目

## 1. VScode编写CMakeLists.txt

```cmake
#指定cmake最小版本
cmake_minimum_required(VERSION 3.10)

#指定项目名称
project(Hello_Triangle)

#使用的标准版本
set(CMAKE_CXX_STANDARD 11) 

#设置工程包含当前目录，非必须
SET(CMAKE_INCLUDE_CURRENT_DIR ON) 

#设置编译时 将所有dll和exe放入bin目录下，将所有lib放入lib目录下。 可选
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

#添加头文件搜索目录
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

#添加子目录
add_subdirectory(src)
```

## 1.1 显式处理某些文件

```cmake
# Qt会对某些文件进行处理，比如带有 Q_OBJECT 宏的头文件，需要被 moc 程序处理
# .ui 界面设计文件，需要被 uic 程序处理
# .qrc 资源文件，需要被 rcc 程序处理
# 在cmake中，需要显式声明对这些文件进行处理

# 设置自动生成moc文件,AUTOMOC打开可以省去QT5_WRAP_CPP命令
SET(CMAKE_AUTOMOC ON)

# 设置自动生成ui.h文件,AUTOUIC打开可以省去QT5_WRAP_UI命令
SET(CMAKE_AUTOUIC ON)

#通过Ui文件生成对应的头文件，一定要添加
#QT5_WRAP_UI(WRAP_FILES ${UI_FILES})
```

## 1.2获取需要的组件

```cmake
#find_package 获取QT5中的Core、Gui等组件
find_package(Qt5 REQUIRED COMPONENTS Core Gui Widgets)
```

## 1.3打包编译

```cmake
#打包文件 以一个变量名表示
file(GLOB SOURCES *.cpp *.h)
file(GLOB UI_FILES *.ui)

#将ui文件和生成文件整理在一个文件夹中，非必须
SOURCE_GROUP("Ui" FILES ${UI_FILES} ${WRAP_FILES} )

#将源文件资源编译为可执行文件
add_executable(main ${SOURCES} ${UI_FILES} )

#添加程序所需要的的链接库
target_link_libraries(main Qt5::Core Qt5::Widgets Qt5::Gui)
```

## 1.4CMake编译

```shell
#使用bat脚本来指定编译方式
mkdir build
cd build

cmake -G "Visual Studio 15 2017" -A "x64" -T "host=x64" ^
    -DCMAKE_BUILD_TYPE=Release ^
    ../
```

## 1.5dll复制

```shell
# vs编译时可能会提示缺少dll，去复制一下就行。
```

## 1.6 OpenGL窗口渲染

```shell
#自定义一个widget，将它继承自QOpenGLWidget这个类 就可以进行渲染了，具体可以查看Qt官方帮助文档
```

