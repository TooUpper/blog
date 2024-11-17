---
title: "SysTick"
date: 2024-11-08 21:57:00
categories: "ARM"
tags: 
- "GD32F407"
---

SysTick 叫做系统滴答时钟、系统定时器，是系统内核中的一个片上外设被捆绑在 NVIC 中，用来产生 SYSTICK 异常 SysTick_Handler（异常号:15）。
**SysTick 是一个 24bit 向下递减的计数器**，当计到 0 时，从 RELOAD 寄存器中自动装载定时初值，并触发中断，进行周期性任务。

## 系统定时器的用途

- **没有操作系统：**只用于延时（使用内核的SysTick定时器来实现延时，可以不占用系统定时器，节约资源）
- **有操作系统：**（ucos2、ucos3、freertos为操作系统提供精准的系统时基(1ms~50ms）

## SysTick 时钟来源

可以来自两个地方：

- **内核时钟：**AHB 时钟 8 分频
- **内核时钟的 1/8：**HCLK 时钟 / (AHB 时钟)

> 通常情况下我们一般选择内核时钟，因为他的精度更高

## 时钟树

![F407shizhongsujiegoutu](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407shizhongsujiegoutu.png)

## SysTick 寄存器

**操作 SysTick 通常通过直接访问寄存器来完成**。SysTick 是一个硬件模块，其功能由特定的寄存器控制。

![F407SysTickjicuqni](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407SysTickjicuqni.png)

## 工作流程

SysTick 的工作流程如下：

1. 配置时钟源
   - 选择 HCLK 或 HCLK/8 作为计数器的时钟输入。
2. 加载重载值
   - 将所需的计时周期转换为计数值，写入 LOAD 寄存器。
   - 计数周期为 LOAD + 1 个时钟周期。
3. 计数递减
   - 每个时钟周期，计数器递减 1。
   - 当前值保存在 VAL 寄存器。
4. 计数到零
   - 当计数器递减到零：
     - 如果启用了中断，会触发中断处理程序。
     - 自动从 `LOAD` 寄存器重新加载初始值，继续计数。

## 实现

在 ARM32 架构中通常都帮我们写好了相关配置，我们只需要调用 SysTick_Config 这个函数并传入周期值，他会在计数到 0 时自动触发中断，我们在相应的中断处理函数中进行操作即可；

```c
#ifndef SYS_TICK_H
#define SYS_TICK_H

#include <stdint.h>

/* configure systick */
void systick_config(void);
/* delay a time in milliseconds */
void delay_1ms(uint32_t count);
void delay_1us(uint32_t count);
/* delay decrement */
void delay_decrement(void);

#endif /* SYS_TICK_H */
===================================================
#include "gd32f4xx.h"
#include "systick.h"

volatile static uint32_t delay;

void systick_config(void) {
    /* setup systick timer for 1000Hz interrupts */
    if(SysTick_Config(SystemCoreClock / 1000000U)) {
        /* capture error */
        while(1) {
        }
    }
    /* configure the systick handler priority */
    NVIC_SetPriority(SysTick_IRQn, 0x00U);
}

void delay_1ms(uint32_t count) {
    delay = count * 1000;
    while(0U != delay) {
    }
}

void delay_1us(uint32_t count) {
    delay = count;
    while(0U != delay) {
    }
}

/*!
    \brief    delay decrement
    \param[in]  none
    \param[out] none
    \retval     none
*/
void delay_decrement(void) {
    if(0U != delay) {
        delay--;
    }
}
===========================================
void SysTick_Handler(void)
{
//    led_spark();
    delay_decrement();
}
// 处理函数
void TimingDelay_Decrement(void) {    
	if (uwTimingDelay != 0x00) { 
		uwTimingDelay--;
	}
}
```

