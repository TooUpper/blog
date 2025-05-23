---
title: "make 和 makefile"
date: 2024-08-01 20:12:00
categories: "Linux"
tags: 
- "make"
---

## make 与 makefile介绍

make：是一个工具程序，通过读取"makefile"文件以实现自动化构建软件。

- **解析源程序之间的依赖关系**
- 根据依赖关系**自动维护**编译工作
- 执行宿主操作系统中的各种命令

**众多编译辅助工具中的王者**

makefile：是一个配置文件，一个描述文件。

- **定义一系列规则**来指定源文件编译的先后顺序
- **拥有特定的语法规则**，支持函数定义和函数调用
- 能够**直接集成**操作系统中的各种命令

**二者关系**：makefile 中的描述用于指导 make 程序如何完成工作；make 根据 makefile 中的规则执行命令，最后完成编译输出。

![erzheguanxi](/public/image/Linux/make/erzheguanxi.png)

简单示例

make 程序的使用示例：

```makefile
# 以 hello 关键字作为目标查找 mf.txt 文件，并执行 hello 处的命令。
make -f mf.txt hello

# 简写

# 在当前目录下以 hello 关键字为目标查找 makefile 或 Makefile 文件，并执行 hello 处的命令。
make hello

# 在当前目录下查找 makefile 或 Makefile 文件中最顶层目标，并执行最顶层目标的命令。
make
```

小结

- make 只是一个**特殊功能的**应用程序
- make 用于根据指定的目标**执行相应的命令**
- makefile 用于**定义目标**和实现目标所需的命令
- makefile 有特定的语法规则，支持函数定义和调用

## makefile 的结构

**makefile 的意义**

- makefile 用于定义源文件之间的依赖关系

- makefile 说明如何编译各个源文件并生成可执行文件

```makefile
# makefile语法
# 第一种写法
target(目标文件): 文件1 文件2(依赖文件列表); command(命令) 

# 第二种写法
target(目标文件): 文件1 文件2(依赖文件列表) 
`\t`command
```

注意事项

target 可以包含多个目标
- 使用空格对多个目标进行分割

依赖 可以包含多个依赖
- 使用空格对多个依赖进行分割

[Tab]键：`\t`

- 每一个命令必须以[Tab]字符开始
- [Tab]字符告诉 make 此行是一个命令行

续航符: \
- 可以将内容分开写到下一行，提高可读性

**makefile的依赖示例**

```makefile
all: test
	echo "make all"
test: 
	echo "make test"

# 输出： 可使用 @ 进行无回写设置
# echo "make test"
# make test
# echo "make all"
# make all
```

依赖规则

- 当**目标对应的文件不存在时**，执行对应命令
- **当依赖在时间上比目标更新时**，执行对应命令(要理解下)
- **当依赖关系连续发生时**，对比依赖链上的每一个目标

小技巧

- makefile 中可以在命令前加上 @ 符，作用为命令无回显。
- 工程开发中可以将最终可执行文件名和all同时作为 makefile 中第一条规则的目标

```makefile
案例：
hello.out all: main.o func.o
	gcc -o hello.out main.o func.o
main.o: main.c
	gcc -o main.o, -c main.c
func.o: func.c
	gcc -o func.o -c func.c
```

小结

- makefile 用于定义源文件间的**依赖关系**

- makefile 说明**如何编译各个源文件**并生成可执行文件
- makefile 中的目标之间存在**连续依赖关系**
- **依赖存在**并且**命令执行成功**是目标完成的充要条件
