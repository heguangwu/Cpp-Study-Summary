# CMake简明教程

<details>
<summary><b>目录</b></summary>

[通用方法](#通用方法)</br>
[指定最低版本](#指定最低版本)</br>
[项目信息](#项目信息)</br>
[生成可执行文件](#生成可执行文件)</br>
[制作静态库](#制作静态库)</br>
[制作动态库](#制作动态库)</br>
[定义变量](#定义变量)</br>
[添加依赖](#添加依赖)</br>
[搜索源文件](#搜索源文件)</br>
[搜索头文件](#搜索头文件)</br>
[链接静态库](#链接静态库)</br>
[链接动态库](#链接动态库)</br>
[`CMake`嵌套](#cmake嵌套)</br>
[打印日志](#打印日志)</br>
[列表操作](#列表操作)</br>
[定义宏](#定义宏)</br>
</details>

`CMake`的作用就是为了生成`Makefile`，因为编辑`CMakeLists.txt`要比直接编辑`Makefile`要简单。本文侧重从生成`Makefile`的角度来讲述的。

## 通用方法

### 指定最低版本

```cmake
# 可选
cmake_minimum_required(VERSION 3.12)
```

### 项目信息

```cmake
# 指定项目名称、软件版本（可选）、项目描述（可选）、主页URL（可选）、编程语言（可选）
project(PROJECT_NAME)
```

### 生成可执行文件

```cmake
# add_executable(可执行程序名 源文件名称)
add_executable(Hello ${SRC_LIST})
```

这里的`可执行程序名`和上面的`项目名称`没有任何关系。
`源文件名称`可以多个，之间用空格分隔，也可以通过变量指定([变量定义如下](#定义变量))。

### 制作静态库

```cmake
# add_library(库名称 STATIC 源文件名称)
add_library(calc STATIC ${SRC_LIST})
```

- 在`Linux`下，生成的库为`libcalc.a`
- 在`Windows`下，生成的库为`libcalc.lib`

### 制作动态库

```cmake
# add_library(库名称 SHARED 源文件名称)
add_library(calc SHARED ${SRC_LIST})
```

- 在`Linux`下，生成的库为`libcalc.so`
- 在`Windows`下，生成的库为`libcalc.dll`

## 定义变量

```cmake
# set(变量名 变量名/变量值)
set(SRC_LIST main.cpp hello.cpp)
set(SRC_LIST ${SRC_LIST} world.cpp)
```

- 内置变量：C++版本 CMAKE_CXX_STANDARD

```cmake
# 指定使用 C++ 11标准
set(CMAKE_CXX_STANDARD 11)
# 指定使用 C++ 14标准
set(CMAKE_CXX_STANDARD 14)
# 指定使用 C++ 17标准
set(CMAKE_CXX_STANDARD 17)
```

- 内置变量：执行文件输出路径  EXECUTABLE_OUTPUT_PATH

```cmake
set(EXECUTABLE_OUTPUT_PATH /opt/bin)
```

**输出目录会自动创建**。

- 内置变量：库文件输出路径  LIBRARY_OUTPUT_PATH

```cmake
set(LIBRARY_OUTPUT_PATH /opt/lib)
```

## 添加依赖

### 搜索源文件

```cmake
# 搜索 src 目录下的源文件赋值给 SRC_LIST
aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC_LIST)

# 递归搜索 src 目录下的cpp后缀文件赋值给 SRC_LIST
file(GLOB_RECURSE SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

# 搜索 src 目录下的cpp后缀文件赋值给 SRC_LIST
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
```

- `PROJECT_SOURCE_DIR`是执行`cmake`命令后的路径
- `CMAKE_CURRENT_SOURCE_DIR` 是当前执行的`CMakeLists.txt`的路径

### 搜索头文件

```cmake
# 指定头文件路径，多个以空格隔开
include_directories(${PROJECT_SOURCE_DIR}/include)
# 另外一种方法
# target_include_directories(目标 权限如PUBLIC 头文件路径)
```

### 链接静态库

```cmake
link_directories(库路径1 库路径2)
link_libraries(库名称1 库名称2)
```

### 链接动态库

```cmake
link_directories(库路径1 库路径2)
# target_link_libraries(target(可以是可执行文件、动态库等) 访问权限(默认PUBLIC，可选) 库名称)
target_link_libraries(my_app pthread)
```

**`target_link_libraries`要放最后，也可以连接静态库。**

### `find_package`搜索库

使用`link_directories`有一个很重要的问题，当每一个机器安装的库路径不一样的时候，需要根据本机情况进行路径修改，那么有没有一种方法可以解决依赖库不同安装路径的问题呢，答案就是使用`find_package`。

```cmake
# 添加 opencv 的依赖
find_package(OpenCV REQUIRED)

# 头文件
target_include_directories(my_app PRIVATE ${OpenCV_INCLUDE_DIRS})

# 库文件
target_link_libraries(my_app ${OpenCV_LIBS})
```

`find_package`的原理：

- 第三方库（如`OpenCV`）提供被系统环境变量找到，目前存在两种模式，`Module`模式和`Config`模式。
- `Module`模式查找路径：`CMAKE_MODULE_PATH`和`CMake`安装路径（即`CMAKE_ROOT`变量）下的`Modules`目录，查找到名为`Find<PackageName>.cmake`的配置文件
- `Config`模式下是要查找名为`<PackageName>Config.cmake`或`<lower-case-package-name>-config.cmake`的模块文件
，`Config`模式查找路径：
  - `<PackageName>_DIR`的CMake变量或环境变量路径
  - `CMAKE_PREFIX_PATH`、`CMAKE_FRAMEWORK_PATH`、`CMAKE_APPBUNDLE_PATH`的`CMake`变量或环境变量路径
  - `PATH`环境变量路径
- `Config`模式找到后会设置`<PackageName>_INCLUDE_DIRS`和`<PackageName>_LIBRARIES`两个变量

## `CMake`嵌套

```cmake
# 指定子节点目录: calc
add_subdirectory(calc)
# 指定子节点目录: sort
add_subdirectory(sort)
```

- 父`CMakeLists.txt`定义的变量在子`CMakeLists.txt`可以直接使用。
- 在子`CMakeLists.txt`定义的变量父`CMakeLists.txt`无法访问。

## 高级操作

### 打印日志

```cmake
# message([STATUS|WARNING|AUTHOR_WARNING|SEND_ERROR|FATAL_ERROR])
message("this is import library")
message(STATUS "this is import library")
```

### 列表操作

```cmake
# 拼接： list(APPEND <list> [element])
list(APPEND SRC_LIST ${SRC_LIST} ${EXT_LIST})

# 移除main.cpp文件
# 使用方式：list(REMOVE_ITEM <list> <value>)
list(REMOVE_ITEM ${SRC_LIST} ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp)

# 获取最后一个元素
list(GET ${SRC_LIST} -1)

# 使用`:`连接列表元素组成一个字符串
list(JOIN ${SRC_LIST} : new_string)
```

**`list`内部是由`;`分割的一组字符串，但打印或使用时看不到`;`。**

### 定义宏

```cmake
# add_definitions(-D宏名称)
# 定义 DEBUG
add_definitions(-DDEBUG)
```
