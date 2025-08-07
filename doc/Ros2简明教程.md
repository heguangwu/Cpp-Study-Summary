# `ROS2` 简明教程

## 基础知识

`ros2`的工作空间就是一个目录，在目录下创建`src`目录，就可以在`src`目录下构建功能包：

### python

```shell
# 创建 python 功能包
# ros2 pkg create 功能包名 --build-type ament_python --dependencies 依赖包名 --license 开源协议
ros2 pkg create my_publisher_pkg --build-type ament_python --dependencies rclpy
```

接下来在`setup.py`文件中的`entry_points`的`console_scripts`增加一行，格式为：

`应用名=功能包名.python文件名:main`

如你的`status_publisher`功能包下面有一个`sys_status_pub.py`文件，你给应用起名为`sys_status_pub`，那么增加信息如下：

```python
    entry_points={
        'console_scripts': [
            'sys_status_pub=status_publisher.sys_status_pub:main',
        ],
    },
```

需要增加依赖包，修改`package.xml`文件，如下增加`status_interfaces`依赖：

```xml
<depend>rclpy</depend>
<depend>status_interfaces</depend>
```

### C++

```shell
# 创建 c++ 功能包
# ros2 pkg create 功能包名 --build-type ament_cmake --dependencies 依赖包名 --license 开源协议
ros2 pkg create demo_cpp_topic --build-type ament_cmake --dependencies rclcpp geometry_msgs turtlesim

# 创建 c++ 自定义通信接口功能包
ros2 pkg create status_interfaces --build-type ament_cmake  --dependencies rosidl_default_generators builtin_interfaces
```

- `rosidl_default_generators`: 用于将自定义消息转换为`python`和`c++`源代码模块
- `builtin_interfaces`: `ros`的接口功能包，里面定义了`Time`等数据接口

同样都需要修改`package.xml`文件：

```xml
<depend>rclcpp</depend>
<depend>geometry_msgs</depend>
<depend>turtlesim</depend>
```

#### 接口定义

要增加接口依赖，需修改`CMakeLists.txt`文件，以上两个依赖为例：

```cmake
find_package(rosidl_default_generators REQUIRED)
find_package(builtin_interfaces REQUIRED)

# 生成接口代码必须指定接口定义文件 SystemStatus.msg 以及该文件依赖 builtin_interfaces
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/SystemStatus.msg"
  DEPENDENCIES
  builtin_interfaces
)
```

#### 功能包定义

如下增加了两个应用`turtle_circle`和`turtle_control`：

```cmake
# 1.查找 rclcpp 头文件和库文件
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(turtlesim REQUIRED)

include_directories(include)

# 2.添加可执行文件
add_executable(turtle_circle src/turtle_circle.cpp)
add_executable(turtle_control src/turtle_control.cpp)


# 3.可执行文件添加依赖
ament_target_dependencies(turtle_circle rclcpp geometry_msgs)
ament_target_dependencies(turtle_control rclcpp geometry_msgs turtlesim)

# 4.将 cpp_node 复制到 install 目录
install(TARGETS turtle_circle DESTINATION lib/${PROJECT_NAME})
install(TARGETS turtle_control DESTINATION lib/${PROJECT_NAME})
```

**如需添加非ROS的第三方库，使用`target_link_libraries`而不是`ament_target_dependencies`**

## ROS中使用QT显示界面

```cpp
#include <QApplication>
#include <QLabel>
#include <QString>
#include "rclcpp/rclcpp.hpp"

class SysStatusDisplay : public rclcpp::Node {
    //省略相关实现
}

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    QApplication app(argc, argv);
    auto node = std::make_shared<SysStatusDisplay>();
    // QT和ROS都需要阻塞执行，所以这里启动一个线程来启动ros
    std::thread spin_thread([&]() -> void {rclcpp::spin(node);});
    spin_thread.detach();
    app.exec();
    rclcpp::shutdown();
    return 0;
}
```

```cmake
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(status_interfaces REQUIRED)
find_package(Qt5 REQUIRED COMPONENTS Widgets)

add_executable(sys_status_display src/sys_status_display.cpp)
# 添加QT库
target_link_libraries(turtle_circle Qt5::Widgets)
# 添加ROS相关依赖
ament_target_dependencies(sys_status_display rclcpp status_interfaces)

install(TARGETS sys_status_display DESTINATION lib/${PROJECT_NAME})

```
