---
title: "Timer"
date: 2024-11-3 9:11:00
categories: "ARM"
tags: 
- "GD32F407"
---

**定时器**（Timer）是微控制器（MCU）中的一个硬件模块，用于对**时间进行精确测量**或生成**周期性事件**。定时器的工作基于时钟信号，通过对预设计数值进行递增或递减，可以用来测量时间间隔、生成定时中断、控制PWM输出等。

## 定时器的工作原理

定时器接收来自 RCC（时钟控制器）的时钟信号，经过**预分频器**对时钟进行分频以生成新的计数时钟来驱动计数器。**计数器按设定的方向递增或递减**，当计数器的值达到**自动重装载寄存器**设定的阈值时，触发更新事件，生成中断信号或DMA请求，同时计数器重新加载起始值继续计数。控制模块负责启动、停止、清零和设置计数方向等操作，并可在计数达到设定值时触发其他外设输出或事件，使定时器能够实现精确的定时、周期性触发、PWM输出等功能。

![F407dingshiqiyuanli](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407dingshiqiyuanli.png)

## 定时器的分类

**内核定时器**

- 主要包括 SysTick 定时器。SysTick 是一个简单的定时器模块，由 ARM Cortex-M 内核自带。通常用于操作系统的时基生成，或者简单的定时任务，如延时和计时。

**常规定时器**

- **通用定时器**：用于多种应用的基础定时器，通常可以支持基本的延时、中断和计数功能。
- **高级定时器**：通常具有更复杂的功能，如高级 PWM 生成、死区时间控制等，适合电机控制等高要求的应用。
- **基本定时器**：功能比较简单，一般只支持基础的定时中断，不具备高级的捕获比较功能。

**专用定时器**

- **独立看门狗**：用于监控系统运行，防止程序跑飞或异常，当检测到系统出现问题时可以复位系统。

- **窗口看门狗**：比独立看门狗更加灵活，可以设置窗口期来进行复位操作，通常用于安全要求较高的系统。

- **RTC（实时时钟）**：提供低功耗的实时时钟功能，可以记录年月日时分秒，适用于需要精确时间管理的应用。

## GD32F407 系列定时器说明

![F407dingshiqifenlei](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407dingshiqifenlei.png)

### 配置项目含义

**可重复性：**指定时器在达到计数上限后是否可以自动重装载并重新开始计数的能力。

**捕获/比较通道：**用于处理输入捕获、输出比较、以及 PWM（脉宽调制）等操作。

- **捕获输入**用于测量输入信号的时间特性，比如脉冲宽度或周期。当外部输入信号发生时（如上升沿或下降沿），捕获通道会将当前计数器的值保存到一个捕获寄存器中。通过读取这个寄存器的值，可以计算出信号的周期或高低电平时间。
- **比较输出**用于生成输出控制信号，比如 PWM 输出或定时中断。定时器在计数时会不断与设置的比较值进行比较，当计数器的值与比较寄存器中的预设值相等时，会触发一个事件，比如输出信号翻转、产生中断等。

**互补/死区时间：**

- **互补输出**指的是定时器产生两个**相互补充的 PWM 信号**，一个为高电平时另一个为低电平，反之亦然。
- **死区时间**是为了避免桥式电路中的**短路**问题而设置的延时，就是在互补信号之间插入的一段小的延时；

**中止输入：**是一种保护机制，用于紧急情况下迅速停止定时器输出的功能；允许外部信号或内部事件触发“中止”操作，从而**立即停用定时器的输出**。

**单脉冲：**定时器在触发条件满足时，只产生**一个脉冲**输出，而不是连续的脉冲信号；在单脉冲模式中，定时器会等待触发信号（可以是外部信号或软件触发）。当触发信号到来时，定时器开始计数，并在达到设定的时间后输出一个单独的脉冲。

**正交译码器：也称为**增量式编码器译码器**，是一种用于检测和解码正交信号的电路或模块。正交译码器的主要用途是对**正交编码器**（Quadrature Encoder）信号进行处理，从而测量旋转或线性位移的位置和方向。**

**主-从管理：**“主-从管理”是指定时器的工作模式，其中一个定时器作为“主定时器”，控制其他定时器作为“从定时器”的工作方式。

**内部连接：**指的是定时器模块内部的不同功能或模块之间的信号连接或通信。这些内部连接允许定时器模块的不同部分相互协调和配合工作，或使定时器与其他外设（如 PWM 输出、输入捕获、输出比较等）之间进行交互。

**DMA：**指定时器与 DMA（Direct Memory Access，直接内存访问）控制器协同工作的一种模式。在这个模式下，定时器的事件（例如计数溢出、比较匹配等）可以触发 DMA 控制器自动从定时器中读取数据并传输到内存，或者将内存中的数据传输到定时器的相关寄存器中，而无需 CPU 的干预。

**Debug 模式：**指定时器在调试过程中如何与调试工具（如调试器、调试接口）配合工作的一种模式。

> 在使用时要注意区分，不同的定时器支持的功能和配置参数不同；

## 计数模式

![F407Timerjishumoshi](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407Timerjishumoshi.png)

**向上计数模式（Upcounting Mode）**：

- 左上图展示的是**向上计数模式**。计数器从 0 开始计数，向上计数直到达到 CAR 的值。
- 达到 CAR 后，计数器重置为 0 并重新开始计数，同时触发一次定时器溢出中断。

**向下计数模式（Downcounting Mode）**：

- 右上图展示的是**向下计数模式**。计数器从 CAR 值开始向下计数，直到计数到 0。
- 到达 0 后，计数器重置为 CAR 并重新开始计数，同时触发一次定时器中断。

**中心对齐双向模式（Center-Aligned Both Mode）**：

- 计数器从 0 向上计数到 ARR，再从 ARR 计数到 0，完成一次完整的计数周期。
- 每次到达 ARR 和 0 时都会触发更新事件，因此在一个周期中触发两次事件。

**中心对齐向上模式（Center-Aligned UpMode）**：

- 计数器从 0 向上计数到最大值（ARR），然后反向重新计数到 0，再次向上计数。
- 定时器模块在计数达到（ARR）时产生一个上溢事件；

**中心对齐向下模式（Center-Aligned Down Mode）**：

- 计数器从最大值（ARR）向下计数到 0，然后反向重新计数到 ARR 值，再次向下计数。
- 在此模式下，每次计数到达 0 时触发中断或更新事件，波形在中心对称。

## 实现

功能：使用定时器实现LED灯的闪烁（改为了串口打印，因为将 GPIO 的控制权交给 Timer 之后就无法通过 GPIO 去改变引脚的状态了，这时候要使用 Timer 的捕获通道进行修改？）

### 代码

```c
#include "gd32f4xx.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"
#include "bsp_usart0.h"

// USART 配置...

// LED1 GPIO Config
void GPIO_LED_Config(void) {
	// LED 总开关
	rcu_periph_clock_enable(RCU_GPIOC);
	gpio_mode_set(GPIOC, GPIO_MODE_OUTPUT, GPIO_PUPD_PULLUP, GPIO_PIN_6);
	gpio_output_options_set(GPIOC, GPIO_OTYPE_PP, GPIO_OSPEED_2MHZ, GPIO_PIN_6);
	// LED1
	rcu_periph_clock_enable(RCU_GPIOD);
	gpio_mode_set(GPIOD, GPIO_MODE_AF, GPIO_PUPD_PULLUP, GPIO_PIN_15);
	gpio_af_set(GPIOD, GPIO_AF_2, GPIO_PIN_15);
	gpio_output_options_set(GPIOD, GPIO_OTYPE_PP, GPIO_OSPEED_2MHZ, GPIO_PIN_15);
	
	// 总开关常开
	gpio_bit_reset(GPIOC, GPIO_PIN_6);	
}

// LED1 TImer Config
void TImer_LED_Config(void) {
    // 初始化
	timer_deinit(TIMER3);
    // 使能定时器外设
	rcu_periph_clock_enable(RCU_TIMER3);
    
    // 由时钟树或者总线框图可以看到 Timer 是挂在 APB1 这条总线上的,频率为 42MHz
    // 为了方便计算，我们将其倍频为 168HMz 进行在进行分频（注意查看框图看起最大支持的频率是多少）
    // 通过时钟树框图可以看到 Timer 支持对 APB1 进行 1、2、4、8、16倍分频
    // 根据要求我们选择 4 倍分频即可
    // 不倍频也可以，那就要按照 42MHz 去进行计算
	rcu_timer_clock_prescaler_config(RCU_TIMER_PSC_MUL4);
	// 根据定时器的工作原理，我们要先将 APB1 总线传递过来的频率进行分频后才可使用
    // 在配置其参数、
	timer_parameter_struct initpara;
    
    /*
    initpara->prescaler = 0U; 预分频器的值，分频系数
    initpara->alignedmode = TIMER_COUNTER_EDGE; // 中断触发方式
    initpara->counterdirection = TIMER_COUNTER_UP; // 定时器计数方式
    initpara->period = 65535U; // 周期值，上升计数时候从0数到这个值触发
    // 设置定时器时钟的分频系数，在 prescaler 后的基础上在进行分频
    initpara->clockdivision = TIMER_CKDIV_DIV1; 
    // 重复计数，每次溢出时会执行多少次重复计数操作。
    // 当溢出中断触发时，重复执行的次数
    initpara->repetitioncounter = 0U; 是    
    */
		
	timer_struct_para_init(&initpara);
	initpara.prescaler         = 16800; // 168000000 / 16800 = 10000 -> 1S
    initpara.period            = 30000; // 3S
		
	timer_init(TIMER3, &initpara);	
	// 触发中断 配置NVIC
	nvic_irq_enable(TIMER3_IRQn, 2, 2);	
	timer_interrupt_enable(TIMER3, TIMER_INT_UP);	
	
	timer_enable(TIMER3);
}

// 定时器3中断处理函数
void TIMER3_IRQHandler(void) {
	if(SET == timer_interrupt_flag_get(TIMER3, TIMER_INT_FLAG_UP)) {
        timer_interrupt_flag_clear(TIMER3, TIMER_INT_FLAG_UP);
		printf("123");		
    }		
}

int main(void) {
    systick_config();
	GPIO_USART_Config();
    USART_Config();
	GPIO_LED_Config();
	TImer_LED_Config();
	printf("Hello\n");
    while(1) {
    }
}
```

> prescaler：计数器时钟等于 TIMER_CK 时钟除以(PSC+1)，每次当更新事件产生时，PSC 的值 被装入到对应的影子寄存器。

### 过程

**Timer 配置（中断）**

- 使能外部 Timer 时钟
- 倍频（可以先试试看频率是否正确，不对就要去查看时钟树结构图将其倍频为 168MHz）
- Timer 配置（分频值、自动重载器的初值、计数的方式、触发的方式）
- 使能 NVIC
- 使能 Timer 中断

- 编写中断处理函数

## 问题

一、**定时器无法直接操控引脚，那为什么还可以将引脚复用为 Timer？**

定时器的核心功能是**计数和生成时间基准**，它本身不会直接输出高低电平信号到 GPIO 上。定时器可以产生中断、溢出标志等，但这些事件并不直接影响 GPIO 的输出电平。

要让定时器影响 GPIO 引脚的状态，需要借助定时器的**输出模式**，如 PWM 模式或输出比较模式。这些模式通过定时器的比较寄存器、占空比和计数周期来调节 GPIO 输出信号。

二、**在配置时候，预分频值和周期值为什么要减一**

**预分频值**

预分频器可以将定时器的时钟（TIMER_CK）频率按 1 到 65536 之间的任意值分频，分频后 的时钟 PSC_CLK 驱动计数器计数。所以我们在使用的时候需要将其减一，因为他再写入寄存器时候会自动加一；

**周期值**

因为定时器的中断方式为溢出中断，

从 0 开始向上计数时。在（TIMERx_CREP+1）次上溢后产生更新事件。

向下计数时候，在（TIMERx_CREP+1）次下溢后产生更新事件。
