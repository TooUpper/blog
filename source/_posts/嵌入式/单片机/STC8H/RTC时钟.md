---
title: "RTC时钟"
date: 2024-09-11 08:41:00
categories: "嵌入式"
tags: 
- "STC8H"
---

在嵌入式开发中，**RTC（Real-Time Clock，实时时钟）**是一个重要的**硬件**模块，用来跟踪当前的时间和日期。与一般的系统时钟不同，RTC 具有独立的电源（通常为电池），即使系统断电或进入低功耗模式，RTC 仍然能够保持运行。

**RTC的基本特性**

- **独立性**：RTC 时钟是**独立于系统主处理器运行的**，即使系统进入低功耗模式或关机，RTC 仍然能正常计时。这通常是通过一块**备用电池**（如纽扣电池）来供电实现的，如果没有电池，系统断电后 RTC 会停止工作，导致时间信息丢失。

- **低功耗**：RTC 的设计非常节能，因为它需要在设备断电或处于待机状态下保持时间。典型的 RTC 功耗极低，使得它能够在备用电池的支持下持续工作多年。

- **精度**：RTC 时钟的频率通常由一个外部的 **32.768kHz晶振** 提供，这种频率的晶振能够精确跟踪时间变化。之所以选择这个频率，是因为它可以通过二进制的除法很方便地分频为1秒。

  32.768kHz 可以是默认，也可以是可设置的

- **计时功能**：除了记录当前时间，RTC 通常还能提供**闹钟、定时**等功能。例如，可以设定某个时间触发闹钟中断，唤醒处理器执行特定任务。

**RTC时钟的组成部分**

- **晶振（Oscillator）**：通常，RTC 时钟使用一个 **32.768kHz** 的石英晶体振荡器来提供计时参考。这种晶振因为频率较低，功耗小，且稳定性高，适合长时间精确计时。

- **备用电池**：为了确保系统断电后，RTC 仍能继续保持正确的时间，通常会有一个小型纽扣电池为其供电。即使系统关闭或断电，RTC 依然可以正常工作。

- **寄存器**：RTC 中包含用于存储当前时间、日期、以及闹钟设定的寄存器。操作系统或嵌入式程序可以通过访问这些寄存器来获取或设置时间信息。

**RTC 与系统时钟的区别**

- **RTC（实时时钟）**：RTC的主要任务是记录“实际时间”，即年、月、日、时、分、秒等。它不依赖系统的运行状态，即使在设备断电的情况下，它仍能通过备用电池继续运行。

- **系统时钟**：系统时钟（或CPU时钟）是提供处理器和系统运行时基的时钟信号。它的频率通常较高（以MHz或GHz为单位），主要用于协调系统中各个组件的工作。

**RTC 中时间和日期的存储**

RTC 负责跟踪当前的时间和日期，这些信息通常以寄存器的形式存储在 **RTC 芯片**中。不同 RTC 芯片的实现略有不同，但一般来说，RTC 会将时间和日期以**二进制编码的十进制（BCD）格式**或直接的**二进制格式**存储。

> PCF8563 芯片使用 BCD 格式进行存储

![Decimal2BCD](/public/image/嵌入式/MCU/STC8H/Decimal2BCD.jpg)

![shijiyinjiao](/public/image/嵌入式/MCU/STC8H/shijiyinjiao.jpg)

## STC8H

**以 PCF8563 为例**

PCF8563：PCF8563 是一款低功耗的 I2C RTC 时钟芯片，能够以 BCD 格式存储时间和日期信息，并具有时钟报警、时钟输出等功能。它具有低功耗、集成度高、工作稳定等特点，适用于需要长时间运行且功耗要求较低的应用场景。

PCF8563 通过**I2C总线**与主控芯片（如微控制器）进行通信。

**原理图**

![RTCshizhongyuanlitu](/public/image/嵌入式/MCU/STC8H/RTCshizhongyuanlitu.png)

![I2Cyuanlitu](/public/image/嵌入式/MCU/STC8H/I2Cyuanlitu.png)

引脚说明：

1. INT： 中断引脚。当触发到定时任务时，会触发引脚高低电平变化。
2. SCL和SDA：为I2C通讯的两个引脚。用来保证MCU和RTC时钟芯片间进行通讯的。

### 实现

```c
// main.c
#include	"STC8G_H_Delay.h"
#include	"STC8G_H_GPIO.h"
#include	"STC8G_H_NVIC.h"
#include	"STC8G_H_Switch.h"
#include	"STC8G_H_I2C.h"
#include 	"STC8G_H_UART.h"

// UART_GPIO
void UART_GPIO_Config(void) {

    GPIO_InitTypeDef UART_GPIO_init;
	//IO模式,GPIO_PullUp,GPIO_HighZ,GPIO_OUT_OD,GPIO_OUT_PP
    UART_GPIO_init.Mode = GPIO_PullUp;		
    UART_GPIO_init.Pin = GPIO_Pin_0 | GPIO_Pin_1; //要设置的端口
    GPIO_Inilize(GPIO_P3, &UART_GPIO_init);        
}

// 配置 UART
void UART_Config(void) {
    
    COMx_InitDefine UART_Init;   
    //模式,UART_ShiftRight,UART_8bit_BRTx,UART_9bit,UART_9bit_BRTx
    UART_Init.UART_Mode = UART_8bit_BRTx;
    //使用波特率,   BRT_Timer1,BRT_Timer2,BRT_Timer3,BRT_Timer4
    //查看手册 UART 与 Timer 要对应起来
	UART_Init.UART_BRT_Use = BRT_Timer1;		
    //波特率,一般 110 ~ 115200
	UART_Init.UART_BaudRate = 115200;	
    //多机通讯允许, ENABLE,DISABLE
	UART_Init.Morecommunicate = DISABLE;	
    //允许接收,ENABLE,DISABLE
	UART_Init.UART_RxEnable = DISABLE;		
    //波特率加倍, ENABLE,DISABLE
	UART_Init.BaudRateDouble = DISABLE;	
    
    UART_Configuration(UART1, &UART_Init);
    UART1_SW(UART1_SW_P30_P31);
}

void main() {
    
    //初始化一个显示函数
    struct RTC_Time time = {
        // 按照结构体定义顺序，年月日，时分秒，周
        2024,
        9,
        12,
        18,
        4,
        24,
        3        
    };
    
    EA = 1; // 打开全局中断使能 UART 要用
    
    UART_Config();
    RTC_Init();
    
    //向 RTC 写入初始时间
    RTC_WriteTime(&time);
    
    //读出 RTC 时间并 UART 打印
    while(1) {
        
        // 读取 RTC 时间
        RTC_ReadTime(&time);
        
        printf("%d-%d-%d %d:%d:%d \n",time.year, (int)time.month, (int)time.day, (int)time.hour, (int)time.minute, (int)time.second);
        printf("week=%d\n", (int)time.week);

        //1s间隔读一次时间
        delay_ms(250);
        delay_ms(250);
        delay_ms(250);
        delay_ms(250);
    }
}
==================================================================
// RTC.c
 
// 零时存储时间的数组
u8 timeDate[7];

// 十进制转 BCD
#define Decimal2BCD(time) ((time / 10) << 4 + (time %10) 

// BCD 转十进制
#define DCB2Decima(i, b) (((timeDate[i] & b) >> 4) * 10) + (timeDate[i] & 0x0F)

#define DEV_ADDR 0xA2
#define MEN_ADDR 0x02
    
void RTC_GPIO_Config(void) {
    GPIO_InitTypeDef RTC_GPIO_init;
	//IO模式,GPIO_PullUp,GPIO_HighZ,GPIO_OUT_OD,GPIO_OUT_PP
    RTC_GPIO_init.Mode = GPIO_OUT_OD;		
    RTC_GPIO_init.Pin = GPIO_Pin_2 | GPIO_Pin_3; //要设置的端口
    GPIO_Inilize(GPIO_P3, &RTC_GPIO_init);
}

void RTC_I2C_Config() {
    I2C_InitTypeDef I2C_Init;
    //总线速度=Fosc/2/(Speed*2+4),0~63
    // 注意不要超过 PCF8563 的最大传输速度
    I2C_Init.I2C_Speed = 13;				
    //I2C功能使能,ENABLE, DISABLE
	I2C_Init.I2C_Enable = ENABLE;				
    //主从模式选择,  I2C_Mode_Master,I2C_Mode_Slave
	I2C_Init.I2C_Mode = I2C_Mode_Master;		
    //主机使能自动发送,  ENABLE, DISABLE
	I2C_Init.I2C_MS_WDTA = DISABLE;				
	
    I2C_Init(&I2C_Init);
	//u8	I2C_SL_ADR; //从机设备地址,  0~127
	//u8	I2C_SL_MA; //从机设备地址比较使能,  ENABLE, DISABL
    
    I2C_SW(I2C_P33_P32);
}
    
void RTC_Init(void) {
    EAXSFR(); // 开启特殊寄存器使能
    
    RTC_GPIO_Config();
    RTC_I2C_Config();
}

// 写入 RTC
void RTC_WriteTime(RTC_Time* time) {
    // 1,将时间存入数组，注意要转换为 BCD 码
    // 注意：在写入时，库函数是将数组按顺序存入 RTC 寄存器中的，
    // 所以此时要将数组的参数与 RTC 寄存器中时间的存储顺序对应
    // RTC 中存储的寄存器顺序为：
    // 秒、分钟、小时、日、星期、月/世纪、年
    timeDate[0] = Decima2BCD(time->RTC_Sec);
    timeDate[1] = Decima2BCD(time->RTC_Min);
    timeDate[2] = Decima2BCD(time->RTC_Hour);
    timeDate[3] = Decima2BCD(time->RTC_Day);
    timeDate[4] = (time->RTC_Week);
    timeDate[5] = Decima2BCD(time->RTC_Month);
    timeDate[6] = (((time->RTC_Year) % 100 / 10) << 4) + ((time->RTC_Year) % 10);
    
    //世纪处理：
    if(time->RTC_Year > 2100) {
        timeDate[5] |= 0x80;
    }else {
        timeDate[5] &= 0x80;
    }
    
    //写入时间
    //从设备地址、从设备寄存器地址、写入的数据、要写入数据多少
    I2C_WriteNbyte(DEV_ADDR, MEN_ADDR, timeDate, 7);
}

// 从 RTC 中读取
void RTC_ReadTime(RTC_Time* time) {
    
    I2C_ReadNbyte(DEV_ADDR, MEN_ADDR, timeDate, 7);
    
    // 将读取的 BCD 转为十进制并存入结构体
    // 因为读取和写入都是按照顺序进行的，这里存放也要对应
    // 秒、分钟、小时、日、星期、月/世纪、年
    time->RTC_Sec = DCB2Decima(0, 0x70);
    time->RTC_Min = DCB2Decima(1, 0x70);
    time->RTC_Hour = DCB2Decima(2, 0x70);
    time->RTC_Day = DCB2Decima(3, 0x70);
    time->RTC_Week = timeDate[4];
    time->RTC_Month = DCB2Decima(5, 0x70);
    time->RTC_Year = DCB2Decima(6, 0x70);
    
    if(timeDate[5] * 0x80) {
        time->RTC_Year += 2100;
    }else {
        time->RTC_Year += 2000;
    }
}
==================================================================
//RTC.h
#ifndef __RTC_H
#define __RTC_H
    
#include "type_def.h"
// RTC 时间结构体    
type struct {
    u16	RTC_Year;  //RTC 年, 
	u8	RTC_Month; //RTC 月, 01~12
	u8	RTC_Day;   //RTC 日, 01~31
	u8	RTC_Hour;  //RTC 时, 00~23
	u8	RTC_Min;   //RTC 分, 00~59
	u8	RTC_Sec;   //RTC 秒
    u8  RTC_Week;  //RTC 周
}RTC_Time;   

// 初始化RTC
void RTC_Init();

RTC_WriteTime(RTC_Time* time);
RTC_ReadTime(RTC_Time* time);
    
#endif    
```

## 问题

**在 PCF8563芯片中，晶振频率为32.768kHz，但我们为什么可以选择 32.768kHz、1024kHz、32Hz、1Hz，他不是只有个 32.768kHz 的晶振么**

在 **PCF8563** 实时时钟（RTC）芯片中，虽然它使用了**32.768kHz**的晶振作为基准频率，但通过内部的**分频电路**，可以将该基准频率分频为其他不同的频率输出，如**1024Hz**、**32Hz**、和**1Hz**。这意味着尽管晶振频率固定为32.768kHz，但通过硬件电路的分频机制，PCF8563芯片可以输出不同的时钟频率。

**I2C 的总线到底如何配置**

I2C（Inter-Integrated Circuit）总线的速度是指主设备和从设备之间数据传输的速率。

公式为：总线速度 = **Fosc/2/(Speed*2+4),0~63**

**FOSC**：这是系统的主时钟频率（通常是微控制器的晶振频率）。

**MSSPEED**：用于控制 I2C 速度的一个寄存器值或配置参数。

**2 和 4**：这是 I2C 控制器内部的**定值**，用于分频公式中的基准值。

> MSSPEED 是一个控制分频的参数，用于将主时钟（FOSC）分频成较低的频率，从而得到 I2C 总线所需的时钟频率（SCL）。
>
> 通常情况下，I2C 的速度取决于具体应用的要求、设备能力和电路布局。如果连接的设备需要高速传输，则可以选择快速模式或更高的模式；而在低功耗应用中，标准模式的 100 kbit/s 速度可能就足够了。

**在开发文档中 MSSPEED 所对应的值为时钟数，那么这个是时钟数是什么？**

在开发文档中提到的 "时钟数" 是指 **I2C 总线传输一个数据位所需的时钟周期数**。具体来说，它是 I2C 通信中每发送或接收一位数据时，SCL（串行时钟线）所需要的时钟周期数。
