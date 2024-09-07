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
3. **占空比（Duty Cycle）**：
   - 占空比是指信号在一个周期内处于**高电平的时间长度**与**总周期长度**的比例。占空比的范围从0%到100%。
   - 占空比 = （高电平时间 / 总周期时间）× 100%
     - 例如，占空比为50%的PWM信号意味着高电平和低电平各占一半时间；占空比为25%则表示信号只有四分之一的时间处于高电平。

维持 20 毫秒的时间不动

## STC8H

STC8H 系列的单片机内部集成了8 通道 16 位高级PWM 定时器，分成两周期可不同的 PWM，分别命名为 PWMA 和PWMB ，可分别单独设置。

第一组 PWMA 可配置成4 组互补/对称/死区控制的PWM 或捕捉外部信号。

第二组 PWMB 可配置成4 路PWM 输出或捕捉外部信号。

两组 PWM 的时钟频率可分别独立设置。

## 实现

控制引脚P2.7实现LED灯1的呼吸效果

拷贝所需库文件（其他必备库请自行准备）

- `STC8H_PWM.c` `STC8H_PWM.h`
- `NVIC.c` `NVIC.h`
- `witch.h`

```c
#include "Config.h"
#include "GPIO.h"
#include "Delay.h"
#include "NVIC.h"
#include "Switch.h"
#include "STC8H_PWM.h"

#define LED_SW P45 // LED 总开关
// LED 
#define LED1 P27
#define LED2 P26
#define LED3 P15
???? ????
1000 1111    
----------
?000 ????     
0110 0000
----------
?110 ????    
#define FREQ 1000

#define PERIOD 	((MAIN_Fosc / FREQ) - 1)	// 周期

PWMx_Duty dutyA;


void GPIO_config(void) {
    GPIO_InitTypeDef	GPIO_InitStructure;		//结构定义
    // 指定引脚
    GPIO_InitStructure.Pin  = GPIO_Pin_5;		
    // 指定IO工作模式
    GPIO_InitStructure.Mode = GPIO_OUT_PP;	
    // 端口/寄存器 初始化
    GPIO_Inilize(GPIO_P4, &GPIO_InitStructure);//初始化
    
    // 指定引脚
    GPIO_InitStructure.Pin = GPIO_Pin_6 | GPIO_Pin_7;
    // 指定IO工作模式
    GPIO_InitStructure.Mode = GPIO_PullUp;
    // 端口/寄存器 初始化
    GPIO_Inilize(GPIO_P2, &GPIO_InitStructure);//初始化
}

void PWM_start_config(void){
    PWMx_InitDefine		PWMx_InitStructure;
		
	// 配置 PWM4
    // 设置工作模式
    PWMx_InitStructure.PWM_Mode = CCMRn_PWM_MODE2;
    PWMx_InitStructure.PWM_Duty = 0; //PWM占空比时间, 0~Period
    //输出通道选择，ENO1P,ENO1N,ENO2P,ENO2N,ENO3P,ENO3N,ENO4P,ENO4N / ENO5P,ENO6P,ENO7P,ENO8P
    PWMx_InitStructure.PWM_EnoSelect  = ENO4P | ENO4N;	
    
    PWM_Configuration(PWM4, &PWMx_InitStructure); //初始化 PWM4 口

	// 配置 PWMA
    PWMx_InitStructure.PWM_Period   = PERIOD; //周期时间,   0~65535
    PWMx_InitStructure.PWM_DeadTime = 0; //死区发生器设置, 0~255
    // 全局控制 PWM 输出是否被允许
    PWMx_InitStructure.PWM_MainOutEnable= ENABLE; //主输出使能, ENABLE,DISABLE
    PWMx_InitStructure.PWM_CEN_Enable   = ENABLE; //使能计数器, ENABLE,DISABLE
    
    PWM_Configuration(PWMA, &PWMx_InitStructure); //初始化PWM通用寄存器,  PWMA,PWMB

	// 切换 PWM4 选择 PWM4_SW_P26_P27 引脚
    // PWM4_SW_P16_P17,PWM4_SW_P26_P27,PWM4_SW_P66_P67,PWM4_SW_P34_P33
    PWM4_SW(PWM4_SW_P26_P27);			

	// 初始化PWMA的中断
    // PWM 不涉及中断，但是可以使用中断，如不使用可以不配置
    NVIC_PWM_Init(PWMA,DISABLE,Priority_0);
}


void main() {
    char direction = 1;
    u8 duty_percent = 0;// 0 -> 100

    // 
    EAXSFR();		/* 扩展寄存器访问使能， 必写！ */
    GPIO_config();
    PWM_config();
    EA = 1;

    // 总开关
    LED_SW = 0;
    LED1 = 0; // P2.7 PWM4
    LED2 = 0;
    LED3 = 0;

    // 循环之前，设置一次pwm（可选）
    dutyA.PWM4_Duty = PERIOD * duty_percent / 100;
    UpdatePwm(PWM4, &dutyA);
    // 0 -> 100
    while(1) {
        duty_percent += direction;
        // 让duty_percent一直在0-100来回往返
        if(duty_percent >= 100) {
            duty_percent = 100;
            direction = -1;
        } else if(duty_percent <= 0) {
            duty_percent = 0;
            direction = 1;
        }
        // 修改PWM4的duty
        dutyA.PWM4_Duty = PERIOD * duty_percent / 100;
        UpdatePwm(PWM4, &dutyA);
				
        delay_ms(10);
    }
}
```

## 配置

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

PWM周期指的是一个完整的PWM波形从开始到结束的时间长度，通常以时间单位（如毫秒或微秒）表示。在一个周期内，PWM信号会从高电平变化到低电平，然后再回到高电平，形成一个完整的波形。

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

这通常是PWM定时器的使能控制位。当启用时，定时器开始计数并生成PWM波形。如果设置为DISABLE，定时器停止计数，PWM信号也停止输出。

> **PWM（脉宽调制）** 一般需要配合**计数器（定时器）**一起使用。PWM信号的生成过程依赖于计数器的工作机制，定时器用于控制PWM信号的频率和占空比。定时器通过不断计数和重置，来实现PWM信号的周期性变化。

**选择输出引脚**

这一步将具体的 PWM 通道与物理引脚进行绑定。PWM4 的正相和反相信号将分别从使用的引脚输出。

## 特殊功能寄存器

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
