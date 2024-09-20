---
title: "PWM"
date: 2024-09-07 09:27:00
categories: "嵌入式"
tags: 
- "STC8H"
---

**PWM（Pulse Width Modulation，脉宽调制）** 是嵌入式开发中一种非常常见的信号调制技术。它通过调节**脉冲信号的宽度**（占空比），来控制电流或电压的平均值，以达到对设备（如电机、LED、音频设备等）的精确控制。

**PWM的基本概念：**

1. **脉冲信号**：
   - PWM 信号是一个方波信号，它在一定时间周期内反复地在高电平（“开”）和低电平（“关”）之间切换。
2. **周期（T）**：
   - PWM信号的周期是指一个完整方波的时间长度。通常用频率（Hz）来表示，频率是周期的倒数，即 **频率 = 1/周期**。
   - 例如，如果 PWM 信号的周期为 2 毫秒（ms），那么频率 = 1 / (2 * 10<sup>-3</sup>) = 500 Hz
3. **占空比（Duty Cycle）**：
   - 占空比是指信号在一个周期内处于**高电平的时间长度**与**总周期长度**的比例。占空比的范围从0%到100%。
   - 占空比 = （高电平时间 / 总周期时间）× 100%
     - 例如，占空比为50%的PWM信号意味着高电平和低电平各占一半时间；占空比为25%则表示信号只有四分之一的时间处于高电平。

## STC8H

STC8H 系列的单片机内部集成了8 通道 16 位高级 PWM 定时器，分成两周期可不同的 PWM，分别命名为 PWMA 和PWMB ，可分别单独设置。

第一组 PWMA 可配置成 4 组互补 / 对称 / 死区控制 的PWM 或捕捉外部信号。

第二组 PWMB 可配置成 4 路PWM 输出或捕捉外部信号。

两组 PWM 的时钟频率可分别独立设置。

### 实现

串口实现 PWM 启停 0x01启动，0x00停止

拷贝所需库文件（其他必备库请自行准备）

```c
#include "STC8G_H_Delay.h"
#include "STC8G_H_GPIO.h"
#include "STC8G_H_NVIC.h"
#include "STC8G_H_Switch.h"
#include "STC8H_PWM.h"
#include "STC8G_H_UART.h"

// PWM 周期时间 24000000L
#define PERIOD (MAIN_Fosc/1000)

// 标记位，是否更新PWM
int is_update_PWM = 1;

void GPIO_Config(void) {
    // 配置马达 IO 引脚
    GPIO_InitTypeDef GPIO_Init;
    // 摄制工作模式为 推挽输出
    GPIO_Init.Mode = GPIO_OUT_PP;		
    GPIO_Init.Pin = GPIO_Pin_1;	    
    GPIO_Inilize(GPIO_P0, &GPIO_Init);

    // 配置 UART IO 引脚
    GPIO_Init.Pin = GPIO_Pin_0 | GPIO_Pin_1;
    GPIO_Inilize(GPIO_P3, &GPIO_Init);
}

void PWM_Config(void) {
    PWMx_InitDefine PWM_Init;
    // PWM 工作模式
    PWM_Init.PWM_Mode = CCMRn_PWM_MODE1;
    // 设置周期时间 1ms
    PWM_Init.PWM_Period = PERIOD - 1;
    // 设置占空比
    PWM_Init.PWM_Duty = 0;
    // 设置死区时间
    PWM_Init.PWM_DeadTime = 0;
    // 输出通道选择
    PWM_Init.PWM_EnoSelect = ENO6P;
    // 使能计数器，开启输入捕获/比较输出
    PWM_Init.PWM_CEN_Enable = ENABLE;
    // 主使能输出
    PWM_Init.PWM_MainOutEnable = ENABLE;
    
    // 既要配置小组，也要配置大组
    PWM_Configuration(PWM6, &init);// 初始化 PWM4 口
    PWM_Configuration(PWMB, &init);// 初始化PWM通用寄存器,  PWMA,PWMB
    
    // 不需要开启中断使能
    // PWM 不涉及中断，但是可以使用中断，如不使用可以不配置
   
    // 切换引脚
    PWM6_SW(PWM6_SW_P01);
}

void UART_Config(void) {
    COMx_InitDefine UART_Init;
    
    UART_Init.UART_Mode = UART_8bit_BRTx; //模式,UART_ShiftRight,UART_8bit_BRTx,UART_9bit,UART_9bit_BRTx
    UART_Init.UART_BRT_Use = BRT_Timer1; //使用波特率, BRT_Timer1,BRT_Timer2,BRT_Timer3,BRT_Timer4
    UART_Init.UART_BaudRate = 115200ul; //波特率, 	   一般 110 ~ 115200
    UART_Init.Morecommunicate = DISABLE;	//多机通讯允许, ENABLE,DISABLE
    UART_Init.UART_RxEnable = ENABLE; //允许接收,   ENABLE,DISABLE
    UART_Init.BaudRateDouble = ENABLE; //波特率加倍, ENABLE,DISABLE

    UART_Configuration(UART1, &UART_Init);

    // 打开串口中断使能
    NVIC_UART1_Init(ENABLE, Priority_1);
    // 切换引脚
    UART1_SW(UART1_SW_P30_P31);
}

int direction = 1; //控制占空比的变化 1 / -1
int percent = 0; // 占空比值    
PWMx_Duty duty; // 占空比设置结构体

// 启动PWM马达
void start_motor() {
    PWM_Config();
    //让pwm从0开始
    percent = 0;
    direction = 1;
}

// 停止PWM马达
void stop_motor() {
    PWMx_InitDefine PWM_Init;
    // PWM 工作模式
    PWM_Init.PWM_Mode = CCMRn_FORCE_INVALID;
    PWM_Configuration(PWM6, &init);
}

void main() {
    EA = 1;
    // 由于PWM的配置相关特殊功能寄存器位于扩展RAM区域，
    // 访问这些寄存器,需先将P_SW2的BIT7设置为1,才可正常读写。
    EAXSFR(); //  扩展寄存器访问使能   
    GPIO_Config();
    PWM_Config();    
    UART_Config();
    
    while(1) {
        // 根据表示位判断是否跟新占空比
        if(is_update_PWM) {
            // 占空比 0~100 循环
            percent += direction;
            if(percent >= 100) {
                direction = -1;
            }else if(percent <= 0) {
                direction = 1;
            }           
            // 设置占空比
            duty.PWM6_Duty = PERIOD * percent / 100;
            // 更新占空比
            UpdatePwm(PWM6, &duty);
        }        
        // 获取串口指令
        if(COM1.RX_Cnt > 1) {
            if(RX1_Buffer[0] == 0x01) {
                is_update_PWM = 1;
                // 启动PWM
                start_motor();               
            }else if(RX1_Buffer[0] == 0x00) {
                is_update_PWM = 0;
                stop_motor();
            }
        }
        COM1.RX_Cnt = 0;        
    }
    delay_ms(20);          
}
```

### 配置

**选择工作模式**

常用的有 模式1、模式2

一般来说，PWM 模式 2 是“下降模式”，即在计数器从最大值向下计数时输出高电平，而从0计数向上时输出低电平，反之亦然。这种模式决定了PWM信号的工作方式和输出形态。

![PWMmoshi.png](/public/image/嵌入式/MCU/STC8H/PWMmoshi.png)

**设置PWM的占空比**

占空比：**占空比是指信号为高电平的时间与一个周期内总时间的比值**（也可以理解为在一个 PWM 周期内高电平所占的比值）。占空比为0意味着信号始终为低电平，而占空比为100%意味着信号始终为高电平。

**选择PWM输出的通道**

正相/反相输出：正相（P）输出即正常的PWM波形，而反相（N）输出是正相波形的反相信号。在某些场合，如电机控制，需要同时输出正反相信号。

> 配置好后，直接通过物理引脚输出即可，为什么还要选择或者说设置一个"PWM输出的通道"?
>
> 不同的微控制器（MCU）通常内部有多个PWM生成模块。每个PWM模块内部可以生成多个不同的PWM信号，称为**通道**。
>
> 假设你使用一个MCU有一个定时器，可以生成4个PWM通道。如果你需要输出两个不同频率和占空比的 PWM 信号到两个引脚上，就可以配置通道 1 和通道 2，分别对应这两个 PWM 信号，并将它们分别映射到不同的物理引脚。

![PWMtongdaoyinjiaotu](/public/image/嵌入式/MCU/STC8H/PWMtongdaoyinjiaotu.png)

**设置PWM的周期**

PWM 周期指的是一个完整的 PWM 波形从开始到结束的时间长度，通常以时间单位（如毫秒或微秒）表示。在一个周期内，PWM 信号会从高电平变化到低电平，然后再回到高电平，形成一个完整的波形。

频率 = 1/周期：频率它描述了PWM信号中脉冲（或周期）出现的快慢。具体来说，频率是单位时间内脉冲重复出现的次数，通常用赫兹（Hz）作为单位来表示。例如，如果一个PWM信号的周期是1毫秒（ms），那么它的频率就是1毫秒的倒数，即1000次/秒，或者说1千赫兹（kHz）。

![PWMzhouqitu](/public/image/嵌入式/MCU/STC8H/PWMzhouqitu.png)

周期的长短决定了PWM信号的频率。周期越短，频率越高；周期越长，频率越低。具体的单位取决于定时器的配置和时钟频率。

> STC8H 的时钟频率为 24 000 000，也就是 1 秒内有 24 000 000 个周期变化
>
> 我们这里将 PWM 周期设置为 1 ms 也就是 24 000 000 / 1000 = 24 000个周期变化
>
> 我们没法通过赋值 1，2这样来设置周期，只能通过上述的方式来进行计算；

**设置死区时间**

死区时间：在一些应用中，尤其是控制 H 桥电路或电机驱动时，正相和反相信号之间可能需要一个短暂的时间间隔，称为**死区时间**。这个间隔可以防止在两个 MOSFET 或者 IGBT 同时导通时造成短路。如果你不需要这种保护，就可以将死区时间设置为0。

**启用PWM主输出**

主输出：这是一个全局开关，控制所有已配置的PWM输出通道是否真正输出信号。如果设置为DISABLE，则即便其他配置已经完成，信号仍不会输出。

**启用定时器计数**

这通常是 PWM 定时器的使能控制位。当启用时，定时器开始计数并生成PWM波形。如果设置为DISABLE，定时器停止计数，PWM信号也停止输出。

> **PWM（脉宽调制）** 一般需要配合**计数器（定时器）**一起使用。PWM信号的生成过程依赖于计数器的工作机制，定时器用于控制PWM信号的频率和占空比。定时器通过不断计数和重置，来实现PWM信号的周期性变化。

**选择输出引脚**

这一步将具体的 PWM 通道与物理引脚进行绑定。PWM4 的正相和反相信号将分别从使用的引脚输出。

### 问题

一、**为什么周期计算要-1**

假设我们希望 PWM 信号的周期时间是 100 个时钟周期。

定时器会从 `0` 开始计数，到达 `99` 后重置回 `0`。(从 0 数到 99 一共 99 次，然后在加一 就变成 0 了 也就是 数了 100 次)

从 `0` 计到 `99`，共计 100 次。

因此，我们设定的周期值应当是 `100 - 1 = 99`。

二、**PWM 要用到计数器，那么为什么没有配置，或者说配置的内容在哪**

三、**特殊功能寄存器**

由于 PWM 的配置相关特殊功能寄存器位于扩展 RAM 区域，访问这些寄存器,需先将P_SW2 的 BIT7 设置为 1,才可正常读写。

```c
EAXSFR();		/* 扩展寄存器访问使能 */
/*
将 EAXFR 置1
STC8H.h
#define	EAXSFR()		P_SW2 |= 0x80 
*/
```

![teshugongnengyjiaojicunqi](/public/image/嵌入式/MCU/STC8H/teshugongnengyjiaojicunqi.png)

![teshujicunqishuom](/public/image/嵌入式/MCU/STC8H/teshujicunqishuom.png)

> 详细可参见STC8手册：
>
> - 3.1.2 《外设端口切换控制寄存器 2（P_SW2）》
> - 9.2.8 《扩展 SFR 使能寄存器 EAXFR 的使用说明》

四、**没有指定为高电平（1）为什么马达也可以震动？**

当你配置完占空比并指定引脚后，PWM 模块已经在你所指定的引脚上开始工作，产生脉冲信号。在 PWM 模式下，即便你没有手动拉高马达引脚，PWM 模块会自动根据配置产生高低电平信号。这种信号的变化（方波）会驱动马达振动。
