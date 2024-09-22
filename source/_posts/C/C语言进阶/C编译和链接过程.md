---
title: "C编译和链接过程"
date: 2024-09-20 13:30:00
categories: "C语言"
tags: 
- "C语言进阶"
---

C 语言的编译器是将 **C 源代码** 转换为 **机器可执行代码** 的工具。编译器通过几个阶段的处理，把人类可读的高级编程语言代码（如 C 语言）转换为计算机可以直接理解和运行的机器代码。常见的 C 语言编译器有 **GCC**、**Clang**、**MSVC** 等。

一个 C 语言编译器包括：预处理器、编译器、汇编器、链接器等，几个子模块。

## 编译过程

**C 编译器的主要功能和工作流程：**

1. **预处理（Preprocessing）**：

- 处理所有的注释，以空格代替

- 将所有的 #define 删除，并且展开所有的宏定义

- 处理条件编译指令 #if、#ifdef、#elif、#else、#endif

- 处理 #include，展开被包含的文件

- 保留编译器需要使用的 #pragma 指令

  预处理指令示例：gcc -E file.c -o file.i

2. **编译（Compilation）**：

- 对预处理文件进行**词法分析**、**语法分析**和**语义分析**

  - 词法分析：分析关键字、标识符、立即数等是否合法
  - 语法分析：分析表达式是否遵循语法规则
  - 语义分析：在语法分析的基础上进一步分析表达式是否合法

- 分析结束后进行**代码优化生成相应的汇编代码文件**

  编译指令示例：gcc -S file.i -o file.s

3. **汇编（Assembly）**：

- 汇编器将汇编代码代码转为 **机器指令**（即目标代码，通常是 `.obj` 或 `.o` 文件）。

- 每条汇编语句几乎都对应一条机器指令

  汇编指令示例：gcc -c file.s -o file.o

4. **链接（Linking）**：

- 链接器负责将不同的目标文件（包括库文件、外部依赖）合并到一起，生成最终的可执行文件（如 `.exe` 或 `.out` 文件）。
- 链接器还会处理符号解析，确保函数和变量在整个程序中正确引用。

![Clianjieqi](/public/image/C/C语言进阶/Clianjieqi.png)

## 链接过程

连接器的主要作用是把各个模块之间相互引|用的部分处理好,使得各个模块之间能够正确的衔接。

![lianjieguoc](/public/image/C/C语言进阶/lianjieguoc.png)

**链接器的两种工作模式**

### **静态链接**

将所有的库代码和目标文件合并到最终的可执行文件中。

![Cjintailianjie](/public/image/C/C语言进阶/Cjintailianjie.png)

a.out 可以单独运行，不依赖外部库；运行时无需动态加载库，启动速度快。

**Linux 下静态库的创建和使用**

- 编译静态库源码：gcc -c lib.c -o lib.o
- 生成静态库文件：ar -q lib.a lib.o
  - （ar 是个打包命令会将后面列出来的的所有文件打包进 lib.a 中）
- 使用静态库编译：gcc main.c lib.a -o main.out
  - 此时生成的 main.out 程序可以单独运行，即使你将 main.c、lib.a、lib.c、lib.o这些源文件全部删掉也没有影响。

### **动态链接**

- 只在生成可执行文件时记录库的位置，在程序运行时由系统加载动态库（如 `.dll` 文件或 `.so` 文件）。
- 可执行**程序在运行时**才动态加载库进行链接
- 库的内容不会进行到可执行文件中

![Cdongtailianjie](/public/image/C/C语言进阶/Cdongtailianjie.png)

连接器在最后链接时候不会将 lib1.so、lib2.so 直接打包进 a.out，他只知道可执行程序在执行时需要查找的库在什么位置。

stub1、stub2 就是 liib1.so、lib2.so 分别暴露出来告诉编译器他可以使用的内容只有 stub1、stub2；

> 库的更新可能导致不兼容问题（即 "DLL Hell"）。

**Linux 下动态链接的创建和使用**

- 编译动态库源码：gcc -shared dlib.c -o dlib.so
- 使用动态库编译：gcc main.c -ldl -o main.out
- 关键系统调用
  - dlopen：打开动态库文件
  - dlsym：查找动态库中的函数并返回调用地址
  - dlclose：关闭动态库文件
