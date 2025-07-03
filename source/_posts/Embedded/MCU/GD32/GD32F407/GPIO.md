---
title: "GPIO"
date: 2024-10-27 17:00:00
categories: "ARM"
tags: 
- "GD32F407"
---

在 GD32 系列微控制器中，**GPIO（General Purpose Input/Output）**通用输入/输出引脚模块，允许控制和监测外部设备。

GD32 的 GPIO 是由多个端口组成的，每个端口有多个引脚，每个引脚都可以独立配置。通常命名为 GPIOA，GPIOB，GPIOC 等，表示不同的 GPIO **端口**，每个端口中的引脚则表示为 GPIO_PIN_x（例如 GPIO_PIN_0, GPIO_PIN_1）。

### GPIO 架构

CPU 通过总线访问 GPIO 寄存器，通过寄存器设置指令传递到控制器，控制器根据寄存器的配置去操控各个 GPIO 端口的引脚（如 PA0、PA1、PF0 等），从而实现引脚的输入输出控制和模式配置。

![F407GPIOshixianyuanli](/public/image/Embedded/MCU/GD32/GD32F407/F407GPIOshixianyuanli.png)

### GPIO 的工作模式

在嵌入式系统的 GPIO 中，常见的三种工作模式是：**推挽输出模式**、**开漏输出模式**和**高阻抗模式**。每种模式在特定的应用场景下都能发挥不同的作用。

**推挽输出**

- 推挽模式通过两个晶体管（一个与 Vcc 连接，一个与 GND 连接）实现输出。

- 当 GPIO 输出为高电平时，连接 Vcc 的晶体管导通，连接 GND 的晶体管关闭，使引脚输出高电平；当输出为低电平时，连接 GND 的晶体管导通，连接 Vcc 的晶体管关闭，使引脚输出低电平。

![F407tuiwanshuchushiyitu](/public/image/Embedded/MCU/GD32/GD32F407/F407tuiwanshuchushiyitu.png)

**开漏输出**

- 当输出为低电平时，晶体管导通，GPIO 引脚被拉低至 GND。

- 当输出为高电平时，晶体管断开，电路断开，依赖外部上拉电阻将引脚拉至高电平。

![F407kailoushuchushiyitu](/public/image/Embedded/MCU/GD32/GD32F407/F407kailoushuchushiyitu.png)

**高阻输入**

- 在高阻抗模式下，GPIO 引脚既不连接 Vcc，也不连接 GND，而是保持“开放”状态，像一个高电阻。

- 该模式下，GPIO 引脚的输入电压可以被检测，但引脚自身不消耗电流。

![F407gaozushurushiyitu](/public/image/Embedded/MCU/GD32/GD32F407/F407gaozushurushiyitu.png)

### GPIO 输出线与

**推挽线与（禁止）**

在推挽输出中，如果两个引脚分别处于高电平和低电平，直接线与会导致高电平（Vcc）与低电平（地）相连，形成短路，这可能会损坏微控制器或连接的设备。

![F407tuiwanxianyu](/public/image/Embedded/MCU/GD32/GD32F407/F407tuiwanxianyu.png)

> 开漏模式可以在高电平时输出逻辑 1，但由于电流路径、上升时间和阻抗匹配等因素，开漏模式的响应速度通常慢于推挽模式。这使得推挽输出在需要快速切换的应用中更加有效。

### GPIO 输入模式

GPIO 的输入模式通常包括**浮空输入**、**上拉输入**、**下拉输入**、**模拟输入**四种基本类型，每种模式适用于不同的应用场景：

**浮空输入**

- **特点**：引脚不连接任何电压源，输入状态未定义。

- **应用**：适用于不需要任何外部信号的场景，但可能受到电磁干扰，导致输入状态不稳定。

- **注意**：在这种模式下，建议避免直接使用，因为引脚的状态可能会因为噪声而波动。

**上拉输入**

- **特点**：通过内置电阻将引脚连接到 Vcc（高电平），当未连接外部信号时，输入状态为高电平。

- **应用**：常用于按钮开关，确保在未按下时输入为高电平，按下时输入为低电平。

- **优点**：能够有效防止浮空输入带来的干扰，确保输入状态稳定。

**下拉输入**

- **特点**：通过内置电阻将引脚连接到地（低电平），当未连接外部信号时，输入状态为低电平。

- **应用**：同样用于按钮开关，当按钮未按下时输入为低电平，按下时输入为高电平。

- **优点**：与上拉输入类似，可以有效防止输入状态不稳定。

**模拟输入**

- **特点**：引脚可以读取连续的电压值，而不是简单的高低电平。

- **应用**：用于传感器输入（如温度传感器、光敏电阻等），可以提供更丰富的信息。

- **优点**：能够获取更细腻的变化，适合需要高精度读数的应用。

## ARM 系列的 GPIO

**引脚内部结构框图**

![F407ARMGPIO](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMGPIO.png)

### GD32/STM32 系列中 GPIO 的工作模式

**浮空输入**

![F407ARMfukongshuru](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMfukongshuru.png)

**上拉输入**

![F407ARMshanglashuru](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMshanglashuru.png)

**下拉输入**

![F407ARMxialashuru](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMxialashuru.png)

**模拟输入**

![F407ARMmonishuru](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMmonishuru.png)

**推挽输出**

![F407ARMtuiwanshuchu](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMtuiwanshuchu.png)

**开漏输出**

![F407ARMkailoushuchu](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMkailoushuchu.png)

**复用推挽输出**

![F407ARMfuyongtuiwanshuchu](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMfuyongtuiwanshuchu.png)

**复用开漏输出**

![F407ARMfuyongkailou](/public/image/Embedded/MCU/GD32/GD32F407/F407ARMfuyongkailou.png)

## GPIO 实现

**功能：通过按键控制LED灯**

```c
#include "gd32f4xx.h"
#include "gd32f4xx_gpio.h"
#include "gd32f4xx_rcu.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"

// GPIO_LED
void GPIO_LED_Config(void) {

    // 使能GPIOB外部时钟
    rcu_periph_clock_enable(RCU_GPIOB);
    // 设置工作模式GPIO_MODE_OUTPUT
    gpio_mode_set(GPIOB, GPIO_MODE_OUTPUT, GPIO_PUPD_NONE, GPIO_PIN_2);
     // 配置输出选项
    gpio_output_options_set(GPIOB, GPIO_OTYPE_PP, GPIO_OSPEED_2MHZ, GPIO_PIN_2);
}

// GPIO_Key
void GPIO_KEY_Config(void) {
    // 设置GPIOC外部时钟
    rcu_periph_clock_enable(RCU_GPIOC);
    // 设置工作模式
    gpio_mode_set(GPIOC, GPIO_MODE_INPUT, GPIO_PUPD_PULLUP, GPIO_PIN_0);
   
}

int main(void) {

    systick_config();
    GPIO_KEY_Config();
    GPIO_LED_Config();

    FlagStatus KeyFllag =  gpio_input_bit_get(GPIOC, GPIO_PIN_0);
    
    while(1) {
        FlagStatus ThisKeyFllag =  gpio_input_bit_get(GPIOC, GPIO_PIN_0);
        if(KeyFllag == 1 && ThisKeyFllag == 0) {
            // 按下
            KeyFllag = 0;
            gpio_bit_set(GPIOB, GPIO_PIN_2);
            delay_1ms(1000);
        } else if (KeyFllag == 0 && ThisKeyFllag == 1) {
            // 松开
            KeyFllag = 1;
            gpio_bit_reset(GPIOB, GPIO_PIN_2);
            delay_1ms(1000);
        }
    }
}
```

> 步骤：
>
> 1. 查看板子原理图，找到使用的引脚是哪一个，外部有无上/下拉电阻；这样方便确定输入或者输出的工作模式。
>
> - 开启外部时钟
> - 选择GPIO的工作模式
> - 如果是输出要配置输出模式，复用要配置并指定复用的功能是哪一个;

## 问题

一、**GPIO 的默认初始**

在 ARM32 中，GPIO 引脚的默认电平状态总结如下：

1. **输出模式**：
   - 引脚配置为输出模式时，默认电平为**低电平（0）**。
   - 即使设置了上拉模式，电平仍为低，因为引脚输出的是寄存器的值，而寄存器的初始值为 0，除非在代码中手动设置为高电平。
   - 当 GPIO 引脚配置为输出引脚，输出寄存器（GPIOx_OCTL）的值将会从相应 I/O 引脚上输出。**端口输出控制寄存器（GPIOx_OCTL）复位初值为：0x0000 0000**
2. **输入模式**：
   - 引脚配置为输入模式时，默认电平取决于**上拉**或**下拉**配置：
     - **上拉（Pull-up）**：默认电平为**高电平**。
     - **下拉（Pull-down）**：默认电平为**低电平**。
     - **浮空（No Pull）**：引脚未连接上拉或下拉，电平浮动，容易受到噪声影响。
   - 当 GPIO 引脚配置为输入引脚时，所有的 GPIO 引 脚内部都有一个可选择的**弱上拉**和**弱下拉**电阻。外部引脚上的数据在每个 AHB 时钟周期时都 会装载到数据输入寄存器（GPIOx_ISTAT）。**端口输入状态寄存器（GPIOx_ISTAT）复位初值为 0x0000 XXXX**
