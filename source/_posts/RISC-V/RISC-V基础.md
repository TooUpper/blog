---
title: "RISC-V基础"
date: 2025-05-12 22:20:00
categories: "RISC-V"
tags: 
- "RISC-V"
---

**模块化 ISA 和增量式 ISA**
模块化 ISA：将指令集分为核心与可选扩展模块，便于灵活组合和定制处理器功能。
增量式 ISA：在已有指令集基础上逐步增加新指令，实现向后兼容并扩展系统能力。

**自陷(trap)**
	在 RISC-V 架构中，有些指令是可选的扩展指令（比如某些特定的浮点运算或向量指令），硬件可能并未实现这些指令。如果软件试图执行一条硬件不支持的扩展指令，硬件会触发一个**自陷（trap）**，即一种异常或中断。
	当自陷发生时，控制权会转移到软件层（通常是操作系统或运行时环境）。软件层会通过**标准库**中的代码来模拟或执行这条未被硬件实现的指令的功能。这种机制允许软件在硬件不支持某些指令的情况下仍然能够正确运行，提供了向后兼容性或跨硬件平台的灵活性。

**“宏”融合**
通过将多条简单指令组合（或“融合”）成一个更复杂的操作来提升性能，而无需在指令集体系结构（ISA）中显式定义这些复杂指令。

**5 级流水的微处理器**
