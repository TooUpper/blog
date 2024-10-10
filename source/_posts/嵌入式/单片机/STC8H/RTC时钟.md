---
title: "RTC时钟"
date: 2024-09-11 08:41:00
categories: "嵌入式"
tags: 
- "STC8H"
---

在嵌入式开发中，**RTC（Real-Time Clock，实时时钟）**是一个重要的**硬件**模块，用来跟踪当前的时间和日期。与一般的系统时钟不同，RTC 具有独立的电源（通常为电池），即使系统断电或进入低功耗模式，RTC 仍然能够保持运行。

> 这个当前时间与日期，指的是我的 RTC 中设置的时间和日期，一般不是 PC 的当前时间和日期；

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

## STC8H

**以 PCF8563 为例**

PCF8563：PCF8563 是一款低功耗的 I2C RTC 时钟芯片，能够以 **BCD** 格式存储时间和日期信息，并具有时钟报警、时钟输出等功能。它具有低功耗、集成度高、工作稳定等特点，适用于需要长时间运行且功耗要求较低的应用场景。

PCF8563 通过**I2C总线**与主控芯片（如微控制器）进行通信。

**存储格式**

> PCF8563 芯片使用 BCD 格式进行存储

![Decimal2BCD](/public/image/嵌入式/MCU/STC8H/Decimal2BCD.jpg)

![shijiyinjiao](/public/image/嵌入式/MCU/STC8H/shijiyinjiao.jpg)

**原理图**

![RTCshizhongyuanlitu](/public/image/嵌入式/MCU/STC8H/RTCshizhongyuanlitu.png)

![I2Cyuanlitu](/public/image/嵌入式/MCU/STC8H/I2Cyuanlitu.png)

引脚说明：

1. #INT： 中断引脚。当触发到定时任务时，会触发引脚高低电平变化。
2. SCL和SDA：为I2C通讯的两个引脚。用来保证MCU和RTC时钟芯片间进行通讯的。
3. OSCI：振荡器输入
4. OSCO：振荡器输出
5. Vss：地
6. SDA：串行数据 I/O
7. SCL：串行时钟输入
8. CLKOUT：时钟输出（开漏）
9. VDD：正电源

### 实现时间的设置与读取

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

### 实现闹钟

- 闹钟是基于 RTC 提供的实际时间设置的。你需要指定一个绝对的触发时间，包括小时、分钟、甚至日期等。

- 一旦时间到达设定的时刻，闹钟触发一次。

- 闹钟不具备自动重设功能，需要手动重新设定。

这里为了实现闹钟和定时器先介绍两个寄存器：

**控制 / 状态寄存器**

![RTCkongzhijicuncunqi](/public/image/嵌入式/MCU/STC8H/RTCkongzhijicuncunqi.png)

> 这里我们要关注下AF、TF、AIE、TIE
>
> AF：报警标志位，当报警事件发生时，AF 标志位会被置为 1。需要我们手动软件置零。
>
> TF：定时器标志位，当定时器事件触发时，TF 标志位会被置为 1。需要我们手动软件置零。
>
> AIE：报警中断使能位，AIE 控制的是闹钟的中断使能。如果 AIE 置位为 1，AF 置位时将触发中断。
>
> TIE：定时器中断使能位，TIE 控制的是定时器的中断使能。如果 TIE 置位为 1，TF 置位时将触发中断。

**报警寄存器**

![RTCnaozhongjicunqi](/public/image/嵌入式/MCU/STC8H/RTCnaozhongjicunqi.png)

> 这里可以看到我们想要设置某个闹钟时间，只需要将他的最高位置 0，余下的 7 位设置为具体的时间即可。

这里要理解所谓的实现闹钟的功能，并不是说我们设定一个时间后，这个 RTC 到达指定时间后自己会发声，而是指他在达到我们设置顶的事件后他的中断输出引脚会产生一个低电平（或者是高电平），我们要去捕捉这个中断，然后通过蜂鸣器去实现这个闹钟的功能，定时器也是同理。 

**实现**

```c
// 所谓的闹钟就是设定的时间到了，#INT 引脚会发出中断（也就是一个高电平或者低电平）
// 我们需要在这个中断中去进行闹钟功能的触发
// 1.将这个引脚初始化
// RTC.c
void RTC_EXTI_Config(){
	EXTI_InitTypeDef init;
	//中断模式,EXT_MODE_RiseFall 0 上升沿/下降沿中断
    // 		  EXT_MODE_Fall 1 下降沿中断
	init.EXTI_Mode = EXT_MODE_RiseFall;			
	
	Ext_Inilize(EXT_INT3,&init);
	
	NVIC_INT3_Init(ENABLE , Priority_0);
}
// 2.设置闹钟寄存器
void RTC_StartAlarm(RTC_Alarm * alarm) {
	
		//一开始就设置了4个最高位都是1的数据
    	// 4 个 AE 默认 1 不可用
    	// 按寄存器的顺序分别是分钟、小时、天、星期
		u8 arr[4] = {0x80 , 0x80 , 0x80 , 0x80 };

	//1. 配置闹钟开关 :: 允许启用闹钟
	
		//1.1 先把控制寄存器的数据读取出来
		I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
	
		//1.2 修改第1位【AIE = 1】 --- 允许闹钟中断
    	// 闹钟中断使能位
		rtc_dat |= 0x02;

		//1.3 修改第3位【AF = 0】 ---- 表示闹钟还没有响过，
        // 将报警标志位置 0 表示没有闹钟发声
		rtc_dat &= ~0x08;
		
		//1.4 再把修改好的数据写回去
		I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
		
		//再度一次，看看我的配置有没有写进去？
		I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
			
		//2. 到底配的是几点? 什么时间响闹钟?
    	// 配置闹钟多少点响，也就是什么时候触发中断	
		//判断分钟  0 ~ 59
		if(alarm->minute != -1) {
			//2.1 指定闹钟的分
            // 因为是以 BCD 存储的所以要转为 BCD
			arr[0] = Decimal2BCD(alarm->minute );		
			//2.2. 启用闹钟
			arr[0] &= ~0x80 ; // 0111 1111

		}
		
		//判断小时 0 ~ 23
		if(alarm->hour != -1) {
			//2.1 指定闹钟的小时
			arr[1] = Decimal2BCD(alarm->hour );		
			//2.2. 启用闹钟
			arr[1] &= ~0x80 ; // 0111 1111
		}
		
		//判断日期 1 ~ 31
		if(alarm->day != -1) {
			//2.1 指定闹钟的小时
			arr[2] = Decimal2BCD(alarm->day );		
			//2.2. 启用闹钟
			arr[2] &= ~0x80 ; // 0111 1111
		}
		
		//判断星期 0 ~ 6
		if(alarm->week != -1) {
			//2.1 指定闹钟的小时
			arr[3] = Decimal2BCD(alarm->week );		
			//2.2. 启用闹钟
			arr[3] &= ~0x80 ; // 0111 1111
		}			
		// 回写到寄存器
		I2C_WriteNbyte(DEV_ADD, ALA_ADD, arr, 4);		
}

// 判断是闹钟中断还是定时器中断
u8 RTC_ISAlarmINT() {
	u8 dat;
	I2C_ReadNbyte(0xA2, 0x01, &dat, 1);	
	//判断第3位 AF 是否是1，如果是1，即表示是闹钟引发的中断，否则就是其他引发的中断
	return dat & 0x08;
}

// 3.接下来就是中断的配置，
// 当 RTC 的 #INT 引脚触发了高电平（低电平），我们需要指定 #INT 引脚所对应的中断发声
// 从 GPIO 配置中可以看到我们用的 EXIT3 也就是外部中3
===========================================================================
// Exti_lsr.c
extern void handle_tit3_interrupte();    
//========================================================================
// 函数: INT3_ISR_Handler
// 描述: INT3中断函数.
// 参数: none.
// 返回: none.
// 版本: V1.0, 2020-09-23
//========================================================================
void INT3_ISR_Handler (void) interrupt INT3_VECTOR		//进中断时已经清除标志
{
	// TODO: 在此处添加用户代码
//	P03 = ~P03;
	WakeUpSource = 4;
	handle_tit3_interrupte(); // 这就是我们自订的处理函数
}  
=============================================================================
// 4.定义我们处理的逻辑，也就是我们要做的事情（发声）
// 这里又要思考一点事情
// 4.1 前面寄存器介绍中所过 AF / TF 要软件置 0
// 4.2 我们只有一个 #INT 口，却有两个可以触发中断的方法闹钟和定时器
//     这里需要判断下是闹钟还是定时器触发的中断
// main.c
void handle_tit3_interrupte() {
    if(RTC_ISAlarmINT()) {
        os_create_task(TASK_BUZZER); // 是闹钟就发声
    }else {
        os_create_task(TASK_LED); // 不是闹钟就肯定是定时器，就做其他的处理
    }
}   

// 将对应的闹钟中断标志位置 0
void RTC_ClearAlarmFlag() {

	// 清除闹钟控制寄存器配置。
    // 把AF置 0 【响过闹钟之后，这个AF会 置1 ，要手动清0，否则下一次闹钟到了之后，不会响了！】
	I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);	
	// 把第3位 置 0
	rtc_dat &= ~0x08;	
	//再写回去
	I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
}

// 停止（关闭）闹钟的逻辑，将闹钟的中断使能位置 0
// 如果不关闭，假设我设置 30min 时候发声，他会到每一个 30min 发声
void RTC_StopAlarm(){
	// 设置控制寄存器里面的AIE = 0 ， 表示闹钟中断无效！
	I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
	rtc_dat &= ~0x02;
	I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
}
```

### 实现定时器

- PCF8563 定时器是基于倒计时的，可以设定为倒计时模式。当倒计时时间到达 0 时，定时器触发一个事件，并可以选择重新倒计时或者停止。

- 定时器的时间间隔可以设定为秒、分钟等。

- 定时器一般是循环计时，可以周期性地执行任务，时间结束后重新开始计时。

**倒计时定时器寄存器**

![RTCdaojisjishijicunqi](/public/image/嵌入式/MCU/STC8H/RTCdaojisjishijicunqi.png)

> 大致与上面闹钟同理：区别在于两个参数：
> TD1、TD0：用于设置定时器的时钟频率，
>
> 定时器倒计数数值寄存器位描述：具体倒计时要数的数，通过这两个可以设置倒计数的时长
>
> 以 64 Hz 为例，那么 设置寄存器值位 64 就是 1s

```c
// RTC.c
// 1.中断引脚配置（引脚的配置和上面闹钟是一致的可以共用）

// 2.设置定时器配置寄存器
void RTC_StartTimer(RTC_Hz hz , u8 count){
		//2.1 配置定时器开关.. 
		
		//2.1.1 先把控制寄存器读取出来
		I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
		
		//2.1.2 把TIE 【第0位】置1
		rtc_dat |= 1;
		
		//2.1.3 把TF【第2位】 置0
		rtc_dat &= ~0x04;
		
		//2.1.4 把配置再写回去！
		I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
						
		//2.2 配置定的是什么时间 【需要有两个寄存器来协同完成】...
		
		//2.2.1 先配置定时器时钟频率，以及启用这种频率 00: 4096，01: 64，10:  1，11: 1/60
		rtc_dat = 0x80 | hz;
		I2C_WriteNbyte(DEV_ADD, TIM1_ADD, &rtc_dat, 1);
		
		//2.2.2 再配置定时器的时钟周期
		rtc_dat = count; // 结合上面看，就是1s钟触发一次定时操作！
		I2C_WriteNbyte(DEV_ADD, TIM2_ADD, &rtc_dat, 1);
}

// 停止定时器
void RTC_StopTimer() {
	// 设置控制寄存器里面的TIE 【第0位】 = 0 ， 表示定时器中断无效！
	I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
	rtc_dat &= ~0x01;
	I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
}

// 清除定时器中断标志位
void RTC_ClearTimerFlag() {

	// 定时器来过之后，TF位就会置1，我们要手动置0，否则下一次定时器不来了
	I2C_ReadNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
	
	// 把第2位置0
	rtc_dat &= ~0x04;

	I2C_WriteNbyte(DEV_ADD, CON_ADD, &rtc_dat, 1);
}
```

## 问题

一、**在 PCF8563芯片中，晶振频率为32.768kHz，但我们为什么可以选择 32.768kHz、1024kHz、32Hz、1Hz，他不是只有个 32.768kHz 的晶振么**

在 **PCF8563** 实时时钟（RTC）芯片中，虽然它使用了**32.768kHz**的晶振作为基准频率，但通过内部的**分频电路**，可以将该基准频率分频为其他不同的频率输出，如**1024Hz**、**32Hz**、和**1Hz**。这意味着尽管晶振频率固定为32.768kHz，但通过硬件电路的分频机制，PCF8563芯片可以输出不同的时钟频率。

二、**I2C 的总线到底如何配置**

I2C（Inter-Integrated Circuit）总线的速度是指主设备和从设备之间数据传输的速率。

公式为：总线速度 = **Fosc/2/(Speed*2+4),0~63**

**FOSC**：这是系统的主时钟频率（通常是微控制器的晶振频率）。

**MSSPEED**：用于控制 I2C 速度的一个寄存器值或配置参数。

**2 和 4**：这是 I2C 控制器内部的**定值**，用于分频公式中的基准值。

> MSSPEED 是一个控制分频的参数，用于将主时钟（FOSC）分频成较低的频率，从而得到 I2C 总线所需的时钟频率（SCL）。
>
> 通常情况下，I2C 的速度取决于具体应用的要求、设备能力和电路布局。如果连接的设备需要高速传输，则可以选择快速模式或更高的模式；而在低功耗应用中，标准模式的 100 kbit/s 速度可能就足够了。

三、**在开发文档中 MSSPEED 所对应的值为时钟数，那么这个是时钟数是什么？**

在开发文档中提到的 "时钟数" 是指 **I2C 总线传输一个数据位所需的时钟周期数**。具体来说，它是 I2C 通信中每发送或接收一位数据时，SCL（串行时钟线）所需要的时钟周期数。
