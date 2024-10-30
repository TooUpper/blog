---
title: "新建工程步骤"
date: 2024-10-24 19:48:00
categories: "ARM"
tags: 
- "STM32F103"
---

**命名规则**

![STM32F103mmguize](/public/image/嵌入式/MCU/ARM/Cortex-M3/STM32/STM32F103/STM32F103mmguize.png)

1. 创建工程文件夹、Keil 中新建工程，选择芯片型号
2. 工程文件夹中创建 Start、Library、User 等文件夹，复制固件库中的文件到对应文件夹中
3. Keil 工程中添加对应的 Start、Libray、User等同名的分区,然后将文件夹内的文件添加到对应的组里
4. Keil 工程选项，C/C++，Include Paths 中添引入工作文件夹
5. Keil 工程选项，C/C++, Define 内定义 USE_STDPERIRH_DRIVER



启动文件,ARM启动时候要初始化堆栈指针。等等

外设寄存器描述文件和时钟配置文件

内核的寄存器描述文件

```c
Project/
│
├── Docs/                          // 说明文档
│   ├── Guides/                   // 使用指南
│   ├── API/                      // 接口文档
│   └── Changelog/                // 更新日志
│
├── Source/                       // 源代码文件
│   ├── Main/                     // 主程序
│   ├── Utils/                    // 工具函数
│   └── Tasks/                    // 任务管理
│
├── Drivers/                      // 硬件驱动程序
│   ├── GPIO/                     // GPIO 驱动
│   ├── ADC/                      // ADC 驱动
│   └── UART/                     // UART 驱动
│
├── Vendor_Libs/             // 官方提供的固件库函数
│   └── [固件库相关文件]
│
├── Custom_Libs/                  // 自己封装的库
│   ├── Math/                     // 数学库
│   └── Communication/            // 通信库
│
└── Build/                        // 程序生成的可执行文件
    ├── Binaries/                 // 二进制文件
    └── Logs/                     // 日志文件
```

