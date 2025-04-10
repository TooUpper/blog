---
title: "Keil调试"
date: 2025-1-10 20:39:00
categories: "嵌入式"
tags: 
- "烧录调试"
---

Keil 是 ARM 旗下的一款 IDE（集成开发环境），主要用于嵌入式系统开发，特别适用于 Cortex-M 处理器系列。Keil 提供了一整套的开发工具，包括代码编辑、编译、链接和调试。

## Keil调试方式

Keil 提供多种调试方式，包括：

**软件仿真（Simulator）**：在 PC 上模拟 MCU 的执行，不依赖实际硬件，适用于代码逻辑验证。
**JTAG/SWD 仿真器调试**：使用硬件调试工具（如 J-Link、ST-Link）进行单步调试、寄存器查看、变量监视等。
**Flash 下载调试**：将程序烧录到 MCU 的 Flash 中，适用于真实硬件运行测试。



## JTAG/SWD 仿真器调试



![ch340lj](/public/image/嵌入式/烧录调试/串口烧录/ch340lj.png)

