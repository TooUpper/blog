---
title: "C模块"
date: 2024-12-10 19:13:00
categories: "ESP32"
tags: 
- "ESP32-S3"
---

在 **MicroPython** 中，**C 模块**是指用 C 语言编写并编译的模块，它可以作为扩展模块嵌入到 MicroPython 中，与 Python 代码一起运行。C 模块的主要作用是扩展 MicroPython 的功能，尤其在性能和底层硬件交互方面有显著优势。

MicroPython 中自带的模块无法满足需求（如我们这里需要一个摄像头的驱动、需要特殊的通信协议、算法库等），这里就需要通过 C 模块实现新的功能扩展。

> C 模块并不是任意的他必须能被 MicroPython 解释器加载并执行的，也就是说他需要符合 MicroPython 的规范或者说他能够被 MicroPython 解释器所识别才可以被编译到 MicroPython 中去。

**C 模块作用**

![image-20241211171604360](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20241211171604360.png)

> **ESP-IDF 是 ESP32 官方提供的开发框架**，包含硬件驱动、协议栈和工具链，用于底层硬件控制和系统功能实现。
> **MicroPython 固件**对 ESP-IDF 的底层 C 接口进行了二次封装，将硬件控制和系统功能以更易用的 Python 接口暴露给开发者，同时提供了 **MicroPython 解释器**，用于解释和执行 MicroPython 脚本代码。这种封装机制让开发者可以通过简洁的 Python 语法快速控制 ESP32 的硬件，例如 GPIO、I2C 和 Wi-Fi 等外设。
>
> 然而，对于某些硬件外设或功能，MicroPython 并未直接提供二次封装的接口支持。这种情况下，我们可以使用 **C 模块**来扩展 MicroPython 的功能：
>
> 1. **C 模块的作用**
>    C 模块允许开发者直接调用硬件提供的底层 C API 驱动，同时将这些驱动接口封装为 Python 可调用的形式。这种方式结合了 **C 语言的高性能和硬件控制能力**，以及 **Python 的简单易用性**。
> 2. **C 模块的实现方式**
>    开发者可以通过 MicroPython 提供的扩展机制，将 C 代码中的硬件控制逻辑按照 MicroPython 规定的格式封装为自定义 Python 模块接口。应用层代码可以直接调用这些接口，而无需直接编写 C 代码。

**Python 调用 C 函数的实现原理**

关键在于，如何用 C 语言的形式在 MicroPython 源代码中表示函数的入参和出参

![image-20241211223614684](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20241211223614684.png)

> 在 Python 中万物皆对象，我们要将接收到的对象转换为我们对应的数值，然后再将处理后的结果从数值转换为对象然后返回；

## 编写 C 模块

这里我们以 cexample C 模块为例，可以在 GitHub 上下载 MciroPython 的源码然后在 \examples\usercmodule\cexample 文件中找到该示例。

该目录下有三个文件

![image-20241211233039943](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20241211233039943.png)

三个文件的作用如下：

**.c 文件**

核心文件，负责实现 C 模块的具体功能逻辑。

定义 MicroPython 模块中要暴露给 Python 层的函数和对象。

需要按照 MicroPython 的 C API 格式实现，主要包括：

- 模块方法表：定义模块中的函数。
- 模块对象结构：描述模块的属性和方法。
- 模块注册：将模块注册到 MicroPython 的虚拟机中。

**.make 文件**

提供模块的编译配置，用于 **CMake 构建系统**。

描述如何将 examplemodule.c 编译并链接到 MicroPython 的固件中。

定义模块的源文件、目标名称及依赖关系。

**.mk 文件**

提供模块的编译配置，用于 **Makefile 构建系统**。

定义如何通过 Make 工具将模块编译和链接到 MicroPython 的固件中。

功能类似于 micropython.cmake，但适用于传统的 Makefile 系统。

**这里展示第一个示例：通过模块.函数名的方式调用的如何定义**

```python
# .c 内容示例
/* 第一部分：添加所需要 API 的头文件 */
// 包含 MicroPython API.
// 提供了 MicroPython 运行时相关的各种函数、类型定义等基础功能，是在 C 代码中与 MicroPython 交互的核心头文件
#include "py/runtime.h"
// 用于在涉及定时器（Timer）类示例等场景下获取时间相关操作的支持，方便后续代码与 MicroPython 中时间相关特性协同工作。
#include "py/mphal.h"

/* 第二部分：实现功能 */
// 这是将从 Python 调用的函数，名为 cexample.add_ints（a，b）。
// 这里定义了一个名为 example_add_ints(a, b) 的静态函数
// 它可以在 MicroPython 环境下实现 C 和 Python 的交互
static mp_obj_t example_add_ints(mp_obj_t a_obj, mp_obj_t b_obj) {
    // 从 MicroPython 输入对象中提取整数。
    int a = mp_obj_get_int(a_obj);
    int b = mp_obj_get_int(b_obj);

    // 相加后转换为 MicroPython 对象返回。
    return mp_obj_new_int(a + b);
}

/* 第三部分：将 example_add_ints 函数添加到 example_add_ints_obj 这个模块当中 */
// 定义对上述函数的 Python 引用。使得这个 C 函数可以在 MicroPython 模块的层面被识别和调用，
static MP_DEFINE_CONST_FUN_OBJ_2(example_add_ints_obj, example_add_ints);


/* 第四部分：将模块注册到模块列表 */
// 定义模块的所有属性。
// 表条目是属性名(字符串)和MicroPython对象引用的键/值对。
// 所有标识符和字符串都写为 MP_QSTR_xxx，并将由构建系统优化为字长整数(互联字符串)。
// 这里定义了模块名以及模块内可被外部访问的函数等相关属性信息。
static const mp_rom_map_elem_t example_module_globals_table[] = {
    { MP_ROM_QSTR(MP_QSTR___name__), MP_ROM_QSTR(MP_QSTR_cexample) },
    { MP_ROM_QSTR(MP_QSTR_add_ints), MP_ROM_PTR(&example_add_ints_obj) },
};

/* 第五部分：将模块列表注册到 example_module_globals 字典对象中 */
static MP_DEFINE_CONST_DICT(example_module_globals, example_module_globals_table);

/* 第六部分：定义模块对象 */
const mp_obj_module_t example_user_cmodule = {
    .base = { &mp_type_module },
    .globals = (mp_obj_dict_t *)&example_module_globals,
};

/* 第七部分：注册该模块，使其在 Python 中可用 */
// 注册到 MicroPython 环境中，使得在 Python 代码里可以导入并使用这个 C 语言实现的模块及其提供的功能
MP_REGISTER_MODULE(MP_QSTR_cexample, example_user_cmodule);
```

> 那么我们在 MicroPython 中使用他的步骤如下：
>
> import cexample
>
> cexample.add_ints(a，b)，即可使用
>
> 这里注意导入时候前面要有个 c 

**这里展示第二种：导入一个模块后，通过模块声明一个对象，然后对象.函数名的形式如何定义**



## 编译到 MicroPython

