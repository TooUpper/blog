---
title: "RTOS介绍"
date: 2024-11-22 10:52:00
categories: "FreeRTOS"
tags: 
- "FreeRTOS"
---

FreeRTOS 是一个开源的轻量级**实时操作系统**（RTOS），专为嵌入式系统设计。由 Real Time Engineers Ltd. 开发，目前由亚马逊维护并以 MIT License 开源。它广泛应用于物联网（IoT）、工业自动化和其他对实时性要求较高的场景。

FreeRTOS 的任务调度本质上是**并发（Concurrency）**而非并行（Parallelism）。

## 特点

**轻量化设计**

- 代码量小，占用内存低，适合资源受限的嵌入式系统。
- 内核大小通常只有几 KB。

**可移植性**

- 支持多种架构（ARM Cortex-M、RISC-V、x86 等）。
- 可跨平台运行，容易适配不同硬件。

**实时性**

- 支持**抢占式调度**、时间片轮转调度和优先级调度。
- 延迟和抖动低。

**模块化**

- 可裁剪的内核，用户可以根据需求选用或裁剪功能模块。

**丰富的功能**

- 提供任务管理、信号量、队列、互斥锁、定时器等功能。

**社区支持和生态**

- 大量开发者和维护团队支持。
- 与 AWS IoT 集成，提供 IoT 应用的解决方案。

## 编码规范

### 命名规范

RTOS 内核和演示应用程序源代码使用以下惯例:[FreeRTOS编码标准风格指南](https://freertos.org/zh-cn-cmn-s/Documentation/02-Kernel/06-Coding-guidelines/02-FreeRTOS-Coding-Standard-and-Style-Guide)

- 变量

  - *uint32_t* 类型变量以 *ul* 为前缀，其中“u”表示“unsigned” ，“l”表示“long”。
  - *uint16_t* 类型变量以 *us* 为前缀，其中“u”表示“unsigned” ，“s”表示“short”。
  - *uint8_t* 类型变量以 *uc* 为前缀，其中“u”表示“unsigned” ，“c”表示“char ”。
  - 非 stdint 类型的变量（我们自定义的变量类型，非 uint32_t、uint16_t 等这些类型）以 *x* 为前缀。例如，BaseType_t 和 TickType_t， 二者分别是可移植层定义的定义类型，主要架构的自然类型或最有效类型， 以及用于保存 RTOS 滴答计数的类型。(类似这样：BaseType_t  xA;)

  - 非 stdint 类型的未签名变量（Unsigned Type）存在附加前缀 *u*。例如， UBaseType_t（未签名 BaseType_t）类型变量以 *ux* 为前缀。
  - *size_t* 类型变量也带有 *ux* 前缀。
  - 枚举变量以 *e* 为前缀
  - 指针以附加 *p* 为前缀，例如，指向 uint16_t 的指针将以 *pus* 为前缀。
  - 根据 MISRA 指南，未限定标准 *char* 类型仅可包含 ASCII 字符， 并以 *c* 为前缀。
  - 根据 MISRA 指南，char* 类型变量仅可包含指向 ASCII 字符串的指针， 并以 *pc* 为前缀。

- 函数

  - 文件作用域静态（私有）函数以 *prv* 为前缀。
  - 根据变量定义的相关规定，API 函数以其返回类型为前缀， 并为 *void* 添加前缀 *v*。
  - API 函数名称以定义 API 函数文件的名称开头。例如，在 tasks.c 中定义 vTaskDelete， 并且具有 void 返回类型。

- 宏

  - 宏以定义宏的文件为前缀。前缀为小写。例如， configUSE_PREEMPTION 在 FreeRTOSConfig.h 中定义。
  - 除前缀外，所有宏均使用大写字母书写，并使用下划线来分隔单词。

### 数据类型

仅使用 stdint.h 类型和 RTOS 自带的 typedef，但以下情况除外：

- char

  根据 MISRA 指南，仅在未限定字符类型包含 ASCII 字符方可使用未限定字符类型。

- char *

  根据 MISRA 指南，仅在未限定字符指针指向 ASCII 字符串时方可使用未限定字符指针 。使用需要 char * 参数的标准库函数时， 无需抑制良性编译器警告，此举尤其考虑到将一些编译器默认为未限定 char 类型是签名的， 而其他编译器默认未限定 char 类型是未签名的。

针对每个移植定义四种类型。即：

- TickType_t

  - 如果 configUSE_16_BIT_TICKS 设置为非零 (true) ，则将 TickType_t 定义为未签名的 16 位类型。
  - 如果 configUSE_16_BIT_TICKS 设置为零 (false)，则将 TickType_t 定义为未签名的 32 位类型。

  请参阅 API 文档的[自定义](https://freertos.org/Documentation/02-Kernel/03-Supported-devices/02-Customization/)章节 获取完整信息。

  32 位架构应始终将 configUSE_16_BIT_TICKS 设置为 0。

- BaseType_t

  架构中最有效、最自然的类型。例如，在 32 位架构上， BaseType_t 会被定义为 32 位类型。在 16 位架构上，BaseType_t 会被定义为 16 位类型 。如果将 BaseType_t 定义为 char， 则须特别注意确保将签名字符用于可能为负的函数返回值来指示错误。

- UBaseType_t

  未签名的 BaseType_t。

- StackType_t

  意指架构用于存储堆栈项目的类型。通常是 16 位架构上的 16 位类型 和 32 位架构上的 32 位类型，但也有例外情况。供 FreeRTOS 内部使用。

> pd 是 **Portable Data**（可移植数据）的缩写，用于表示布尔值、错误状态或状态码等。它通常出现在宏定义和返回值中，提供了一种统一的、平台无关的表示方式。

## 核心组件

### **任务（Task）**

- FreeRTOS 的基本单元，每个任务相当于一个独立的线程。
- 每个任务有优先级，调度时优先级高的任务先执行。
- 任务的状态：
  - 就绪态（Ready）
  - 运行态（Running）
  - 阻塞态（Blocked）
  - 挂起态（Suspended）

### **调度器（Scheduler）**

- **抢占式调度**：优先级高的任务可以抢占低优先级任务。
- **时间片轮转**：相同优先级的任务按时间片轮流执行。
- **静态优先级调度**：优先级固定，调度简单高效。

### **中断服务（Interrupt Handling）**

- 支持中断嵌套。
- 提供中断安全 API，如 xQueueSendFromISR。

### **信号量（Semaphore）**

- **二值信号量**：任务间简单同步。
- **计数信号量**：用于计数资源访问次数。

### **互斥锁（Mutex）**

- 防止多个任务同时访问共享资源。
- 提供优先级继承机制，防止优先级反转。

### **消息队列（Queue）**

- 用于任务间通信或中断与任务间通信。
- 支持任意类型数据的传递。

### **定时器（Timer）**

- 软件定时器，用于定时执行任务。
- 提供单次触发和周期触发两种模式。

### **内存管理**

提供多种动态内存分配策略：

- heap_1：固定大小分配，不支持释放。
- heap_2：简单合并算法，支持释放。
- heap_4：最常用，内存碎片少。
- heap_5：支持多内存区域。
