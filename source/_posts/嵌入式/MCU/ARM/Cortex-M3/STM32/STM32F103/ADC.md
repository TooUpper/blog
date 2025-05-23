---
title: "ADC"
date: 2025-02-28 20:32:00
categories: "ARM"
tags: 
- "STM32F103"
---

ADC（模数转换器）是一种12位逐次逼近型转换器，支持最多18个通道（16个外部信号源和2个内部信号源）。它支持单次、连续、扫描或间断模式转换，转换结果可左对齐或右对齐存储在16位数据寄存器中。内置模拟看门狗功能，可监测输入电压是否超出用户设定的阈值。ADC输入时钟由PCLK2分频产生，最高频率不超过14MHz。

## 主要特性

**基础性能**

- 12位分辨率。
- 供电范围：2.4V至3.6V。
- 输入电压范围：VREF- ≤ VIN ≤ VREF+。

**工作模式**

- 支持单次、连续、自动扫描、间断模式。
- 规则转换和注入转换均支持外部触发。
- 双重模式（适用于多ADC器件）。

**功能特性**

- 转换结束、注入转换结束、模拟看门狗事件触发中断。
- 自校准功能，数据对齐（内嵌一致性校验）。
- 支持通道独立的采样间隔编程。
- 规则通道转换期间可产生DMA请求。

**转换时间**（典型值）

- STM32F103xx增强型：1μs（56MHz时钟）或1.17μs（72MHz）。
- STM32F101xx基本型：1μs（28MHz时钟）或1.55μs（36MHz）。
- STM32F102xx USB型：1.2μs（48MHz时钟）。
- STM32F105xx/107xx：1μs（56MHz时钟）或1.17μs（72MHz）。

**注意事项**

- **VREF-引脚连接**：若封装包含VREF-引脚，必须将其与VSSA（模拟地）连接。
- **时钟限制**：输入时钟频率不可超过14MHz，需通过PCLK2分频配置。

## ADC模块框图

![ADCmkkt](/public/image/嵌入式/MCU/ARM/Cortex-M3/STM32/STM32F103/ADCmkkt.png)

## ADC引脚

![ADCyj](/public/image/嵌入式/MCU/ARM/Cortex-M3/STM32/STM32F103/ADCyj.png)

## ADC通道和引脚对应关系

![ADCtd](/public/image/嵌入式/MCU/ARM/Cortex-M3/STM32/STM32F103/ADCtd.png)

## 模板代码

https://github.com/TooUpper/SensorDrive

说明一下这里采样到的数值是 0~4095 因为寄存器最大就是 12 位，所以是 4095；

**ADC值 0**：表示输入电压为 **0V**。

**ADC值 4095**：表示输入电压为 **V_ref**（通常为 3.3V 或 5V）。

**ADC值 2047**：表示输入电压大约为 **V_ref / 2**，例如 1.65V 如果 **V_ref = 3.3V**。

这里得到的 0~4095 不停波动的值指的是该引脚采样所获取的电压值；
