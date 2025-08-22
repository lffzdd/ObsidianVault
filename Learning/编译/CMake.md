> [CMake 教程 | 菜鸟教程](https://www.runoob.com/cmake/cmake-tutorial.html)
>CMake 是个一个开源的跨平台自动化建构系统，用来管理软件建置的程序，并不依赖于某特定编译器，并可支持多层目录、多个应用程序与多个函数库。
>CMake 通过定义一个高层次的构建描述文件（通常名为 `CMakeLists.txt`），自动生成不同平台的构建文件（如 Makefile、Ninja 构建文件、Visual Studio 工程文件等），简化了项目的编译和构建过程。
>CMake 本身不是构建工具，而是生成构建系统的工具，它生成的构建系统可以使用不同的编译器和工具链。
### CMake 的关键特点：
1. **跨平台支持**：
    - CMake 可以在 Windows、macOS 和 Linux 等多个操作系统上工作，支持大多数编译器。
2. **简化构建过程**：
    - CMake 通过自动生成构建系统所需的配置文件，减少了手动管理 Makefile 或其他构建文件的复杂性。
3. **灵活的输入**：
    - CMake 可以处理复杂的项目结构，支持包含多个子目录、外部库、不同的构建配置（如 Debug 和 Release）、特性和选项。
## 基本工作流程
1. **编写 CMakeLists.txt 文件：** 定义项目的构建规则和依赖关系。
2. **生成构建文件：** 使用 CMake 生成适合当前平台的构建系统文件（例如 Makefile、Visual Studio 工程文件）。
3. **执行构建：** 使用生成的构建系统文件（如 `make`、`ninja`、`msbuild`）来编译项目。
4. **模块化和复用**：
    - CMake 允许开发者创建可重用的模块和库，促进代码的共享和复用。
5. **可扩展性**：
    - CMake 提供了一套丰富的命令和变量，可以用于自定义构建流程和配置。
![[Pasted image 20241108175119.png]]
# 语法
## CMake 的基本用法：
1. **创建 CMakeLists.txt 文件**：
   - 该文件描述项目的构建要求和设置。
   ```cmake
   cmake_minimum_required(VERSION 3.10)
   project(MyProject)
   add_executable(my_program main.cpp utils.cpp)
   ```
2. **生成构建文件**：
   - 通过终端进入项目目录，运行以下命令生成构建文件：
   ```bash
   mkdir build
   cd build
   cmake ..
   ```
3. **执行构建**：
   - 使用生成的构建系统来编译项目。例如，如果生成的是 Makefile，则可以运行：
   ```bash
   make
   ```
# 实例
当然！下面是一个详细的 CMake 实际应用流程的示例，以一个简单的 C++ 项目为基础。这将涵盖整个过程，从项目结构的创建到构建及运行程序。
### 项目示例
我们将创建一个名为 `MyCalculator` 的简单计算器项目，其包含以下 C++ 源文件：
- `main.cpp`: 主程序文件，负责输入和输出。
- `calculator.cpp`: 定义计算器的逻辑。
- `calculator.h`: 声明计算器的函数。
### 第一步：创建项目结构
首先，创建一个文件夹来存放项目，然后在其中创建相应的文件。
```bash
mkdir MyCalculator
cd MyCalculator
mkdir src include
touch src/main.cpp src/calculator.cpp include/calculator.h
```
### 项目文件内容
- **include/calculator.h**:
```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H
class Calculator {
public:
    int add(int a, int b);
    int subtract(int a, int b);
};
#endif // CALCULATOR_H
```
- **src/calculator.cpp**:
```cpp
#include "calculator.h"
int Calculator::add(int a, int b) {
    return a + b;
}
int Calculator::subtract(int a, int b) {
    return a - b;
}
```
- **src/main.cpp**:
```cpp
#include <iostream>
#include "calculator.h"
int main() {
    Calculator calc;
    int a = 5, b = 3;
    std::cout << "Add: " << calc.add(a, b) << std::endl;
    std::cout << "Subtract: " << calc.subtract(a, b) << std::endl;
    return 0;
}
```
### 第二步：创建 CMakeLists.txt 文件
在项目根目录下创建一个名为 `CMakeLists.txt` 的文件，内容如下：
```cmake
cmake_minimum_required(VERSION 3.10) # CMake 最低版本要求
project(MyCalculator)                 # 项目名称

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 指定包含目录
include_directories(include)

# 添加可执行文件
add_executable(my_calculator src/main.cpp src/calculator.cpp)
```
### 第三步：生成构建文件
1. **创建构建目录**：
   在项目根目录下创建一个构建目录（通常命名为 `build`）：
   ```bash
   mkdir build
   cd build
   ```
2. **运行 CMake**：
   在构建目录中运行 CMake，以生成构建文件：
   ```bash
   cmake ..
   ```
   这条命令告知 CMake 去上一级目录寻找 `CMakeLists.txt` 文件并生成构建文件。
### 第四步：编译项目
在构建目录中运行构建命令：
```bash
make
```
这将根据生成的 Makefile 来构建项目。在此过程中，您会看到编译器输出和构建过程的详细信息。
### 第五步：运行生成的可执行文件
编译成功后，您将找到生成的可执行文件（此处为 `my_calculator`）。通过以下命令运行它：
```bash
./my_calculator
```
您应该会看到如下输出：
```
Add: 8
Subtract: 2
```
### 总结
这是一个简单的 CMake 实际应用流程的示例。该流程显示了如何：
1. 组织项目结构。
2. 使用 CMake 生成构建系统所需文件。
3. 编译代码并生成可执行文件。
4. 运行程序并查看结果。
通过这个实例，您可以看到 CMake 如何简化跨平台的项目管理和构建过程，使得开发者能够集中精力于代码本身。
