# 是什么?
Ninja 是一个专注于速度的小型构建系统，它是为了替代传统的构建工具（例如 Make）而设计的。Ninja 的主要目的是高效地重新构建大型代码库，尤其是在处理大量源文件和依赖关系时。它通常与其他工具（如 CMake）结合使用，以生成构建配置。
### Ninja 的特点：
1. **高效性**：
   - Ninja 一般比传统的构建工具更快速，因为它设计得非常简单，专注于只执行必要的构建步骤。
   - 它通过较少的文件 I/O 操作和并行执行来加速构建过程。
2. **并行构建**：
   - Ninja 可以轻易地利用多核心 CPU 的并行处理能力，能够同时执行多个任务，从而加快构建速度。
3. **简洁的语法**：
   - Ninja 的构建文件（通常为 `build.ninja`）使用简单的语法来定义目标、规则和依赖关系。在很多情况下，Ninja 的构建文件比 Makefile 更简洁。
4. **与其他工具兼容**：
   - ==Ninja 通常与 CMake、GN 等工具结合使用==，这些工具负责生成 Ninja 构建文件。这样，开发者可以在使用高层次编译系统的同时，享受 Ninja 的构建速度。
### 示例：
一个简单的 `build.ninja` 文件示例：
```ninja
# 目标：可执行文件
rule cc
    command = gcc -o $out $in
build my_program: cc main.c utils.c
```
在这个示例中：
- 定义了一个名为 `cc` 的规则，用于使用 `gcc` 编译源文件。
- 建立了一个名为 `my_program` 的目标，该目标依赖于 `main.c` 和 `utils.c`。
### 结论：
Ninja 是一个非常高效且便捷的构建系统，尤其适合大型项目和需要快速构建的开发环境。它通过简单的构建文件和并行构建的能力，帮助开发者更高效地管理项目的构建过程。
# 语法
Ninja 的语法非常简洁，使得编写构建文件易于理解。下面是 Ninja 语法的一些基本元素和规则：
### 1. 规则 (Rules)
规则定义了如何构建目标。每个规则有一个名称和一个执行的命令。
```ninja
rule rule_name
    command = command_to_execute
```
### 2. 构建语句 (Build Statements)
构建语句使用已定义的规则来指定目标及其依赖项。
```ninja
build output_target: rule_name input_files
```
### 3. 变量 (Variables)
Ninja 支持变量，可以在构建文件中定义和使用。
```ninja
cxx = g++
cflags = -Wall -O2
build my_program: cxx main.cpp | utils.cpp
    command = $cxx $cflags -o $out $in
```
### 4. 生成的文件 (Generated Outputs)
使用 `$out` 和 `$in` 前缀引用输入和输出文件。
- `$out` 表示输出文件的名称。
- `$in` 表示输入文件的名称，多个输入文件可以用空格分隔。
### 5. 伪目标 (Phony Targets)
Ninja 支持定义伪目标，这些目标不对应实际文件。
```ninja
build clean: phony
    command = rm -f my_program
```
### 6. 完整示例
下面是一个简单的 Ninja 构建文件示例：
```ninja
# 定义编译器和选项 
cc = gcc 
cflags = -Wall -O2
# 定义规则
rule cc_rule
    command = $cc $cflags -o $out $in
# 构建可执行文件
build my_program: cc main.c utils.c
# 定义清理目标
build clean: phony
    command = rm -f my_program
```
- **规则**：是关于如何构建目标的指令和命令。它不是可执行文件，而是表示生成可执行文件或其他目标的过程。
- **构建目标**：是实际的输出文件（如可执行文件）以及生成这些文件所需的依赖源。它是构建过程的结果。
因此，您可以将规则视作“构建的方式”，而将构建目标和它的依赖视作“需要构建的东西”。这两者结合起来，用于理解和执行 Ninja 构建文件。
