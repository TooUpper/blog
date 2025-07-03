---
title: "RTC"
date: 2024-11-6 19:44:00
categories: "ARM"
tags: 
- "GD32F407"
---

在 STM32 微控制器中，**RTC（实时时钟）** 是一个片上外设，用于提供精确的时间跟踪功能。它可以记录日期和时间，并支持定时任务和闹钟功能。RTC 还具备低功耗特性，即使系统掉电或进入待机模式时，RTC 仍然能够继续运行，保持时间（外部电池）。

**RTC 本质上就是一个 1 秒计数器,通过秒来换算出时间。因此需要我们提供一个 1HZ 频率的时钟。**

## 结构框图

结构框图展示了其各个子模块和数据流之间的关系。

![F407ATCjgkt](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407ATCjgkt.png)

组件和功能的简要说明：

1. **时钟源**：
   - 支持多个低速时钟源，包括内部的 **IRC32K**、外部晶振 **LXTAL**（32.768kHz）和其他可选择的时钟源。
   - 这些时钟源通过不同的分频器生成合适的时钟信号供 RTC 使用。
2. **分频器**：
   - 包含多个分频器，进行预分频和相位校准，用于生成 RTC 所需的 1Hz 信号。
   - 分频器包括 7 位异步分频器（默认分频值为 128）和 15 位同步分频器（默认分频值为 256）。
3. **日历模块**：
   - 核心功能模块之一，用于管理日期和时间（年、月、日、时、分、秒）并保持同步。
   - 支持自动增量，确保时间连续更新。
4. **闹钟模块**：
   - 支持两个闹钟（Alarm-0 和 Alarm-1），用于定时唤醒或中断功能。
   - 通过设置不同的时间参数，可以实现多种闹钟触发。
5. **输出控制**：
   - 提供多个 RTC 事件的输出，控制不同的中断和标志输出，如 **RTC_OUT** 和 **RTC_ALARM** 输出。
   - 具有中断标志（Alarm Flag）和输出控制，以支持闹钟事件和报警信号输出。
6. **备份寄存器**：
   - 提供备用数据存储，允许在主系统掉电时保存重要数据。
7. **时间戳和唤醒定时器**：
   - RTC 支持时间戳功能（RTC_TS），记录某些事件发生的时间。
   - 支持低功耗模式下的自动唤醒，具备唤醒定时器（RTC_WTRV），可以在特定时间间隔触发唤醒。
8. **控制信号**：
   - 包括多个控制信号，用于配置和管理 RTC 的工作模式和中断响应。

这张图总体上展示了 STM32 中 RTC 模块的结构，主要功能模块包括时钟源管理、分频和校准、日历功能、闹钟功能、时间戳和备份寄存器等，用于提供稳定的时间跟踪和唤醒功能。

## 电源管理单元（PMU）

**电源管理单元（Power Management Unit, PMU）** 是一个负责管理芯片的电源状态和功耗的模块；而**RTC 位于备用域，因此它的电源和一些关键配置确实是由 PMU 管理的**。

![F407MPUdygldy](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407MPUdygldy.png)

组件和功能的简要说明：

**电源输入**

- **V_BAT**：电池电压输入，一般用于供电给备份域（Backup Domain）。
- **V_DD**：主电源输入，提供3.3V，主要用于V_DD域供电。
- **V_DDA**：模拟电源输入，提供3.3V，专门供给模拟模块（如ADC和DAC）。

**电源开关**

图中间有一个**Power Switch**（电源开关），用于切换电源来源，在不同电源域中提供电源管理功能。当主电源（V_DD）丢失时，它可以自动切换到V_BAT电池电源，保证备份域继续工作。

**各个电源域**

- **V_DD Domain**（V_DD域）：提供数字模块的电源，工作电压为3.3V。
  - **FWDGT**：独立看门狗定时器，用于监测系统是否正常运行。
  - **HXTAL**：高速晶振电路，用于提供高频时钟信号。
  - **POR/PDR**：上电复位（POR）和掉电复位（PDR）电路，用于监测电源电压的变化，在电源上电或电压跌落时复位系统。
  - **LDO**：低压差稳压器（Low Dropout Regulator），用于提供稳定的电压给内部电路。
- **V_DDA Domain**（V_DDA域）：提供模拟模块的电源，电压为3.3V。
  - **IRC16M/IRC32K**：内部RC振荡器，分别提供16MHz和32KHz的内部时钟源。
  - **ADC**：模数转换器，用于将模拟信号转换为数字信号。
  - **DAC**：数模转换器，用于将数字信号转换为模拟信号。
  - **LVD**：低压检测器，用于检测电压是否低于设定值。
  - **PLLs**：锁相环，用于生成高频时钟信号。
- **Backup Domain**（备份域）：主要用于低功耗备份模块，工作电压为1.2V。
  - **LXTAL**：低速晶振电路，用于提供低频时钟源。
  - **BPOR**：备份域复位电路。
  - **RTC**：实时时钟模块，用于提供时间管理功能。
  - **BLDO**：备份低压差稳压器，用于将电压稳定在1.2V，为备份域提供稳定电源。
- **1.2V Domain**（1.2V域）：核心电压域，为处理器和高速外设提供1.2V电源。
  - **BSRAM**：备份SRAM，支持低功耗模式下的数据保存。
  - **Cortex-M4**：ARM Cortex-M4处理器核心。
  - **AHB/IPs 和 APB/IPs**：高级高性能总线（AHB）和高级外设总线（APB），用于连接片上外设。

**电源管理**

- **PMU CTL**：电源管理控制器，负责管理不同电源域的电源状态，包括低功耗模式的切换（如SLEEPING、SLEEPDEEP模式）。
- **WKUP**：唤醒信号，用于在低功耗模式下唤醒系统。
- **WKUPR**：备份域唤醒复位信号，用于在备份域中唤醒微控制器，通常用于系统的复位或重新启动。

> RTC 的时钟源可以是低速内部 RC 振荡器（IRC32K）或低速外部晶体振荡器（LXTAL），或高速 外部晶体振荡器（HXTAL）时钟2~31 分频。当 VDD 被关闭时，RTC 只能选择 LXTAL 作为时钟源。

## 影子寄存器

在 STM32 和 GD32 微控制器中，**影子寄存器（Shadow Register）** 是一种特殊的寄存器机制，用于缓冲和同步外设寄存器的内容。

有影子寄存器存在的寄存器实际上他是两个寄存器，一个为预装载寄存器，一个为执行寄存器（影子寄存器）；我们正常都是先更新（写入）到预装载寄存器中，然后再将预装载寄存器的内容写入到影子寄存器中去，在使用的时候通常都是使用的影子寄存器；

## BCD 码

BCD（Binary-Coded Decimal，二进制编码的十进制）是一种将十进制数编码为二进制数的表示方法。BCD 码将每个十进制数的位（0–9）单独编码成四位二进制格式。例如：

- 十进制数 `5` 在 BCD 中表示为 `0101`
- 十进制数 `23` 在 BCD 中表示为 `0010 0011`

BCD 编码的主要特点是 **每一个十进制位（0-9）都用 4 位二进制来表示**。这样可以让数字更直观地表示出来，而且在某些需要和显示设备、时间处理等操作打交道的应用中更方便。

## 备份寄存器

RTC 时钟内部集成 20 个 32 位（共80字节）通用备份寄存器，能够在省电模式下保存数据。当有外部事件侵入 时，备份寄存器将会复位；

备份寄存器的内容在系统断电、复位或主电源掉电的情况下不会丢失，只要有备用电源（如电池或超级电容）供电。备份寄存器和 RTC 的核心电路一样，可以由备用电源提供电力支持。

可以在用户手册 RTC 寄存器章节中中找对应的寄存器名，然后到 .h 头文件中查找使用，默认初始值为 0

## 报警（闹钟）

RTC 的闹钟功能用于在预设的时间生成一个事件或中断，提醒系统执行特定的任务。

RTC 闹钟的中断一般属于外部中断，因为 RTC 模块与其他外设模块（如 UART、I2C 等）一样， 属于片上的外设模块，产生的中断信号来自核心之外，因此被称为外部中断。所以要设置闹钟的中断需要开启 NIVC 和 EXTI；

在使用时根据用户手册中说明的进行配置即可；

> 闹钟是上升沿触发，用户手册中有说明；

## 实现

功能：串口每隔一秒打印时间

### 日历

```c
#include "gd32f4xx.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"
#include "bsp_usart0.h"

#define BCD2NUM(DCB) ((DCB >> 4)*10 + (DCB & 0x0F))

// 年月日，时分秒
int data_time[6] = {0};

// RTC 配置
void RTC_Config(void) {
	// 使能PMU外部时钟 PMU 是电源管理模块
	rcu_periph_clock_enable(RCU_PMU);
	// 使能备份域写保护
	pmu_backup_write_enable();
	
	// 复位备份域
    // 当我们需要重启还继续原来的时间时，该语句需要通过 if 控制只执行一次
	// RCU（复位与控制单元）
	rcu_bkp_reset_enable();
	// 复位备份域，只是将置位复为标志
	// 在置位期间他会一直复位，所以我们还需要关闭复位
	rcu_bkp_reset_disable();
/***************************************/    
	// 打开芯片内部 32KHz 的晶振
    // 如果使用外部高速晶振则不需要打开，因为我们上电时默认打开了
	rcu_osci_on(RCU_IRC32K);
	
	// 等待晶振震荡稳定后才可进行使用
	// 晶振稳定后他返回1，如果是错误或者超时则返回 0
	ErrStatus status = rcu_osci_stab_wait(RCU_IRC32K);
	if(status == ERROR) { 
		printf("RTC OSCI WAIT ERROR...");	
		return ;	
	}
/***************************************/    
    /*
    如果使用的是高速晶振也就是与我们时钟主频共用一个就不需要开启晶振
    我们只需要指定其为时钟源即可
    // 选择 RTC 计数时钟
	rcu_rtc_clock_config(RCU_RTCSRC_IRC32K);
    */
	
	// 选择 RTC 计数时钟
	rcu_rtc_clock_config(RCU_RTCSRC_HXTAL_DIV_RTCDIV);
	// 使能外部RTC时钟
	rcu_periph_clock_enable(RCU_RTC);
	// 等待 RTC 寄存器与 RTC 的 APB 时钟同步并且影子寄存器更新；
	status = rtc_register_sync_wait();
	if(status == ERROR) {
		printf("RTC CLOK sync ERROR...");	
		return;
	}	
}

void rtc_Init() {
	// 按照用户手册中的 RTC 初始化和配置说明进行操作：
	// 设置 INITM 位为 1 进入初始化模式。等待 INITF 位被置 1。
	// 这一步在 被包含在了 rtc_init 函数中
	rtc_parameter_struct rtc_initpara_struct;
	
	rtc_initpara_struct.year = 0x24; /*!< RTC year value: 0x0 - 0x99(BCD format) */
    rtc_initpara_struct.month = 0x11; /*!< RTC month value */
    rtc_initpara_struct.date = 0x07;   /*!< RTC date value: 0x1 - 0x31(BCD format) */
    rtc_initpara_struct.day_of_week = 0x04; /*!< RTC weekday value */
    rtc_initpara_struct.hour = 0x20;        /*!< RTC hour value */
    rtc_initpara_struct.minute = 0x57;      /*!< RTC minute value: 0x0 - 0x59(BCD format) */
    rtc_initpara_struct.second = 0x32;      /*!< RTC second value: 0x0 - 0x59(BCD format) */
    rtc_initpara_struct.am_pm = RTC_PM;     /*!< RTC AM/PM value */
    rtc_initpara_struct.display_format = RTC_24HOUR; /*!< RTC time notation */				
	
	// 始终源经过这两次分屏要化为 1Hz
	// 我们使用的晶振为 32KHz = 32000Hz / (127 + 1) = 250 / （259+1） = 1 
	rtc_initpara_struct.factor_asyn = 0x7F;     /*!< RTC asynchronous prescaler value: 0x0 - 0x7F */
    rtc_initpara_struct.factor_syn = 0xF9;         /*!< RTC synchronous prescaler value: 0x0 - 0x7FFF */
	// 启用外部高速晶振时候要记得重新设置
    // rtc_initpara_struct.factor_syn  = 0xF9;
    
	rtc_init(&rtc_initpara_struct);
	
	// 清除 INITM 位退出初始化模式。
	rtc_init_mode_exit();
}

void bsp_rtc_readDate(void) {
	
	rtc_parameter_struct rtc_initpara_struct;
		
	rtc_current_time_get(&rtc_initpara_struct);
	
	data_time[0] = BCD2NUM(rtc_initpara_struct.year) + 2000;
	data_time[1] = BCD2NUM(rtc_initpara_struct.month);
	data_time[2] = BCD2NUM(rtc_initpara_struct.date);
	data_time[3] = BCD2NUM(rtc_initpara_struct.hour);
	data_time[4] = BCD2NUM(rtc_initpara_struct.minute);
	data_time[5] = BCD2NUM(rtc_initpara_struct.second);
	
	printf("%d-%d-%d %d:%d:%d",data_time[0], data_time[1], data_time[2], data_time[3] ,data_time[4] ,data_time[5]);
}

int main(void) {
    systick_config();
	bsp_USART0_Init();
	RTC_Config();
	rtc_Init();
	printf("App_start...\n");
    while(1) {
		bsp_rtc_readDate();
		delay_1ms(1000);
    }
}
===================================================
// 使用备份寄存器实现只重置一次
// 使用备份寄存器时候，让第二次启动（不断电）不执行复位备份域的操作    
if(RTC_BKP0 != 1) {
    rcu_bkp_reset_enable();
    printf("RUn2...");
    rcu_bkp_reset_disable();
} 
// 备份域默认为 0
// 只在第一次上电后执行，其余重启都不执行初始化操作；
if(RTC_BKP0 != 1) {
    rtc_Init();
    printf("Run...\n");
    RTC_BKP0 = 1;
}
```

从 PMU 中可以看到 RTC 是位于备份域中的，在默认情况下备份域是处于写保护的状态，所以要操作 RTC 我们先要关闭写保护，也就是说我们要先操作 PMU 模块；

如果遇到一些参数不知道怎么填的，可以去看下官方示例，然后去对于的 .h 文件中查找；

两个预分频器的组合选项太多了，可以看下用户手册我们按照用户手册中的进行配置；

在 RTC 中有两个预分频器用来实现日历功能和其他功能。一个分频器是 7 位异步预分频器， 另一个是 15 位同步预分频器。异步分频器主要用来降低功率消耗。如果两个分频器都被使用， 建议异步分频器的值尽可能大。

我们看 RTC 的结构框图可以发现，异步预分是 7 位的，所以我们设置的最大值为 7F；

**复位备份域**

备份域复位会影响备份域中的所有相关模块和寄存器，包括 **RTC** 配置、**LXTAL** 配置，以及任何备份寄存器中的数据。具体来说，备份域复位会将这些模块和寄存器恢复到它们的默认状态，相当于执行了一次硬件复位。

**使用外部高速晶振要记得降频，不然无法通过两个分配器降低为 1Hz**

**两个分配器计算公式**

![F407ARTCfpqjsgs](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407ARTCfpqjsgs.png)

### 闹钟

闹钟就是在日历的基础上配置了两个东西：

闹钟的初始化配置

闹钟的中断处理函数

```c
// 闹钟的初始化配置
// 闹钟的触发实现、NVIC、EXIT
void bsp_rtc_alarm_config(void) {
	// 清除寄存器 RTC_CTL 的 ALRMxEN（x=0，1）位，禁用闹钟
	rtc_alarm_disable(RTC_ALARM0);
    
	// 设置 Alarm 寄存器（RTC_ALRMxTD / RTC_ALRMxSS）	
	rtc_alarm_struct rtc_alarm_time;
    
	// mask 面具、遮盖的意思
	// 设置忽略响的时间，比如如果不设置，他是四号、20小时、57分钟、45秒都会响动
	//我们需要设置的是四号的第20个小时的57分钟的45秒才响，所以需要忽略前面三个时间
	// 闹钟屏蔽字段选项
	rtc_alarm_time.alarm_mask = (RTC_ALARM_DATE_MASK | RTC_ALARM_HOUR_MASK | RTC_ALARM_MINUTE_MASK);         
	// 制定工作在天还是星期
    rtc_alarm_time.weekday_or_date = RTC_ALARM_WEEKDAY_SELECTED;
    rtc_alarm_time.alarm_day = 0x04;
    rtc_alarm_time.alarm_hour = 0x20;
    rtc_alarm_time.alarm_minute = 0x57;
    rtc_alarm_time.alarm_second = 0x45;
    rtc_alarm_time.am_pm = RTC_PM;                 
	
	rtc_alarm_config(RTC_ALARM0, &rtc_alarm_time);
	
	// 闹钟的中断配置
	// 因为闹钟的中断属于外部中断
	// 所以我们需要配置 NVIC 和 EXTI
	nvic_irq_enable(RTC_Alarm_IRQn, 2, 2);
	// 启用 RTC 中断，指定中断源
	rtc_interrupt_enable(RTC_INT_ALARM0);
	// 清除闹钟 0 的中断标志位，防止干扰
	rtc_flag_clear(RTC_FLAG_ALRM0);
	
	// 设置寄存器 RTC_CTL 的 ALRMxEN 位，使能闹钟
	rtc_alarm_enable(RTC_ALARM0);
	
	// EXTI 配置
	exti_interrupt_flag_clear(EXTI_17);
	exti_flag_clear(EXTI_17);
	// 初始化EXTI_17 （必须初始化）
	exti_init(EXTI_17,EXTI_INTERRUPT,EXTI_TRIG_RISING);
	exti_interrupt_enable(EXTI_17);
}

// 闹钟的中断处理函数
/*闹钟中断属于外部中断，所以可以同时通过闹钟和EXTI的标志位进行判断*/
void RTC_Alarm_IRQHandler(void){
   // 以下判断，二选其一即可  
  if(SET == rtc_flag_get(RTC_FLAG_ALRM0)){
    rtc_flag_clear(RTC_FLAG_ALRM0);
    exti_flag_clear(EXTI_17);
    printf("Alarm_1!\n");
  }  
//  if(SET == exti_interrupt_flag_get(RTC_EXTI_LINE)){
//    exti_interrupt_flag_clear(RTC_EXTI_LINE);
//    exti_flag_clear(RTC_EXTI_LINE);
//    rtc_flag_clear(RTC_FLAG_ALRM0);
//    printf("Alarm_2!\n");
//  }
  
}
// main 中不要忘记调用闹钟的配置函数
```

## 问题

一、**在以下情况下，备份域复位是必要的：**

- **RTC 重新配置**：在初始化或重新配置 RTC 时，通常需要清除备份域以确保干净的状态。
- **备份数据清除**：当系统需要清除备份域中的数据（如备份寄存器中的参数）时，可以通过复位来达到目的。
- **时钟源切换**：在某些情况下，当 LXTAL 或其他时钟源出现问题或需要切换时，复位备份域可以确保时钟源的重新配置和可靠启动。

二、**为什么 RTC 不是先开启外部时钟**

在操作时候都是要先 rcu_periph_clock_enable(XXX); 开启，那么到 RTC 时候为什么就不需要 rcu_periph_clock_enable(RTC);了呢？

![F407RTCsjly](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407RTCsjly.png)

RTC（实时时钟）在 STM32 微控制器中的工作机制与系统时钟（如 SYSCLK、HSE、HSI 等）不同，它并不会默认启用系统时钟。RTC 的时钟源与主系统时钟（系统时钟树）是独立的，它通常依赖于专用的低速时钟源，如 **外部低速晶振（LSE）** 或 **内部低速时钟（LSI）**，而不是直接依赖于系统时钟。

所以我们需要先给 RTC 设置一个时钟后才需要启用；

三、**rtc_register_sync_wait() 的作用是什么**

等待 RTC 寄存器与 RTC 的 APB 时钟同步并且影子寄存器更新；

（同步 RTC 寄存器和影子寄存器的内容）

四、**为什么时钟频率不一样会导致读取到的数据不一致**

由于 RTC 时钟源频率较低，而 APB 时钟较高，当 APB 去读取 RTC 寄存器而没有等待同步，可能会读取到过时的数据，导致数据有误；或者当 RTC 正在更新的时候，APB 去读会导致读取到的数据与寄存器的值不一致；

五、**如何保证数据一致性**

由于 RTC 时钟源频率较低，而 APB 时钟较高，系统需要确保在读取寄存器时，读取到的数据是准确和一致的。因此，在读取寄存器之前需要确保寄存器和时钟源同步。

为了解决这个问题，ARM32 使用了同步机制来确保**读取的数据始终是最新的**。这个同步过程由 **影子寄存器（Shadow Register）** 和 **标志位（RTC_CS）** 来管理。

也就是说我们在读取数据的时候实际访问的是影子寄存器；

每两个 RTC 时钟，影子日历寄存器值会更新为真实日历寄存器的值，与此同时 RSYNF 位也会再次置位。

（RSYNF 每2 个 RTC 时钟周期被置位一次。在这时，影子日历寄存器会更新为真实的日历时间和日期。）

> 注意这个置位不仅仅是指 0 变成 1，还有可能是 1 变成 1
>
> 硬件不会主动将这个位置 0，除非我们手动清 0，否则他也不会默认清 0；

**同步的过程**

1. **RTC 寄存器更新**：当 RTC 真实日历寄存器更新时，他要等两个时钟源的周期才会将真实日历寄存器的值复制到影子日历寄存器中，并将 RSYNF 置 1；
2. **标志位 RSYNF**：当影子寄存器的更新完成后，系统需要通过 RSYNF 标志位来通知外设（APB），寄存器已经同步。
   - **RSYNF = 0**：表示主寄存器（如 RTC_TIME 和 RTC_DATE）还未同步更新，外设（APB）不应读取数据。
   - **RSYNF = 1**：表示主寄存器与影像寄存器已经同步，数据是最新的，APB 可以安全地读取。
3. **APB 读取数据**：APB 通过读取 RTC_TIME 和 RTC_DATE 寄存器来获取最新的时间数据。如果此时 RTC_CRL_RSF 标志位为 1，表示同步已完成，APB 可以放心读取这些数据；如果 RSYNF 为 0，APB 需要等待同步完成，以避免读取到不一致的值；

六、**闹钟的忽略选项怎么理解**

在配置 RTC 闹钟时，alarm_mask 是一个用于**屏蔽（忽略）某些时间字段**的选项。当设置 `alarm_mask` 中的特定位时，RTC 闹钟在比较时会忽略该位对应的时间字段。

alarm_mask 通常包含以下几个掩码位：

1. **秒（Seconds）掩码**：忽略秒字段，允许闹钟每分钟触发一次。
2. **分钟（Minutes）掩码**：忽略分钟字段，允许闹钟每小时触发一次。
3. **小时（Hours）掩码**：忽略小时字段，允许闹钟每天的某一时刻触发一次。
4. **日期（Date）掩码**：忽略日期字段，允许闹钟每月的某些时间触发。
5. **星期（Weekday）掩码**：忽略星期字段，允许闹钟按指定的日期触发，而不是星期几。

七、**打印输出时候串口显示问题**

```c
[17:12:16.839]收←◆2024-11- 7 20:58:2
[17:12:17.849]收←◆62024-11- 7 20:58:27
[17:12:18.849]收←◆2024-11- 7 20:58:28
[17:12:19.854]收←◆2024-11- 7 20:58:29
[17:12:20.849]收←◆2024-11- 7 20:58:30
[17:12:21.849]收←◆2024-11- 7 20:58:3
[17:12:22.853]收←◆12024-11- 7 20:58:32
```

记得要再输出后加 \n 作为换行符，不然串口显示时，自己换行就会有问题；
