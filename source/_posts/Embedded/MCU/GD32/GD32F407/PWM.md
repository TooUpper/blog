---
title: "PWM"
date: 2024-11-3 22:49:00
categories: "ARM"
tags: 
- "GD32F407"
---

PWM（Pulse Width Modulation，脉冲宽度调制）是一种调制技术，通过改变脉冲信号的**占空比**来控制平均输出电压或功率。

## PWM 的相关参数

PWM主要有三个参数：频率、占空比、分辨率

**频率**：

- **定义**：PWM 信号的频率指的是单位时间内一个完整周期（高电平和低电平持续时间总和）的重复次数，通常以赫兹（Hz）为单位。
- **例子**：如果 PWM 信号的频率为 1 kHz，这意味着每秒钟会重复 1000 次完整的高低电平循环。

**占空比**：

- **定义**：占空比是高电平持续时间与整个周期时间的比例，通常以百分比表示。它决定了平均输出功率。
- **例子**：如果一个 PWM 信号的周期为 10ms，其中高电平持续 6ms，低电平持续 4ms，则占空比为 60%（6 ms / 10 ms × 100%）。在 LED 调光中，60% 的占空比会使 LED 发出比 30%占 空比更亮的光。

**分辨率**：

- 就是占空比**变化的快慢**，**占空比变化的细腻程度**。占空比跳的快如按照 1% 跳变与按照 0.1% 跳变，那么 0.1% 的跳变就越细腻，越柔和。

## PWM 工作原理

![F407PWMjibenjiegou](/public/image/Embedded/MCU/GD32/GD32F407/F407PWMjibenjiegou.png)

图片 PWM 的工作过程为：定时器模块中的计数器（CNT）从 0 开始递增，当达到自动重装寄存器（ARR）设置的最大值时，计数器会自动清零并重新开始。捕获/比较寄存器（CCR）存储一个比较值，当计数器的值小于CCR时，输出信号保持高电平；当计数器的值大于等于CCR时，输出信号变为低电平，从而生成占空比可调的 PWM 信号。通过**调整 ARR 可以控制 PWM 的频率**，而通过**改变 CCR 的值可以调整 PWM 信号的占空比**。

## 实现

功能：通过 PWM 实现一个灯的闪烁

### 代码

```c
#include "gd32f4xx.h"
#include "systick.h"

// 预分频值
#define PRESCALER (168 - 1) // 
// 1ms 的周期 168000000 / 168 = 1000000 / 1000 = 1000 
#define PERIOD (((SystemCoreClock / (PRESCALER + 1)) / 1000) - 1)

// LED1 GPIO
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
	rcu_periph_clock_enable(RCU_TIMER3);
	
	timer_deinit(TIMER3);
	// 先倍频到 168MHz
	rcu_timer_clock_prescaler_config(RCU_TIMER_PSC_MUL4);
	
	timer_parameter_struct initpara;
		
	timer_struct_para_init(&initpara);
    // 预分频倍数
	initpara.prescaler         = PRESCALER;
    // 对齐方式
//  initpara.alignedmode       = TIMER_COUNTER_EDGE;
    // 计数方向
//  initpara.counterdirection  = TIMER_COUNTER_UP;
    // 周期
    initpara.period            = PERIOD; // 1ms
		
	timer_init(TIMER3, &initpara);
	
	// 定时器通道配置
	timer_oc_parameter_struct ocpara;	
	timer_channel_output_struct_para_init(&ocpara);
/*
	带N的为反向，带P的为正向。赋值的结果常量也是需要注意是否带N。
	// 通道使能，开启通道
	ocpara->outputstate  = (uint16_t)TIMER_CCX_DISABLE;
    ocpara->outputnstate = TIMER_CCXN_DISABLE;
	// 设置通道极性为高电平
	// 所谓的极性就是指：
	// 1.通道的有效电平
	// 2.当定时器触发时，通道的电平状态
    ocpara->ocpolarity   = TIMER_OC_POLARITY_HIGH;
    ocpara->ocnpolarity  = TIMER_OCN_POLARITY_HIGH;
	// 设置输出比较通道在空闲状态时的电平为低电平。
    ocpara->ocidlestate  = TIMER_OC_IDLE_STATE_LOW;
    ocpara->ocnidlestate = TIMER_OCN_IDLE_STATE_LOW;
*/
	ocpara.outputstate = TIMER_CCX_ENABLE;	
	// 第一个参数是我们使用的是哪一个定时器
	// 第二个参数是我们使用的哪一个通道，也是就在服用编号表中看到的编号 TIMER3_CH3
	timer_channel_output_config(TIMER3, TIMER_CH_3, &ocpara);
	// 输出模式配置
	// 模式0：在向上计数时，一旦计数器值小于 TIMERx_CH0CV 时，O0CPRE 为有效电平，否则为无效电平。
	// 在向下计数时，一旦计数器的值大于 TIMERx_CH0CV 时 O0CPRE 为无效电平，否则为有效电平。
	// 在 TIMERx_CH0CV 下面就是高电平
	timer_channel_output_mode_config(TIMER3, TIMER_CH_3, TIMER_OC_MODE_PWM0);
	// 配置占空比
	// 为了达到流水灯的效果，我们要让他一开始是 熄灭的，然后逐渐亮灭
	// 周期和分频会自动 + 1 ，占空比不会所以需要加回来
	timer_channel_output_pulse_value_config(TIMER3, TIMER_CH_3, (PERIOD + 1));
	
	timer_enable(TIMER3);

}

// 0~100，100~0
void update_duty(float duty) {
	uint32_t pulse = (uint32_t)((duty / 100) * PERIOD);
	timer_channel_output_pulse_value_config(TIMER3, TIMER_CH_3, pulse);
}

int main(void) {
    systick_config();
	GPIO_LED_Config();
	TImer_LED_Config();
    while(1) { // 100 20*19
		for(int i = 20; i >= 0; i--) {
			update_duty(i * 5);
			delay_1ms(50);
		}
		for(int i = 0; i < 21; i++) {
			update_duty(i * 5);
			delay_1ms(50);
		}
    }
}
```

> prescaler：计数器时钟等于 TIMER_CK 时钟除以(PSC+1)，每次当更新事件产生时，PSC 的值被装入到对应的影子寄存器。

**根据引脚所复用的定时器去选择通道**

**如果想要让多个 LED 灯实现流水效果我们可以使用同一个 PWM 的不同通道，将 GPIO 复用为某个具体的通道，然后配置通道即可**

## 问题

一、**通道极性和 PWM0，PWM1 的区别**

通道的极性代表的就是通道的**有效电平**和事件产生时通道的输出电平是高还是低；

PWM 模式 0。在向上计数时，一旦计数器值小于 TIMERx_CH0CV 时，O0CPRE 为有效电平，否则为无效电平。在向下计数时，一旦计数器的值大于 TIMERx_CH0CV 时，O0CPRE 为无效电平，否则为有效电平。

PWM 模式 1。在向上计数时，一旦计数器值小于 TIMERx_CH0CV 时，O0CPRE 为无效电平，否则为有效电平。在向下计数时，一旦计数器的值大于 TIMERx_CH0CV 时， O0CPRE 为有效电平，否则为无效电平。
