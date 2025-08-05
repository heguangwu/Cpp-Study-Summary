# QT 常见问题

## 资源文件无法加载

从`qrc`复制的路径，提示无法打开`Cannot open: qrc:/img/icons/close.png`之类的错误，这是因为没有开启 CMake 自动处理 Qt 资源文件：

```cmake
set(CMAKE_AUTORCC ON)
```
