---
title: "ADC"
date: 2024-09-02 08:41:00
categories: "嵌入式"
tags: 
- "STC8H"
---

ADC(Analog to Digital Converter 模数转换器）是一种将**模拟信号转换为数字信号**的**电路**。在电子系统中，模拟信号常常需要转换为数字信号进行处理和存储。模数转换的基本原理是将模拟信号进行**采样**，并将采样值**量化**为数字表示。

- 采样：是指在一定时间间隔内对模拟信号进行测量，并将测量值存储在数字形式的数据中
- 量化：是将这些连续的模拟信号值离散化为一系列数字值，通常使用二进制表示。

简单理解，**ADC是把模拟信号转换为数字信号的工具**，我们可以认为，一个信号有强弱之分，强弱的体现为电压的高低。在数字电路中，只有0和1之分，也就是高电平或低电平。那么体现不了这个强弱。ADC的作用就是体现强弱，精确化的拿到具体的值。

只有那些在硬件设计中具备 ADC 模块连接的引脚才能执行模数转换操作。

ADC（模数转换器）采样获取的是**电压值**。

**ADC 转换流程图**

![ADCzhuanhuanliuchengtu](/public/image/嵌入式/MCU/STC8H/ADCzhuanhuanliuchengtu.png)

## STC8H

STC8H 芯片有 15 个通道的 ADC 功能引脚

通过控制滑动变阻器，来观察电压变化。

**引脚图**

![dianweiqiyinjiaotu](/public/image/嵌入式/MCU/STC8H/dianweiqiyinjiaotu.png)

### 实现

```c
#include "Config.h"
#include "STC8G_H_Delay.h"
#include "STC8G_H_GPIO.h"
#include "STC8G_H_UART.h"
#include "STC8G_H_NVIC.h"
#include "STC8G_H_Switch.h"

// 用到引脚需要配置 GPIO
void GPIO_Config(void){
    GPIO_InitTypeDef GPIO_Init;    
   
    // ADC引脚
     GPIO_Init.Mode = GPIO_HighZ;
    GPIO_Init.Pin = GPIO_Pin_5
    GPIO_Inilize(GPIO_P0, &GPIO_Init);
    // UART 引脚
    GPIO_Init.Mode = GPIO_PullUp;
    GPIO_Init.Pin = GPIO_Pin_0 | GPIO_Pin_1;
    GPIO_Inilize(GPIO_P3, &GPIO_Init);
}

// 需要用到 UART 查看数值输出也要配置 UART
void UART_Config(void) {
    COMx_InitDefine UART_Init;
    
    UART_Init.UART_Mode = UART_8bit_BRTx;
    UART_Init.UART_BRT_Use = BRT_Timer1;
    UART_Init.UART_BaudRate = 115200ul;
    UART_Init.Morecommunicate = DISABLE; 
    UART_Init.UART_RxEnable = ENABLE;
    UART_Init.BaudRateDouble = DISABLE;
    
    // UART 必须要通过中断，故要设置下开启中断，并设置中断优先级
    NVIC_UART1_Init(ENABLE, Priority_1);
    // 引脚选择
    UART1_SW(UART1_SW_P30_P31)
}

// 用到 ADC 需要配置ADC
void ADC_Config(void) {
    ADC_INitTypeDef ADC_Init;
    
    // ADC 模拟信号采样时间控制, 0~31（注意： SMPDUTY 一定不能设置小于 10）
    ADC_Init.ADC_SMPduty = 31; 
    // 设置 ADC 工作时钟频率 ADC_SPEED_2X1T~ADC_SPEED_2X16T
    // 可以理解为 ADC 采样的频率或者说次数，一秒钟采样多少次
    ADC_Init.ADC_Speed = ADC_SPEED_2X1T;
    // ADC结果调整,	ADC_LEFT_JUSTIFIED,ADC_RIGHT_JUSTIFIED
    ADC_Init.ADC_AdjResult = ADC_RIGHT_JUSTIFIED;
    //ADC 通道选择时间控制 0(默认),1
    ADC_Init.ADC_CsSetup = 0;
    // ADC 通道选择保持时间控制 0,1(默认),2,3
    ADC_Init.ADC_CsHold = 1 ;
    
    // 初始化
    ADC_Inilize(&ADC_Init);
    //中断使能,
    NVIC_ADC_Init(DISABLE,Priority_0);		
    
    // ADC 电源控制 ENABLE或DISABLE.
    ADC_PowerControl(ENABLE);    
}

void main(void) {
    
    float adc_v = 0.0;
    // 中断总控
    EA = 1;
    
    GPIO_Config();
    UART_Config();
    ADC_Config();
    
    while(1) {
        // 查询一次ADC转换结果. 得到的是一个数值
        // ADC_CH13 是我们所使用的引脚，也就是硬件连接的引脚
        u16 adc_result = Get_ADCResult(ADC_CH13)
        printf("adc_result = %d", adc_result);
        
        adc_v = adc_result * 2.5 / 4096;  
        printf("adc_v = %d", adc_v);
    }
}
```

### 问题

一、**什么是模拟信号采样控制时间**

这里的“采样时间控制”指的是 ADC 在开始转换之前对输入的模拟信号进行采样的时间长短。

具体来说，采样时间是指 ADC 在转换开始前，**输入模拟信号与内部采样电容相连的时间**。采样时间的长短会影响采样电容对输入信号充电的充分程度。如果采样时间太短，采样电容无法完全充电，导致输入信号没有被准确读取，从而影响 ADC 转换后的数值精度。

在 STC8H 的 ADC 模块中，采样时间可以通过软件配置

二、**什么是ADC结果调整**

就是 AD 转换结果对齐方式 设置ADC 转换的结果是左对齐还是右对齐，因为 STC8H 系列的单片机内部分别集成了一个 10 / 12 位高速 A/D 转换器；

ADC 转换结果的数据格式有两种，左对齐和右对齐。可以方便用户进行读取；

在 STC8H 系列中 STC8H1K28、STC8H1K08 是10 位高速 A/D 转换器，其他都是 12位的

![ADCzhuanhuankongzhi](/public/image/嵌入式/MCU/STC8H/ADCzhuanhuankongzhi.png)

三、**什么是通道选择时间**

见下图

四、**什么是通道选择保持时间控制**

见下图

![ADCzhuanhuanshijian](/public/image/嵌入式/MCU/STC8H/ADCzhuanhuanshijian.png)

五、**12位 ADC 采样计算中，2 的十二次方最大时 4095，为何我们在换算时通常是除以 4096**

12 位的寄存器用来存储采样数值，那么这种情况下 12 位全零的状态也应该作为一个采样数值，即 寄存器最大值+1

12位 ADC 的 ADCn 通道的输入电压 = 采样数值 / 4096 \* ADC_VRef 是正解;

六、**ADC 中也有中断，它的使用场景是什么**

ADC 中断是在 **模数转换完成后** 自动触发的。当 ADC 完成了一次从模拟信号到数字信号的转换时，系统会自动生成一个中断请求（ADC 中断标志位被置位），通知处理器模数转换已经完成，可以读取转换结果。

七、**为何在计算时用到的是 2.5v 进行计算，而不是 Vcc 提供的 3.3V**

在电路电位器设计中接入了一个芯片 CJ431/CD431，这是一款电压基准芯片，会恒定的输出 2.5V 电压。
