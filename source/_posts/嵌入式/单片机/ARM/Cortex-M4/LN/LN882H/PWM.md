---
title: "PWM"
date: 2025-3-6 9:17:00
categories: "ARM"
tags: 
- "LN"
---

LN882H 提供 6 个高级定时器（Timer0 至 Timer5），每个定时器支持两个 PWM 通道（通道 a 和通道 b），因此共有 12 个 PWM 通道（PWM_CHA_0 至 PWM_CHA_11）。具体映射如下：

- PWM_CHA_0 和 PWM_CHA_1 对应 ADV_TIMER_0_BASE；
- PWM_CHA_2 和 PWM_CHA_3 对应 ADV_TIMER_1_BASE；
- PWM_CHA_4 和 PWM_CHA_5 对应 ADV_TIMER_2_BASE；
- PWM_CHA_6 和 PWM_CHA_7 对应 ADV_TIMER_3_BASE；
- PWM_CHA_8 和 PWM_CHA_9 对应 ADV_TIMER_4_BASE；
- PWM_CHA_10 和 PWM_CHA_11 对应 ADV_TIMER_5_BASE。

由于同一定时器下的两个通道共用相同的定时器计数器，同一定时器的两个 PWM 通道必须设置为相同的频率，但占空比可以独立配置。

![error1](/public/image/嵌入式/MCU/ARM/Cortex-M4/LN882H/error1.png)
