# CMake 速查
## 编写 CMakeLists.txt
假如有一个项目如下：
```shell
MyProject/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   └── mylib.cpp
└── include/
    └── mylib.h
```
* main.cpp：主程序源文件。
    * 包含程序的入口函数。
    * 如
    ```cpp
    #include <iostream>
    #include "math_utils.h"
    int main() {
        int x = 10;
        int y = 5;

        // 调用 add 函数
        std::cout << "Sum: " << add(x, y) << std::endl;

        // 调用 subtract 函数
        std::cout << "Difference: " << subtract(x, y) << std::endl;

        return 0;
    }
    ```
* mylib.cpp：库源文件。
    * 实现头文件中声明的函数或类。它包含具体的代码逻辑。
    * 如
    ```cpp
    #include "math_utils.h"
    // 实现 add 函数
    int add(int a, int b) {
        return a + b;
    }

    // 实现 subtract 函数
    int subtract(int a, int b) {
        return a - b;
    }
    ```
* mylib.h：库头文件。
    * 头文件通常用于声明类、函数、变量等，以便在其他文件中使用。头文件的内容通常包括：
        * 类的声明。
        * 函数的声明。
        * 常量的定义
        * 类型定义
        * 宏定义
    * 如
    ```cpp
    #ifndef MATH_UTILS_H
    #define MATH_UTILS_H
    // 函数声明
    int add(int a, int b);
    int subtract(int a, int b);
    #endif // MATH_UTILS_H
    ```

* CMakeLists.txt：CMake 配置文件。

```shell
cmake_minimum_required(VERSION 3.10)   
# 指定最低 CMake 版本为 3.10。

project(MyProject VERSION 1.0)          
# 定义项目名称为 MyProject，版本为 1.0。

# 设置 C++ 标准为 C++11
set(CMAKE_CXX_STANDARD 11)

# 设置 C++ 标准要求
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 指定头文件目录。
include_directories(${PROJECT_SOURCE_DIR}/include)

# 添加源文件
add_library(MyLib src/mylib.cpp)        
# 创建一个名为 MyLib 的库，源文件是 mylib.cpp
add_executable(MyExecutable src/main.cpp)  
# 创建一个名为 MyExecutable 的可执行文件，源文件是 main.cpp。

# 将 MyLib 库链接到 MyExecutable 可执行文件
target_link_libraries(MyExecutable MyLib)
```

## 编译项目
```shell
    cmake .
    # cmake 后的位置是 CMakeLists.txt 所在的目录。而此条命令所在的位置则是 Nakefile 所在的目录。
    make
```
推荐先建立一个 build 目录，然后进入 build 目录，执行上述命令。